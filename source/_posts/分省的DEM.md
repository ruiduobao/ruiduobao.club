# 1.简介：
数字高程模型（Digital Elevation Model)，简称DEM，是通过有限的地形高程数据实现对地面地形的数字化模拟。
![](https://gitee.com/kitmyfaceplease/image_upload/raw/master/img/202108191839603.png) 
这次分享的数据是全国34个省份的DEM裁剪数据，一共有6期数据。
![](https://gitee.com/kitmyfaceplease/image_upload/raw/master/img/202108191556041.png)
分享的数据效果如上图所示。
# 2.背景介绍
首先，自己的百度云盘堆满了各种DEM数据，但都是单景，每一次使用都需要进行按区域裁剪、镶嵌。做得多也就烦了。
正好，前段时间研究了一下GDAL与Geopandas结合到一起，进行栅格的批量裁剪与转换。我就一次性地把DEM都裁剪到了省。  
![](https://gitee.com/kitmyfaceplease/image_upload/raw/master/img/202108181604673.png)
人数多了，就想要更多人数。  
因此我写了这篇推文，阐述批量裁剪DEM的流程，以及分享各省的裁剪DEM资源。
# 3.数据制作
## 3.1 流程图
总体思路是：DEM数据转换、按省份裁剪DEM、切片镶嵌，结果数据后处理。
![](https://gitee.com/kitmyfaceplease/image_upload/raw/master/img/202108181624185.png)

## 3.1数据准备工作
首先是下载数据，数据量加在一起是60G大小。
![](https://gitee.com/kitmyfaceplease/image_upload/raw/master/img/202108191709881.png)
待下载好之后，检查数据，统一所有数据格式为TIFF。例如NASA_30mDEM数据为1300张hgt格式的影像，需全部转为tiff文件，
这步使用arcgis的栅格批量转换功能。
![](https://gitee.com/kitmyfaceplease/image_upload/raw/master/img/202108181534624.png)
## 3.2 DEM按省份裁剪
### 3.2.1 脚本进行裁剪
使用python的RasterIO模块进行单景的DEM读取，使用Geooandas模块操作省份矢量裁剪。考虑到这条推文的阅读性，所有脚本代码我会在github上公开（https://github.com/KUAIDUOBAO），这里暂时只放重要的裁剪脚本：
```python
#裁剪函数
def clip(pathDir,shpdata,rasterfile):
    for i in tqdm(range(len(pathDir))):
        # 读入栅格文件
        rasterfile = files_path+"\\"+pathDir[i]
        rasterdata = rio.open(rasterfile)
        #获取栅格信息
        profile = rasterdata.profile
        #标识符
        note = pathDir[i]
        # 投影变换，使矢量数据与栅格数据投影参数一致
        shpdata = shpdata.to_crs(rasterdata.crs)
        # 按照所有矢量进行循环裁剪
        for j in range(0, len(shpdata)):
        try:
                # 获取矢量数据的features
                geo = shpdata.geometry[j]
                #获取该要素的属性信息
                data_shp_name=shpdata.全称[j]
                #文件保存位置的文件夹 各省
                data_filepath=str(data_shp_name)
                feature = [geo.__geo_interface__]
                # 通过feature裁剪栅格影像
                out_image, out_transform = rio.mask.mask(rasterdata, feature, all_touched=True, crop=True, nodata=0)
                profile.update(
                    height=out_image.shape[1],
                    width=out_image.shape[2],
                    shape=(out_image.shape[1],out_image.shape[2]),
                    nodata=0,
                    bounds=[],
                    transform=out_transform,
                    )
                # 定义要创建的目录
                mkpath = "目录名"
                # 检测目录是否存在
                mkdir(mkpath)
                # 文件名字
                name="文件名"
                with rasterio.open(name, mode='w', **profile) as dst:
                    dst.write(out_image)
        except:
                pass
                
```
使用上述脚本，可以得到省级行政图裁剪每一张影像后的裁剪结果。
![](https://gitee.com/kitmyfaceplease/image_upload/raw/master/img/202108181422954.png)
这些影像都是按照省级轮廓裁剪后的结果，单张只能覆盖一部分区域，因此需要对所有子集影像进行镶嵌。
## 3.2 DEM按省份镶嵌
需要遍历34个省份的文件夹，镶嵌使用gdal库的Warp函数。

## 3.3 数据后处理
主要是使用python脚本，对每一个省份文件夹中的结果影像进行重命名，并删除多余的切片文件。这一步需要添加一个判定函数，判定是否为镶嵌文件，
是则保留，不是则删除。

## 3.3 数据处理总结
除了上述处理过程，中间也写了数个辅助脚本，用以批量归类DEM文件、多线程处理、批量删除与重命名、匹配文件等。这部分脚本我会上传到github中，不再多做介绍。  
当然，进行DEM分省裁剪最大的困难从不是算法问题，而是巨大的数据量，单类全国30m的DEM数据解压后差不多40G。长时间的跑数据，我的笔记本电脑cpu真的可以烤肉了（跑数据过程中，温度长期稳定在80度）。  
总得来说，工作量很大，下班后都没时间碰"云顶之弈"这个游戏了。
![](https://gitee.com/kitmyfaceplease/image_upload/raw/master/img/202108191808279.gif)

# 4.结果展示
裁剪得到了34个省份的DEM，各有6张影像，三种30米分辨率、一种90米分辨率、一种250米分辨率以及一种1000米分辨率。总共166G文件，已经上传到百度云。
![](https://gitee.com/kitmyfaceplease/image_upload/raw/master/img/202108191555084.png)
![](https://gitee.com/kitmyfaceplease/image_upload/raw/master/img/202108191552359.png)
![](https://gitee.com/kitmyfaceplease/image_upload/raw/master/img/202108191753639.png)
# 5.总结
## 5.1数据总结
## 5.2下一步的计划
考虑到我的台式电脑即将组装完毕，而它的优点就是散热能力更强。因此，在工作的闲暇之余，接下来会做：  
1.使用更高精度的12.5米分辨率的ALOS PALSAR数据和15米分辨率的SRTM-X-DLR，进行一个行政区划的裁剪；
2.从34个省份的DEM裁剪，扩展到全国400个地级市、2700多个县，并进行归类。
![](https://gitee.com/kitmyfaceplease/image_upload/raw/master/img/202108192032073.png)
# 6.数据分享
请在公众号回复“DEM”，获取数据下载方式。

![](https://gitee.com/kitmyfaceplease/image_upload/raw/master/img/202108150001207.jpg)
