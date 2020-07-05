@[TOC]( UGUI三种实现规则与不规则图形Button精准点击效果的策略）)

<br>


# (一) 为什么需要精准点击？
<br>

&ensp;&ensp;&ensp;<font size=4>图 1：没有进行点击效果精准化时的圆形button点击效果(绿色button)：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190912095822916.gif)
<br>
&ensp;&ensp;&ensp;<font size=4>图2：实现点击效果精准化时的圆形button点击效果(蓝色button)：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190912100113486.gif)



 **&ensp; &ensp; &ensp;许多游戏项目之中我们的技能，或者是一些设置按钮，这些功能按钮可能不是像正常的矩形形状 (例如圆形) ，但是他们的响应区域依旧是 rect矩形区域(例如上图1)，这时我们需要一些策略进行实现 点击效果精准化 (实现效果如上图2)。** 




---



<br>
<br>

#  (二) 3种实现规则与不规则图形Button精准点击效果的方案
<br>
<br>

<font size=5 color=balck>**&ensp;&ensp;了解了为什么需要点击精准化之后，我们接下来进行介绍3种实现规则与不规则图形Button精准点击效果的方案:**

<br>


## <font color=blue>1.&ensp;所有图形button都适用的官方解决方案。</font>
<br>

### <font color=red> 实现原理
  <font color=black> &ensp;&ensp;&ensp;&ensp;官方解决方案为自己需要制作自己的button的形状图片，之后由于形状图片区域与图片矩形区域有留白区域（**如图三**），这些留白区域
   都会在unity中显示为透明，因此官方根据我们点击的button的点对应于图片位置的这个透明度设置了我们可利用的参数 **（alphaHitTestMinimumThreshold），当我们点击button区域的透明度小于这个参数值，我们的点击将没有任何响应，只有透明度大于这个参数才会有响应。**
   
   <br>
   
  图三：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190912102247926.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2Nob25nemlfZGFpbWE=,size_16,color_FFFFFF,t_70)

<br>

### <font color=red> 实现步骤
<br>

**（1）首先你想做成的button形状与Texture的图形必须是匹配的（不能你想做成一个圆形的，还用的是画面全铺的矩形状texture），图片的图形区域与矩形图片区域有留白(作为Alpha 通道)，之后勾选图片的Read/Write Enabled属性。**

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190912104943188.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2Nob25nemlfZGFpbWE=,size_16,color_FFFFFF,t_70)
<br>
**（2）&ensp;将图片改为Sprite格式，之后赋予button即可，之后编写代码设置GetComponent<Image>().alphaHitTestMinimumThreshold
         设置之后unity就知道了在这个button范围内区域如果点击想响应的话，这个区域的的透明度必须大于alphaHitTestMinimumThreshold的值，小于则不进行响应。**
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190912110118743.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2Nob25nemlfZGFpbWE=,size_16,color_FFFFFF,t_70)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190912110131236.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2Nob25nemlfZGFpbWE=,size_16,color_FFFFFF,t_70)
<br>
**（3）**<font size=4.5 color=black>  **整体步骤演示Gif**：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190912111011399.gif)


### <font color=red> 优点
   **&ensp;&ensp;实现起来十分简单！！！**
### <font color=red> 缺点
&ensp;&ensp;**想获取并进行更改图片的这个参数，需要勾上Texture的Read/Write Enabled参数，当勾上之后，unity就会拷贝一份texture放入内存中，这时也就是一张图片占了两张图片的内存，同时，该image不利于层级合并，增加drawcall(SetPass calls),显然对整个项目影响都很大。**

---
<br>
<br>
<br>
<br>

## <font color=blue>2.所有图形都适用的Ray-Crossing算法策略。</font>
<br>

### <font color=red> 实现原理

&ensp;&ensp;**我们拿上节博客我们制作的CircleImage组件(利用重新传入顶点信息将矩形画面平铺图片改为圆形)利用Ray-Crossing算法实现精准点击策略，我们需要重写Image的public override bool IsRaycastLocationValid(Vector2 screenPoint, Camera eventCamera)方法，之后利用Ray-Crossing算法判断点击区域是否在CircleImage的范围内。**

### <font color=red>重点介绍：

 **public override bool IsRaycastLocationValid(Vector2 screenPoint, Camera eventCamera)
&ensp;&ensp;&ensp;&ensp;给定一个屏幕点和相加，返回该点击是否有效，之前是image的rect范围内就会响应，我们需要重写这个方法，让Ray-Crossing算法判断是否点击有效。**

**Ray-Crossing算法实现：
&ensp;&ensp;&ensp;&ensp;大概思路是从指定点p发出一条射线(我们以点击点向右水平方向发射)，与多边形相交，假若交点个数是奇数，说明点p落在多边形内，交点个数为偶数说明点p在多边形外。算法结论乍看难以理解，但在逻辑上是可证的，之后我们筛选可以存在交点的边，比如 p 的y为1，我们不可能选择两个构成边的顶点的y值都大于1或者都小于1的，因为这样没有意义，根本不会有交点。筛选出的边，我们利用  y=kx+b 来表达。 利用两顶点坐标，求出表达式的k，b值，之后求 y=p.y这条直线与筛选出的边的交点，求出交点的x值，如果x值大于p的x值，说明该交点有效，小于p的x值，该交点无效，因为由p向右的延长线为射线，不会与x值小于p的x值有交点。**
<font>

 ![在这里插入图片描述](https://img-blog.csdnimg.cn/20190912140935543.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2Nob25nemlfZGFpbWE=,size_16,color_FFFFFF,t_70)![](https://img-blog.csdnimg.cn/20190912141012900.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2Nob25nemlfZGFpbWE=,size_16,color_FFFFFF,t_70)


### <font color=red>  实现步骤
 **首先介绍几个我写的方法，之后利用这些方法进行演示：**
 
**1.&ensp;&ensp;IsRaycastLocationValid(Vector2 screenPoint, Camera eventCamera) ：重写的检测点击是否有效的方法
2.&ensp;&ensp;ClickPointIsVaild(Vector2 localPoint)                                                    ：判断点击点是否有效（通过判断Ray-Crossing算法得到的交点数量的奇偶性 （奇数有效，偶数无效））
3.&ensp;&ensp;GetCrossPointNum(Vector2 localPoint,List<UIVertex> vertexList )        ：通过点击点与多边形顶点集合得到Ray-Crossing算法得到的交点数量
4.&ensp;&ensp;IsYInRange(Vector2 localPoint,UIVertex vertex1,UIVertex vertex2)     ：我们利用Ray-Crossing算法求交点个数时，需要进行筛选边，也可以说筛选组成边的相邻的两个顶点，筛选边的条件是   (vertex1.y>=localPoint.y&&vertex2.y<=localPoint.y) 或者是(vertex2.y>=localPoint.y&&vertex1.y<=localPoint.y) ，这样的边才是合格的。**
<br>
**<font color=red>步骤：</font>**

**1.&ensp;&ensp;由screenPoint转为localPoint(screenPoint是在屏幕上的位置点，localPoint是相对于rectTransform的Pivot的位置)，之后将得到的localPoint传入ClickPointIsVaild(Vector2 localPoint) 进行判断。**
![在这里插入图片描述](https://img-blog.csdnimg.cn/2019091213210398.png)
**2.&ensp;&ensp;ClickPointIsVaild用于判断点击点是否有效（通过判断Ray-Crossing算法得到的交点数量的奇偶性 （奇数有效，偶数无效）），直接将点击点的信息，与多边形顶点集合信息传入到GetCrossPointtNum(Vector2 localPoint,List<UIVertex> vertexList ) 进行得到有效交点数，之后判断有效交点数的奇偶性，奇数返回true，偶数返回false。**
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190912132130647.png)
**3.&ensp;&ensp;GetCrossPointtNum(Vector2 localPoint,List<UIVertex> vertexList ) 方法中：我们先将构成多边形边的相邻顶点依次遍历，之后我们利用IsYInRange方法筛选可以存在交点的边，比如点击点的y为1，我们不可能选择两个构成边的顶点的y值都大于1或者都小于1的，因为这样没有意义，根本不会有交点。筛选出的边，我们利用  y=kx+b 来表达。 利用两顶点坐标，求出表达式的k，b值，之后求 y=p.y这条直线与筛选出的边的交点，求出交点的x值，如果x值大于p的x值，说明该交点有效，小于p的x值，该交点无效，因为由p向右的延长线为射线，不会与x值小于p的x值有交点。**

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190912134549711.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2Nob25nemlfZGFpbWE=,size_16,color_FFFFFF,t_70)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190912134638327.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2Nob25nemlfZGFpbWE=,size_16,color_FFFFFF,t_70)
<br>
**4.&ensp;&ensp;整体过程展示Gif：**
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190912141113454.gif)





<br>

### <font color=red>优点
**&ensp;&ensp;没有任何的性能问题影响，而且点击非常准确，你想做成的button形状与图片的图形不匹配也可以。(**博主相对推荐这一种，毕竟我认为多重写这种底层源码会增强自己的技术，多一点创造力**)**

### <font color=red> 缺点
&ensp;&ensp;**需要重新绘制顶点，记录顶点，面信息，制作对应图案的button，之后也要重写image中的方法，相对来说代码量较多。**

---
<br>
<br>



## <font color=blue>3.圆形除外(圆形实现不易精确)的其他图形都适用的绘制碰撞体方案</font>
<br>

### <font color=red>实现原理
 **1.&ensp;利用unity的2d碰撞体组件PolygonCollider2D，之后编辑器绘制PolygonCollider2D碰撞体的范围包裹住我们的图形button;
 2.&ensp;之后我们依旧重写 Image的IsRaycastLocationValid(Vector2 screenPoint, Camera eventCamera)方法,将判断点是否有效方式写成我们的方案。
 3.&ensp;利用RectTransformUtility.ScreenPointToWorldPointInRectangle(rectTransform, screenPoint, eventCamera, out tempPos)返回一个世界坐标点
4.&ensp;之后利用PolygonCollider2D.OverlapPoint(tempPos) 判断是否这个点在碰撞体上，在的话这个就响应，不在则不响应。**
<br>
<br>
### <font color=red> 实现步骤

**1.&ensp;给图形Button添加Polygon Collider2D组件，并进行EditCollider，（我建议大家不要进行适用这种方式实现圆形button精准点击，很难绘制准确）**

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190912150454131.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2Nob25nemlfZGFpbWE=,size_16,color_FFFFFF,t_70)

**2.&ensp;编写CustomImage脚本，继承Image，重写IsRaycastLocationValid(Vector2 screenPoint, Camera eventCamera)，利用PolygonCollider2D.OverlapPoint(tempPos)返回的值判断是否点击有效。**
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190912151101625.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2Nob25nemlfZGFpbWE=,size_16,color_FFFFFF,t_70)

**3.&ensp;整体过程演示Gif**
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190912155657527.gif)


### <font color=red> 优点
**&ensp;&ensp;代码量少，功能易实现，只需绘制好碰撞体即可。**

### <font color=red>  缺点
**&ensp;&ensp;1.需要自己绘制碰撞体的形状，可能绘制过程绘制不太准确。
&ensp;&ensp;2.仍需要你想做成的button形状与图片的图形匹配。
&ensp;&ensp;3.性能问题消耗了一个额外的碰撞体组件(影响不大)。**
<br>
<br>
<br>
<br>


# (三) 项目工程链接地址(GitHub)
<br>

求github小星星哈，谢谢啦?
[CircleImage组件项目工程链接](https://github.com/HanxianshengGame/AccuracyOfClickingRegularAndIrregularPolygonButtons)(https://github.com/HanxianshengGame/AccuracyOfClickingRegularAndIrregularPolygonButtons)


**参考 scene Example 2.2与Script Example 2.2 进行学习：**


![在这里插入图片描述](https://img-blog.csdnimg.cn/20190910155020635.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2Nob25nemlfZGFpbWE=,size_16,color_FFFFFF,t_70)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190910155042423.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2Nob25nemlfZGFpbWE=,size_16,color_FFFFFF,t_70)

<br>
<br>

  --------------------------------------------------------------------------我是有底线的----------------------------------------------------------------------------------------
  
 &ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;感谢能够观看博客的各位Unity开发爱好者们，有问题发表评论呐，*★,°*:.☆(￣▽￣)/$:*.°★* 。                                                      
