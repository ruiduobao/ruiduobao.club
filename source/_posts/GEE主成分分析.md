# GEE主成分分析全解析
# 0.背景
主成分分析作为数据降维的重要方法，目前中文网站上没有完整的GEE代码与教程。而我的毕业论文也使用到了主成分法，因此和它很有感情，就写下了这篇博客。

# 1.介绍
主成分分析是将众多具有相关性的数据指标，重新组合成一组新的指标,新形成的指标互不相关，并且前几个主成分能代表原始数据的大部分信息。
![](http://pics.landcover100.com/pics//image/20211010042813.png)
在GEE中，可能会遇到波段数非常多的情况，这时就可以考虑使用主成分分析法只生成两、三个主成分，减少后续工作量。

# 2.代码思路
![](http://pics.landcover100.com/pics//image/20211010033224.png)

# 3.实操
## 3.1 数据筛选与预处理
这一步主要是选择研究区(四川省_资阳市_乐至县)的哨兵影像，并对筛选的数据按照行政边界进行镶嵌与裁剪
```javascript
//筛选数据
var sentImages = ee.ImageCollection(Sentinel)
.filter(ee.Filter.lt('CLOUDY_PIXEL_PERCENTAGE', 10))
.filterDate("2021-08-01", "2021-08-08")
.filterBounds(LeZhiXian);

//镶嵌与裁剪数据
var sentmosaic = sentImages.mosaic();
var sentImage = sentmosaic.clip(LeZhiXian);

//加载研究区影像图层
Map.addLayer(sentImage, trueColor, "乐至县真彩色");
```
![](http://pics.landcover100.com/pics//image/20211010034245.png)

## 3.2 数据筛选与预处理
选择需要进行主成分分析的原始影像波段，并且设置主成分分析影像的分辨率等。在进行主成分分析之前，进行预处理（协方差缩减等）。
```javascript
//需要进行主成分分析的波段选择
var bands=["B1","B2","B3","B4","B5","B6","B7","B8","B9","B11","B12"]
sentImage =sentImage.select(bands)

// 输入到主成分函数的参数设置
var region = LeZhiXian;
var image =  sentImage.select(bands);
var scale = 30;
var bandNames = image.bandNames();

//数据平均
var meanDict = image.reduceRegion({
    reducer: ee.Reducer.mean(),
    geometry: region,
    scale: scale,
    maxPixels: 1e9
});
var means = ee.Image.constant(meanDict.values(bandNames));
var centered = image.subtract(means);
```
## 3.3 主成分分析
这部分的代码是从Google earth engine官方文档中copy过来的，针对输入影像，进行转为数组、计算正交矩阵、计算主成分载荷等操作。
```javascript
//主成分分析函数
var getPrincipalComponents = function(centered, scale, region) {
    // 图像转为一维数组
    var arrays = centered.toArray();

    // 计算相关系数矩阵
    var covar = arrays.reduceRegion({
      reducer: ee.Reducer.centeredCovariance(),
      geometry: region,
      scale: scale,
      maxPixels: 1e9
    });
  
    // 获取“数组”协方差结果并转换为数组。
    // 波段与波段之间的协方差
    var covarArray = ee.Array(covar.get('array'));
  
    // 执行特征分析，并分割值和向量。
    var eigens = covarArray.eigen();
  
    // 特征值的P向量长度
    var eigenValues = eigens.slice(1, 0, 1);
    
    //计算主成分载荷
    var eigenValuesList = eigenValues.toList().flatten()
    var total = eigenValuesList.reduce(ee.Reducer.sum())
    var percentageVariance = eigenValuesList.map(function(item) {
      return (ee.Number(item).divide(total)).multiply(100).format('%.2f')
    })
    
    print("各个主成分的所占总信息量比例", percentageVariance)  
      
    // PxP矩阵，其特征向量为行。
    var eigenVectors = eigens.slice(1, 1);
    
    // 将图像转换为二维阵列
    var arrayImage = arrays.toArray(1);
    
    //使用特征向量矩阵左乘图像阵列
    var principalComponents = ee.Image(eigenVectors).matrixMultiply(arrayImage);

    // 将特征值的平方根转换为P波段图像。
    var sdImage = ee.Image(eigenValues.sqrt())
      .arrayProject([0]).arrayFlatten([getNewBandNames('sd')]);
      
    //将PC转换为P波段图像，通过SD标准化。
    principalComponents=principalComponents
      // 抛出一个不需要的维度，[[]]->[]。
      .arrayProject([0])
      // 使单波段阵列映像成为多波段映像，[]->image。
      .arrayFlatten([getNewBandNames('pc')])
      // 通过SDs使PC正常化。
      .divide(sdImage);
    return principalComponents
  };
//进行主成分分析，获得分析结果
var pcImage = getPrincipalComponents(centered, scale, region);
// 主要成分可视化
Map.addLayer(pcImage, {bands: ['pc3', 'pc2', 'pc1'], min: -2, max: 2}, 'Sentinel 2 - PCA');
```
![](http://pics.landcover100.com/pics//image/20211010035733.png)
## 3.4 数据导出
选择需要导出的主成分波段，这里导出的是前三个波段，因为前三个波段累计贡献率超过95%，完全够用了。
![](http://pics.landcover100.com/pics//image/20211010035934.png)
```javascript
//选择导出的波段
var pcImage_output =pcImage.select(['pc1', 'pc2', 'pc3'])
//导出函数
Export.image.toDrive({
  image: pcImage_output,
  description: 'LeZhiXian_Sentinel_PAC',
  folder:'LeZhiXian',
  scale: 10,
  region:LeZhiXian,
  maxPixels: 1e10
});
```
点击Run进行数据下载，一个县的研究区大概10分钟就能得到预期影像。
![](http://pics.landcover100.com/pics//image/20211010040130.png)
将下载的影像，导入到arcgis或envi中，进行你所需要的分析。
这里稍微提一下，在深度学习中，也可以用主成分分析法处理多波段影像，获得三个波段，用于训练与预测。

# 4.测试链接
https://code.earthengine.google.com/8cdedf188b5e1660c65d84c38767b818