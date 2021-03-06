# 非真实感渲染

## Intro 

Photorealistic Rendering的目的是渲染出相片级别的图像，而Non-Photorealistic Rendering则主要用于生成风格化的图像，如卡通，水墨，铅笔画等，本文简要总结了Real-time Rendering 4th Edition 15章 Non-Photograph Rendering中介绍的部分方法，包括Cel Shading和Outline Shading

## Toon/Cel Shading

卡通渲染是NPR领域中应用最广的技术，因其生成的图像风格与卡通十分相似，被广泛用在游戏/影视作品中

在Phong Illumination Model中，我们用$C=dot(light\_dir,normal)$来计算着色的$diffuse$分量，per pixel计算可以获得平滑的着色，Cel Shading的做法是，当C大于/小于某个值的时候，着不同的颜色，这样就能获得色块分明的效果

![](https://upload.wikimedia.org/wikipedia/commons/b/b7/Toon-shader.jpg)


## Outline Shading



1. viewspace下dot(normal,(0,0,1))
    这个值越小，物体表面就越趋近边缘
    > 缺点：不能产生宽度统一的边缘，因为是根据normal计算，cube会失效，距离过远时normal不精确效果变差

2. backface 渲染成黑色并加上bias，再渲染frontface，bias部分就会呈现为outline


    **bias的方法**
      - translate固定值
  
        ![](https://www.realtimerendering.com/figures/thumb/RTR4.15.07.jpg)

        > 缺点：边缘宽度不统一，由角度决定

   - 三角形三条边向外扩张，距离由到到eye的距离决定，再连接三条边（右图），以防止顶点偏离过多

        ![](https://www.realtimerendering.com/figures/thumb/RTR4.15.08.jpg)

        使用这个技术的游戏有

        <img src="https://ubistatic19-a.akamaihd.net/ubicomstatic/en-us/global/media/pop0_ss6_desktop_164581.jpg" width="200" height="200">
        <img src="https://multiplayer.net-cdn.it/thumbs/images/2014/07/07/senza_titolo-1_copia_jpg_640x0_watermark-small_q85.jpg" width="200" height="200">

   - 顶点沿法线方向扩张，扩张的大小通过z值不同计算（笔者猜测是离屏幕越近，偏移越小，反之亦然）
        这种方法在vs中计算很快，使用广泛
        
        ![](https://www.realtimerendering.com/figures/thumb/RTR4.15.10.jpg)

>缺点：cube同样渲染有问题
frontface和backface都要要渲染一遍，效率较低，semi-tranparent物体渲染需要特殊处理

还有一些图像处理的方法，主要做法是对一些特征进行滤波获得边缘

1. 第一个pass正常渲染，记录物体id， 在 postpocessing pass中采样周围的像素，如果有id不同的就渲染成black

2. depth/normal buffer中不连续的部分往往是contour edge，ps中存储这些buffer，再用滤波的方法找出边缘
   
   [Making Concept Art Real for Borderlands](http://stylized.realtimerendering.com/)中改进了sobel算子，得到1-pixel宽度的边缘
此外还有image processing的方法
    > 缺点：depth差异过小时效果不好

获得边缘后可以用dilation/erosion operator来thicken/thinning边缘

## 其他

此外，Real-time Rendering中还介绍了包括，风格化渲染，线条简化，文字渲染等，在此不予一一介绍

---

### Reference

Real-Time Rendering 4th Edition
https://zhuanlan.zhihu.com/p/84075550
[prince-of-persia(GDC 2009)](https://gdcvault.com/play/1374/Illustrative-Rendering-of-PRINCE-OF)
[AFRO SAMURAI (GDC 2009)](https://www.gdcvault.com/play/1709/Style-in-Rendering-The-History)