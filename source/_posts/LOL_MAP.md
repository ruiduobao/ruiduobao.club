# 英雄联盟世界的地图抓取与分析
# 0.背景
S11比赛正在如火如荼的举行。作为一个S3赛季的老玩家和地理人，我想从另外个角度带你了解LOL的世界。  
![](https://img-blog.csdnimg.cn/img_convert/f192633519a75b323fe5a1baafc9af64.png)
# 1.数据获取
## 1.1数据初探
英雄联盟推出了其世界地图网页版，访问地址为:https://map.leagueoflegends.com/.  
![](https://img-blog.csdnimg.cn/img_convert/d783cb3b5e6c0e9ffc095c259ba689bb.png)  
鼠标移动到图标上时，会有该地区的行政图:  
![](https://img-blog.csdnimg.cn/img_convert/9e5ebb105ea88035d4e0d0fb68f6c2b9.gif)  
放大时，该地区会变得清晰，并且有地形的起伏:  
![](https://img-blog.csdnimg.cn/img_convert/ba6c929de9d82e90680c34e194d3dade.gif)  
因此这个地图应该有一个高分辨率的地图、高分辨率的地图，类似于栅格金字塔。同时有一个DEM数据对地形进行拉伸渲染。
## 1.2地图抓取
亘古不变的F12键，查看从页面打开时的网页请求，找到png图片:
![](http://pics.landcover100.com/pics//image/20211025012629.png)


## 1.3英雄联盟地图结构解析
通过后期数据的抓取与思考，我粗略的画了一下LOL展示地图的结构:
![](https://img-blog.csdnimg.cn/img_convert/81b29169d7a5d2c8d067786776d20767.png)  
英雄联盟的地图浏览起来非常丝滑，但是这个地图的原理很复杂。不仅对DEM进行了在线渲染，而且会根据各种掩膜进行判定，比如说被海洋掩膜覆盖的区域，会生成一个“水波荡漾”的效果。  
# 2.本地地图制作
主要分为三个方面:各个城邦矢量图的制作、高分辨率影像图的合成以及DEM数据的重构。
## 2.1城邦矢量化
从官方网页结构中，我们可以获得矢量图，每一个城邦拥有一个范围栅格图(.png格式，2048*2048像素),比如说诺克萨斯城邦:
![](https://img-blog.csdnimg.cn/img_convert/db28b99e64e7fd9c5b1d020694312b3e.png)
我们将其放到arcgis，根据其边界逐一描绘:
![在这里插入图片描述](https://img-blog.csdnimg.cn/f5400b06c23e48178494e6548bf3aeee.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA6ZSQ5aSa5a6d55qE5Zyw55CG56m66Ze0,size_20,color_FFFFFF,t_70,g_se,x_16)
# 2.2 DEM数据的重构
从官网界面，我们可以看到三维的地形数据，实际上是利用了一个DEM图进行了Blender在线渲染，我们从网页中获得的DEM数据是一个Unit8像素深度的栅格图，DEM值的范围是0-255。
![在这里插入图片描述](https://img-blog.csdnimg.cn/c7f11d7e299740059b4dd75f9bea3b30.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA6ZSQ5aSa5a6d55qE5Zyw55CG56m66Ze0,size_20,color_FFFFFF,t_70,g_se,x_16)  
DEM的Value值范围为0-146，我们不知道真实的海拔高度。有没有什么办法大致知道DEM的每一个值代表的真实高度？
![在这里插入图片描述](https://img-blog.csdnimg.cn/e9357fbc8ad743cead3bb3a3f83cab37.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA6ZSQ5aSa5a6d55qE5Zyw55CG56m66Ze0,size_14,color_FFFFFF,t_70,g_se,x_16)  
考虑到德玛西亚城的河谷平原像素值在20左右，以及LOL最高峰应该超过4000米。我们假设每一个像素值代表高度为28米。因此，LOL世界最高山峰的海拔为4116米。
![在这里插入图片描述](https://img-blog.csdnimg.cn/618ac796f08f4f579ed6a5aba4c2ea66.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA6ZSQ5aSa5a6d55qE5Zyw55CG56m66Ze0,size_20,color_FFFFFF,t_70,g_se,x_16)
使用栅格计算器，重新计算LOL世界的DEM：
![在这里插入图片描述](https://img-blog.csdnimg.cn/f51447eefa304af486888e2b1d77b5e5.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA6ZSQ5aSa5a6d55qE5Zyw55CG56m66Ze0,size_12,color_FFFFFF,t_70,g_se,x_16)

在本地使用Blender对DEM数据渲染效果如下:
![在这里插入图片描述](https://img-blog.csdnimg.cn/c47b22b684ee4456be46029f61d40e53.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA6ZSQ5aSa5a6d55qE5Zyw55CG56m66Ze0,size_17,color_FFFFFF,t_70,g_se,x_16)

# 2.3 LOL高分辨地图的重制
LOL官网的地图存在两种分辨率，一种是地图总览的低分辨率，一种是放到到局部的高分辨率。高分辨率的地图需要逐个抓取。
![在这里插入图片描述](https://img-blog.csdnimg.cn/e427ab51b3c946c78362bcdfe81c4c28.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA6ZSQ5aSa5a6d55qE5Zyw55CG56m66Ze0,size_7,color_FFFFFF,t_70,g_se,x_16)
我观察到，高分辨率的地图链接是有规律的，因此写了一个脚本将这些图片都爬取下来：
![在这里插入图片描述](https://img-blog.csdnimg.cn/c45b2acc06fe4d34853fa0da7ac4cea6.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA6ZSQ5aSa5a6d55qE5Zyw55CG56m66Ze0,size_17,color_FFFFFF,t_70,g_se,x_16)
待数据下载完之后，使用我之前写好的镶嵌脚本，逐个对图片进行镶嵌:

```python
#将影像进行镶嵌
import PIL.Image as Image
import os

IMAGES_PATH = r'小切片的路径'  # 图片集地址
# IMAGES_FORMAT = ['.jpg', '.JPG']  # 图片格式
IMAGES_FORMAT = ['.png']  # 图片格式
IMAGE_SIZE = 256  # 每张小图片的大小
IMAGE_ROW = 7  # 图片间隔，也就是合并成一张图后，一共有几行
IMAGE_COLUMN = 6  # 图片间隔，也就是合并成一张图后，一共有几列
IMAGE_SAVE_PATH = 'predict.jpg'  # 图片转换后的地址

#按数字大小进行排序
files = os.listdir(IMAGES_PATH)
files.sort(key=lambda x: int(x.split('.')[0]))

# 获取图片集地址下的所有图片名称
image_names = [name for name in files for item in IMAGES_FORMAT if
               os.path.splitext(name)[1] == item]


# 简单的对于参数的设定和实际图片集的大小进行数量判断
if len(image_names) != IMAGE_ROW * IMAGE_COLUMN:
    raise ValueError("合成图片的参数和要求的数量不能匹配！")
print(image_names)

# 定义图像拼接函数
def image_compose():
    to_image = Image.new('RGB', (IMAGE_COLUMN * IMAGE_SIZE, IMAGE_ROW * IMAGE_SIZE))  # 创建一个新图
    # 循环遍历，把每张图片按顺序粘贴到对应位置上
    for y in range(1, IMAGE_ROW + 1):
        for x in range(1, IMAGE_COLUMN + 1):
            from_image = Image.open(IMAGES_PATH + image_names[IMAGE_COLUMN * (y - 1) + x - 1]).resize(
                (IMAGE_SIZE, IMAGE_SIZE), Image.ANTIALIAS)
            to_image.paste(from_image, ((x - 1) * IMAGE_SIZE, (y - 1) * IMAGE_SIZE))
    return to_image.save(IMAGE_SAVE_PATH)  # 保存新图
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/add8ed158ef047b2a9b1c61c2eb78c05.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA6ZSQ5aSa5a6d55qE5Zyw55CG56m66Ze0,size_15,color_FFFFFF,t_70,g_se,x_16)
低分辨率的图片的大小为:2048*2048，高分辨率的图片的大小为:7168*6144。

# 3.数据分析
# 3.1各个城邦的土地面积分析
使用gis的面积制表工具，统计每一个城邦的个数。
![在这里插入图片描述](https://img-blog.csdnimg.cn/a63a413b9fed4d60b67ec1be37b73f04.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA6ZSQ5aSa5a6d55qE5Zyw55CG56m66Ze0,size_19,color_FFFFFF,t_70,g_se,x_16)
由于LOL世界是否是一个球形星球，也不知道该地图是否有投影变形。我们暂且用像元个数代表每个城邦的面积。诺克萨斯的面积不仅大，还横跨三个大的区域，实力不容小觑。
![在这里插入图片描述](https://img-blog.csdnimg.cn/911650c806084c849d1ce7744bbfcd83.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA6ZSQ5aSa5a6d55qE5Zyw55CG56m66Ze0,size_20,color_FFFFFF,t_70,g_se,x_16)
虽然诺克萨斯的面积是德玛西亚的四倍，但是德玛西亚**永不屈服**。
![在这里插入图片描述](https://img-blog.csdnimg.cn/7c470a3805e84d19b4bc919414bde7a9.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA6ZSQ5aSa5a6d55qE5Zyw55CG56m66Ze0,size_10,color_FFFFFF,t_70,g_se,x_16)
# 3.2各个联邦的地形分析
将LOL世界的海拔进行重分类:  
![在这里插入图片描述](https://img-blog.csdnimg.cn/37d32b28b062473cb1a9448690432627.png)  
海拔1960米以上就可以视为山脉，可以看看各个城邦的山脉分布图:  
![在这里插入图片描述](https://img-blog.csdnimg.cn/fc3dc57cd9d44841be9093210258a915.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA6ZSQ5aSa5a6d55qE5Zyw55CG56m66Ze0,size_15,color_FFFFFF,t_70,g_se,x_16)  
从这个图，我们能猜测出克萨斯和德玛西亚打仗最激烈的地方，所谓狭路相逢勇者胜:  
![在这里插入图片描述](https://img-blog.csdnimg.cn/adeb465786b845b7b45ec680fdeaad33.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA6ZSQ5aSa5a6d55qE5Zyw55CG56m66Ze0,size_15,color_FFFFFF,t_70,g_se,x_16)

# 4.写到最后
（1）该推文只做学习使用，请勿作为非法用途，任何非法用途与本人无关。

（2）祝愿 EDG 取得好成绩，LPL 加油！