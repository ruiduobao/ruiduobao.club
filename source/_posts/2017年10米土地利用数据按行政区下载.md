# 10米土地利用数据按行政区下载
# 0.背景
由于工作原因，接触PIE-Engine的机会比较多。这几天用PIE-Engine时，发现了里面有FROM_GLC的10米分辨率数据(2017年)。
![](http://pics.landcover100.com/pics//image/20211016233628.png) 
有这个数据，就可以直接用PIE-Engine导出按行政边界裁剪的10米分辨率土地利用数据。

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
首先加载影像集，筛选时间，选择波段，镶嵌与裁剪。
```javascript
//FROM_GLC10 土地利用数据下载                  
//加载土地利用数据 筛选、镶嵌、裁剪  
var LZ_landcover_2017=pie.ImageCollection("THU/FROM_GLC10_2017")
              .filterBounds(LZ)
              .filterDate("2017-1-01", "2017-12-31")
              .select(["B1"])
              .first()
              .clip(LZ);

//显示土地利用数据
Map.addLayer(LZ_landcover_2017,visParam,"LZ_landcover_2017")
```
在这里，需要注意一个地方:  

PIE与GEE的裁剪不同，需要指定波段才能进行裁剪。  

![](http://pics.landcover100.com/pics//image/20211016224900.png)


# 3.数据导出
这里几乎和GEE差不多，选择下载区域、下载路径等。
```javascript
//下载
Export.image({
    image:LZ_landcover_2017,
    description: "LZ_landcover_2017",
    assetId: "LZ_landcover_2017",
    region:LZ,
    scale:10,
    maxPixels:1e13
});
```
有两个地方需要注意一下:    

ppqq 如果你不知道下载区域大小，maxPixels记得设置大一点；  

![](http://pics.landcover100.com/pics//image/20211016230727.png)      

ppqq PIE数据处理完之后，在PIE资源里面，点击文件即可下载；  

![](http://pics.landcover100.com/pics//image/20211016225636.png)

# 4.数据使用
数据下载后，加载到gis软件里面，就可以看到研究区2017年的30米土地利用数据。
![](http://pics.landcover100.com/pics//image/20211016234758.png)
10米的土地利用数据集，比30米的细腻很多:  
![](http://pics.landcover100.com/pics//image/20211016235013.png)


# 5.代码链接
代码链接功能提供了两种方式。  
一种是**外部链接**方式用于给非登陆用户使用:  
https://engine.piesat.cn/engine-share/shareCode.html?id=95120018f4ae488ea92c1f8b35b4c3ee 
一种是**内部链接**方式，直接在编辑器中打开用于给登陆用户直接使用:    
https://engine.piesat.cn/engine/home?sourceId=95120018f4ae488ea92c1f8b35b4c3ee

# 6.完整代码
```javascript
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

//FROM_GLC10 土地利用数据下载                  
//加载土地利用数据 筛选、镶嵌、裁剪  
var LZ_landcover_2017=pie.ImageCollection("THU/FROM_GLC10_2017")
              .filterBounds(LZ)
              .filterDate("2017-1-01", "2017-12-31")
              .select(["B1"])
              .first()
              .clip(LZ);

//显示土地利用数据
Map.addLayer(LZ_landcover_2017,visParam,"LZ_landcover_2017")

//下载
Export.image({
    image:LZ_landcover_2017,
    description: "LZ_landcover_2017",
    assetId: "LZ_landcover_2017",
    region:LZ,
    scale:10,
    maxPixels:1e13
});
```
