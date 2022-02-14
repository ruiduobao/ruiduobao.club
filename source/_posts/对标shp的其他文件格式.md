## 文件格式介绍

我是地信专业出身，从大一开始就接触arcgis，现在已经7年有余。我喜欢arcgis的功能强大性，但我更喜欢QGIS的开放性。每个人都有不同的软件使用习惯，在我看来是QGIS比arcgis好使，gson比shp好用。

尽管目前有超过80种矢量数据格式，但在考虑数据格式的开放性上，目前只有少数几个可以挑战SHP的行业霸权。它们分别是：

- **OGC GeoPackage**

GeoPackage 是一种开放的、独立于平台的、可移植的的格式，可用于传输地理空间信息。以单个文件形式存在，支持矢量和栅格两种格式。

![](https://i.bmp.ovh/imgs/2022/02/e63a962f748985b7.png)

- **FlatGeobuf**

FlatGeobuf是描述空间地理信息的二进制编码单个文件，对于性能至关重要的系统，建议将 FlatGeobuf 作为 Shapefile 替代品。



- **GeoJSON**

GeoJSON适合处理复杂的矢量数据并构建复杂的分层数据模型，该数据易于解析，并支持流式传输（无需等待整个文件加载）。

![](https://i.bmp.ovh/imgs/2022/02/b3d56586dc84eeb3.png)

- **OGC GML**

GML被选为欧洲 INSPIRE倡议的主要矢量数据格式，但其标准复杂，很少有软件支持整个标准。

![](https://i.bmp.ovh/imgs/2022/02/130483d4610ce12f.png)

- **SpatiaLite**

SpatiaLite 是一个开源库，旨在扩展 SQLite 核心以支持成熟的 Spatial SQL 功能，整个数据库只对应一个单一的整体文件（没有大小限制）。SpatiaLite 与GeoPackage都是建立在底层技术 SQLite 之上。



- **OGC KML**

KML是基于 XML 的，对存储较大的数据集效率不高，另外该格式只支持 WGS-84 坐标系。

![](https://i.bmp.ovh/imgs/2022/02/60a15f7c7bdae5d7.png)

## 文件对比

（1）文件的最大值（G）

![在这里插入图片描述](https://img-blog.csdnimg.cn/9587133368d943cbb0747499af066d87.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA6ZSQ5aSa5a6d55qE5Zyw55CG56m66Ze0,size_20,color_FFFFFF,t_70,g_se,x_16)

（2）读取文件的元数据耗费时间（秒）

![在这里插入图片描述](https://img-blog.csdnimg.cn/9cf170f432ad41e083ddf4f40a39c529.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA6ZSQ5aSa5a6d55qE5Zyw55CG56m66Ze0,size_20,color_FFFFFF,t_70,g_se,x_16)

（3）通过FID查询数据耗费时间（秒）

![在这里插入图片描述](https://img-blog.csdnimg.cn/35875f3f15ad451eb33067169956f23d.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA6ZSQ5aSa5a6d55qE5Zyw55CG56m66Ze0,size_20,color_FFFFFF,t_70,g_se,x_16)

（4）读取相同数据不同格式数据时耗费时间（秒）

![在这里插入图片描述](https://img-blog.csdnimg.cn/361f7a9b323c439589e61e431e12fdf2.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA6ZSQ5aSa5a6d55qE5Zyw55CG56m66Ze0,size_20,color_FFFFFF,t_70,g_se,x_16)

（5）设定条件筛选出感兴趣区域所耗费时间（秒）

![在这里插入图片描述](https://img-blog.csdnimg.cn/8585ca00a4d742f6a9d807754009ae62.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA6ZSQ5aSa5a6d55qE5Zyw55CG56m66Ze0,size_20,color_FFFFFF,t_70,g_se,x_16)

综上，shp文件处理较为低效。另外，使用者最不方便的还是**文件个数**。

ESRI公司凭借自身市场优势，SHP格式称霸GIS市场。几十年过去了，技术在进步，站在巨人们的肩膀上，或许我们可以用更好的数据格式或者创造更好的地理数据格式。

找资料时，我发现了Shapefile Must Die网站的GitHub仓库，感兴趣的同行可以研究一下。

![在这里插入图片描述](https://img-blog.csdnimg.cn/c5880af30fef46b4b9832d83e9a7c331.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA6ZSQ5aSa5a6d55qE5Zyw55CG56m66Ze0,size_20,color_FFFFFF,t_70,g_se,x_16)

参考：

GitHub.https://github.com/opengeolabs/shapefilemustdie

Switch from Shapefile.http://switchfromshapefile.org/