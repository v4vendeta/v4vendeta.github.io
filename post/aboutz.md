# About Z

### z_buffer为什么是非线性的

引用[humus:A couple of notes about Z](http://www.humus.name/index.php?page=News&ID=255)的一句话

While W is linear in view space it's not linear in screen space. Z, which is non-linear in view space, is on the other hand linear in screen space

### z_buffer精度问题

在投影阶段，我们将view space中的z值映射成了关于1/z的函数
得到一个如下形式的函数图像
![](https://pic.downk.cc/item/5f3a690214195aa594a499b7.png)


在近平面时，z''的精度较高，深度缓冲有较好的表现，我们在深度缓冲时可以很容易判断出物体的前后

在远平面附近，同样z值相差为1，z''的值相差很小，甚至会计算出相同的深度值，这时就无法分辨哪个物体在前，哪个物体在后，这就是depth precision error(z fighting)

![](https://pic.downk.cc/item/5f3a690214195aa594a499b4.png)



**解决方法：**

- 将znear设置的尽可能小可以改善z''的分布，将更多的变化空间留给远平面

- 使用无限远平面

- 用精度更高的数据类型存储

- 使投影矩阵与其他矩阵分开，并将其应用于顶点着色器中的单独操作中，而不是将其组合到视图矩阵中- 

P.S.
>Differentiating depth between close objects is more important than differentiating depth between far objects


<!-- 
# integer depth buffer

假设我们用integer类型存储z值，那么z精确度是均匀分布的，定义映射函数$f(x)$

我们设depthbuffer精确度为s（depthbuffer中两个点z值差的最小值，int类型可设为1），view space精确度$c$

得到$f(z) - f(z+C)=s$
$f(x) = he^{x}$ -->

### reverse-z


在上面的描述中，zbuffer的值应该是近平面趋近0，远平面趋近1（DirectX），渲染出来的效果是近平面颜色深，远平面颜色浅，最近看到一篇博客在分析GTA5渲染时，zbuffer的值正好相反，是用了reverse-z的技术

![](https://images2015.cnblogs.com/blog/503123/201601/503123-20160112092820882-537489601.jpg)

(Nvidia官方的案例是DirectX为例的)

DirectX中z的投影变换（z>0，z映射到[0,1]）
$z''=\frac{z_{far}}{z_{far}-z_{near}}-\frac{z_{far}\ z_{near}}{z_{far}-z_{near}}\frac{1}{zp}$

标准的zbuffer精度曲线

![](https://developer.nvidia.com/sites/default/files/akamai/gameworks/blog/Depthprecision/graph4.jpg)
<center style="font-size:14px;color:#C0C0C0;text-decoration:underline">DirectX</center> 

reverse-z的做法是，将近平面映射到z=1，将远平面映射到z=0，使得浮点的准对数分布在某种程度上取消了1/z非线性，使我们在近平面上的精度与整数深度缓冲区相似，并且极大地提高了其他地方的精度

$z''=\frac{z_{near}}{z_{near}-z_{far}}-\frac{z_{near}\ z_{far}}{z_{near}-z_{far}}\frac{1}{zp}$

![](https://developer.nvidia.com/sites/default/files/akamai/gameworks/blog/Depthprecision/graph5.jpg)

但在OpenGL中，由于z值映射到[-1,1]，远平面部分的数据精度被破坏，无法直接使用这种方法

![](https://developer.nvidia.com/sites/default/files/akamai/gameworks/blog/Depthprecision/graph6.jpg)
<center style="font-size:14px;color:#C0C0C0;text-decoration:underline">OpenGL</center> 



### multiple frusta

rtr4中另一种减少z fighting的做法，渲染多个frustum

Cozzi proposes to use multiple frusta, which can improve accuracy to effectively any desired rate. The view frustum is divided in the depth direction into
several non-overlapping smaller sub-frusta whose union is exactly the frustum. The
sub-frusta are rendered to in back-to-front order. First, both the color and depth
buffers are cleared, and all objects to be rendered are sorted into each sub-frusta that
they overlap. For each sub-frusta, its projection matrix is set up, the depth buffer is
cleared, and then the objects that overlap the sub-frusta are rendered.

### early-z

在late depth test中，是在fragmentshader后计算z值，做深度测试，early-z是在fragmentshader之前计算z值，并根据可视性discard掉多余的像素，这一功能已经集成在图形api中

**reference:**

https://developer.nvidia.com/content/depth-precision-visualized
http://www.adriancourreges.com/blog/2015/11/02/gta-v-graphics-study/
https://www.khronos.org/opengl/wiki/Depth_Buffer_Precision
https://www.gamedev.net/forums/topic.asp?topic_id=539378
http://www.humus.name/index.php?page=News&ID=255