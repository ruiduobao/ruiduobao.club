# GLC30三期土地利用数据按行政边界下载
# 0.背景
由于工作原因，接触PIE-Engine的机会比较多。这几天用PIE-Engine时，发现了里面有Global Land 30的数据。
![](http://pics.landcover100.com/pics//image/20211016223537.png)  
有这个数据，就可以直接用PIE-Engine导出已经裁剪好的数据。

# 1.研究区准备
在PIE里面上传研究区数据，我上传的是四川省_资阳市_乐至县的行政区数据。
![](http://pics.landcover100.com/pics//image/20211016223921.png)
待数据上传后，与GEE不同，该研究区不能直接调用，需要转为geometry类型。
```javascript
//研究区加载 LZ（四川省_资阳市_乐至县）
LZ= LZ.first().geometry(); 
Map.centerObject(LZ,9);    
Map.addLayer(LZ, {color: 'FF0000', fillColor: '00000000', width: 1}, "LZ")
```
# 2.数据筛选
我们以2020年的GLC数据为例，首先加载影像集，筛选时间，选择波段，镶嵌与裁剪。
```javascript
//GLC30 2020年土地利用数据下载
//加载土地利用数据
var LZ_landcover_2020=pie.ImageCollection("NGCC/GLOBELAND30")
              .filterBounds(LZ)
              .filterDate("2020-1-01", "2020-12-31")
              .select(["B1"])
              .mosaic()
              .clip(LZ) ;
//显示土地利用数据
Map.addLayer(LZ_landcover_2020,visParam,"LZ_landcover_2020")
```
在这里，需要注意两个地方:  

ppqq 一个是PIE与GEE的裁剪不同，需要指定波段才能进行
裁剪。  

![](http://pics.landcover100.com/pics//image/20211016224900.png)

ppqq 第二个是GLC数据没有做拼接，需要使用镶嵌函数，对覆盖研究区的多景数据镶嵌。

![](http://pics.landcover100.com/pics//image/20211016225218.png)

# 3.数据导出
这里几乎和GEE差不多，选择下载区域、下载路径等。
```javascript
//下载
Export.image({
    image:LZ_landcover_2020,
    description: "LZ_landcover_2020",
    assetId: "LZ_landcover_2020",
    region:LZ,
    scale:30,
    maxPixels:1e13
});
```
有两个地方需要注意一下:    

ppqq 如果你不知道下载区域大小，maxPixels记得设置大一点；  

![](http://pics.landcover100.com/pics//image/20211016230727.png)      

ppqq PIE数据处理完之后，在PIE资源里面，点击文件即可下载；  

![](http://pics.landcover100.com/pics//image/20211016225636.png)

# 4.数据使用
数据下载后，加载到gis软件里面，就可以看到研究区2000年、2010年
、2020年的30米土地利用数据。
![](http://pics.landcover100.com/pics//image/20211016230004.png)
![](http://pics.landcover100.com/pics//image/20211016230037.png)
![](http://pics.landcover100.com/pics//image/20211016225946.png)

# 5.代码链接
代码链接功能提供了两种方式。  
一种是**外部链接**方式用于给非登陆用户使用:  
https://engine.piesat.cn/engine-share/shareCode.html?id=412ae6c27a3e450b942f45f88d4064a6  
一种是**内部链接**方式，直接在编辑器中打开用于给登陆用户直接使用:    
https://engine.piesat.cn/engine/home?sourceId=412ae6c27a3e450b942f45f88d4064a6

# 6.完整代码
上面展示的是2020年的数据下载代码,2000年和2010年的就是复制一下。（疑问:为什么不循环一下？因为懒，循环哪有ctrl+V来得快）

```javascript
var LZ = pie.FeatureCollection('user/15884411724/quhua/lezhixian')
//研究区加载 四川省_资阳市_乐至县
LZ= LZ.first().geometry(); 
Map.centerObject(LZ,9);    
Map.addLayer(LZ, {color: 'FF0000', fillColor: '00000000', width: 1}, "LZ")

//土地利用数据样式
var visParam = {
    min: 10,
    max: 100,
    palette: '#43B87C,#24DB99,#7EB451,#2E79BC,#2838B8,#8B2CC0,#EEE912,#BC1FA1,#17214F,#B81A74,#B5CF52,#932626,#2B328B,#AA5C5C,#2561E9,#874949,#4ECF61,#AE5151'
};

//GLC30 2000年土地利用数据下载                  
//加载土地利用数据 筛选、镶嵌、裁剪  
var LZ_landcover_2000=pie.ImageCollection("NGCC/GLOBELAND30")
              .filterBounds(LZ)
              .filterDate("2000-1-01", "2000-12-31")
              .select(["B1"])
              .mosaic()
              .clip(LZ);
//显示土地利用数据
Map.addLayer(LZ_landcover_2000,visParam,"LZ_landcover_2000")

//下载
Export.image({
    image:LZ_landcover_2000,
    description: "LZ_landcover_2000",
    assetId: "LZ_landcover_2000",
    region:LZ,
    scale:30,
    maxPixels:1e13
});

//GLC30 2010年土地利用数据下载                 
//加载土地利用数据
var LZ_landcover_2010=pie.ImageCollection("NGCC/GLOBELAND30")
              .filterBounds(LZ)
              .filterDate("2010-1-01", "2010-12-31")
              .select(["B1"])
              .mosaic()
              .clip(LZ) ;
//显示土地利用数据
Map.addLayer(LZ_landcover_2010,visParam,"LZ_landcover_2010")

//下载
Export.image({
    image:LZ_landcover_2010,
    description: "LZ_landcover_2010",
    assetId: "LZ_landcover_2010",
    region:LZ,
    scale:30,
    maxPixels:1e13
});

//GLC30 2020年土地利用数据下载
//加载土地利用数据
var LZ_landcover_2020=pie.ImageCollection("NGCC/GLOBELAND30")
              .filterBounds(LZ)
              .filterDate("2020-1-01", "2020-12-31")
              .select(["B1"])
              .mosaic()
              .clip(LZ) ;
//显示土地利用数据
Map.addLayer(LZ_landcover_2020,visParam,"LZ_landcover_2020")

//下载
Export.image({
    image:LZ_landcover_2020,
    description: "LZ_landcover_2020",
    assetId: "LZ_landcover_2020",
    region:LZ,
    scale:30,
    maxPixels:1e13
});

```
