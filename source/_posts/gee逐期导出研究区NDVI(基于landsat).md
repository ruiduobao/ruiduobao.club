# 编程 | gee逐期导出研究区NDVI(基于Landsat)
# 0.背景
做遥感，绕不过去NDVI。如何快速、便捷的获取一个研究区该年度的每一期NDVI？
是通过USGS下载原始数据?试试GEE，批量导出研究区的每一期NDVI。

# 1.数据筛选
首先，在GEE中导入自己需要的矢量边界ROI，这个ROI可以上传shp或者自己在gee中勾画。随后选择卫星数据类型、时间。
本文选择了2021年之后，覆盖了研究区域的所有Landsat8影像。

```javascript
Map.addLayer(ROI)
Map.centerObject(ROI,9)

//筛选数据
var roi_collection =landsat.filter(ee.Filter.date('2021-01-01','2021-12-31'))
                            .filter(ee.Filter.bounds(ROI))
print(roi_collection)
```
# 2.NDVI的计算函数
这一部分包括波段的选取，以及NDVI的计算。
如果NDVI，可以在bands参数里面修改。
```javascript
//需要导出的波段
var bands=["B1","B2","B3","B4","B5","B6","B7","B8","B9","NDVI"]
//或者只选择NDVI
var bands=["NDVI"]
```
接下来是针对collection中的每一张影像进行计算NDVI
```javascript
//NDVI计算函数
var get_NDVI = function(image) {
        image=image.clip(ROI);
        var NDVI=image.normalizedDifference(['B4','B5']).rename(['NDVI']);
        image=image.addBands(NDVI)
        return image.select(bands)
      };
//针对影像集中的每一张影像裁剪ROI区域、计算NDVI、导出相应波段
var NDVI_Collection = ee.ImageCollection(roi_collection)
          .map(get_NDVI);
print(NDVI_Collection)
```

# 3.引入批量导出函数
```javascript
var batch = require('users/fitoprincipe/geetools:batch')
```
这个函数来自于github的一名大神，他针对gee批量导出做了一个整理，并且做成了API，我们可以直接调用。

![](https://gitee.com/kitmyfaceplease/image_upload/raw/master/img/202110062339451.png)

该批量导出的具体代码如下所示:

```javascript
Download.ImageCollection.toDrive = function(collection, folder, options) {
    
    var defaults = {
      scale: 1000,
      maxPixels: 1e13,
      type: 'float',
      region: null,
      name: '{id}',
      crs: null,
      dateFormat: 'yyyy-MM-dd',
      async: false
    }
    var opt = tools.get_options(defaults, options)
    var colList = collection.toList(collection.size());
    
    var wrap = function(n, img) {
      var geom = opt.region || img.geometry()
      var region = getRegion(geom)
      var imtype = IMAGE_TYPES(img, opt.type)
      var description = helpers.string.formatTask(n)
      var params = {
        image: imtype,
        description: description,
        folder: folder,
        fileNamePrefix: n,
        region: region,
        scale: opt.scale,
        maxPixels: opt.maxPixels
      }
      if (opt.crs) {
        params.crs = opt.crs
      }
      Export.image.toDrive(params)
    }
    
    var i = 0
    while (i >= 0) {
      try {
        var img = ee.Image(colList.get(i));
        var n = tools.image.makeName(img, opt.name, opt.dateFormat).getInfo()
        wrap(n, img)
        i++
      } catch (err) {
        var msg = err.message
        if (msg.slice(0, 36) === 'List.get: List index must be between') {
          break
        } else {
          print(msg)
          break
        }
      }
    }
  }
```
# 4.设置导出函数
主要是设置影像分辨率以及导出区域。
```javascript
//导出影像
batch.Download.ImageCollection.toDrive(NDVI_Collection, 'Landsat_NDVI', {
  scale: 30,
  region: ROI
})
```
# 5.完整代码展示
```javascript
Map.addLayer(ROI)
Map.centerObject(ROI,9)

//筛选数据
var roi_collection =landsat.filter(ee.Filter.date('2021-01-01','2021-12-31'))
.filter(ee.Filter.bounds(ROI))
print(roi_collection)
//需要导出的波段
var bands=["NDVI"]
//NDVI计算函数
var get_NDVI = function(image) {
        image=image.clip(ROI);
        var NDVI=image.normalizedDifference(['B5','B4']).rename(['NDVI']);
        image=image.addBands(NDVI)
        return image.select(bands)
      };
//针对影像集中的每一张影像裁剪ROI区域、计算NDVI、导出相应波段
var NDVI_Collection = ee.ImageCollection(roi_collection)
          .map(get_NDVI);
print(NDVI_Collection)

//引入批量导出函数
var batch = require('users/fitoprincipe/geetools:batch')

//导出影像
batch.Download.ImageCollection.toDrive(NDVI_Collection, 'Landsat_NDVI', {
  scale: 30,
  region: ROI
})
//查看大体效果
var NDVI_IMAGE=NDVI_Collection.mosaic()
print(NDVI_IMAGE)
Map.addLayer(NDVI_IMAGE)
```
# 6.下载
比如我们使用的是四川省_资阳市_乐至县的行政区域，可以通过gee先看看大体效果(非单期)。
```javascript
//查看大体效果
var NDVI_IMAGE=NDVI_Collection.mosaic()
print(NDVI_IMAGE)
Map.addLayer(NDVI_IMAGE)
```
![](https://gitee.com/kitmyfaceplease/image_upload/raw/master/img/202110070106817.png)  

之后是针对每一期影像进行下载，点击Run，直接下载影像。
![](https://gitee.com/kitmyfaceplease/image_upload/raw/master/img/202110070038337.png)
需要注意，为了每一期影像名字保持和原始影像相同。下载的影像是按照ROI区域进行了裁剪,并没有按研究区进行镶嵌。
# 7.测试链接
https://code.earthengine.google.com/39c66eee53c92c2c0503fa04fe566714?noload=true