### canvas 

#### paint

##### **ANTI_ALIAS_FLAG 属性可以得到平滑的图形。**

```
Paint p = new Paint(Paint.ANTI_ALIAS_FLAG);
    //或者
Paint p = new Paint();
p.setAntiAlias(true);
```

正如你看到的，设置 ANTI_ALIAS_FLAG 属性可以产生平滑的边缘。**这里它能起作用是因为默认下每当 onDraw 被调用时系统先将 Canvas 清空然后重绘所有东西。**当我在下文详细讨论 ANTI_ALIAS_FLAG 的工作原理时， 你会意识到这段信息的重要性

##### **ANTI_ALIAS_FLAG 是怎么工作的**

ANTI_ALIAS_FLAG **通过混合前景色与背景色来产生平滑的边缘**。背景色是透明的而前景色是红色的，**ANTI_ALIAS_FLAG 通过将边缘处像素由纯色逐步转化为透明来让边缘看起来是平滑的**

**为什么设置了 ANTI_ALIAS_FLAG 后你们图形的边缘还是十分粗糙**

- 避免重绘。
- 在重绘前清空你的 Bitmap。

添加一行代码让它在每次**重绘前先清空 Bitmap**。当然，如果你觉得纯色更加符合你的需求的话，你也可以不用每次都清空 Bitmap

​	

```java
  @Override
    protected void onDraw(Canvas canvas) {
        super.onDraw(canvas);
        if (bitmap == null) {
            bitmap = Bitmap.createBitmap(200,
                                         200,
                                         Bitmap.Config.ARGB_8888);
            bitmapCanvas = new Canvas(bitmap);
        }
        bitmapCanvas.drawColor(
                  Color.TRANSPARENT,
                  PorterDuff.Mode.CLEAR); //this line moved outside if
        drawOnCanvas(bitmapCanvas);
        canvas.drawBitmap(bitmap, mLeftX, mTopY, p);
    }

    protected void drawOnCanvas(Canvas canvas) {
        canvas.drawCircle(mLeftX + 100, mTopY + 100, 100, p);
    }
```

![](01.jpg)

##### setShadowLayer

```java
public void setShadowLayer(float radius, float dx, float dy, int color)
radius:模糊半径，radius越大越模糊，越小越清晰，但是如果radius设置为0，则阴影消失不见
dx:阴影的横向偏移距离，正值向右偏移，负值向左偏移
dy:阴影的纵向偏移距离，正值向下偏移，负值向上偏移
color: 绘制阴影的画笔颜色，即阴影的颜色（对图片阴影无效）

```

##### view的硬件加速

**硬件加速能够带来性能提升**，android为什么要弄出这么多级别的控制，而不是默认就是全部硬件加速呢？**原因是并非所有的2D绘图操作支持硬件加速**，如果您的程序中使用了**自定义视图或者绘图调用，程序可能会工作不正常**。如果您的程序中只是用了标准的视图和Drawable，放心大胆的开启硬件加速吧！具体是哪些绘图操作不支持硬件加速呢?以下是已知不支持硬件加速的绘图操作：

- Canvas
  - [clipPath()](http://developer.android.com/reference/android/graphics/Canvas.html#clipPath(android.graphics.Path))
  - [clipRegion()](http://developer.android.com/reference/android/graphics/Canvas.html#clipRegion(android.graphics.Region))
  - [drawPicture()](http://developer.android.com/reference/android/graphics/Canvas.html#drawPicture(android.graphics.Picture))
  - [drawPosText()](http://developer.android.com/reference/android/graphics/Canvas.html#drawPosText(char[], int, int, float[], android.graphics.Paint))
  - [drawTextOnPath()](http://developer.android.com/reference/android/graphics/Canvas.html#drawTextOnPath(char[], int, int, android.graphics.Path, float, float, android.graphics.Paint))
  - [drawVertices()](http://developer.android.com/reference/android/graphics/Canvas.html#drawVertices(android.graphics.Canvas.VertexMode, int, float[], int, float[], int, int[], int, short[], int, int, android.graphics.Paint))
- Paint
  - [setLinearText()](http://developer.android.com/reference/android/graphics/Paint.html#setLinearText(boolean))
  - [setMaskFilter()](http://developer.android.com/reference/android/graphics/Paint.html#setMaskFilter(android.graphics.MaskFilter))
  - [setRasterizer()](http://developer.android.com/reference/android/graphics/Paint.html#setRasterizer(android.graphics.Rasterizer))



另外还有一些绘图操作，开启和不开启硬件加速，效果不一样：

- Canvas
  - [clipRect()](http://developer.android.com/reference/android/graphics/Canvas.html#clipRect(android.graphics.Rect))： `XOR`, `Difference和ReverseDifference裁剪模式被忽略，3D变换将不会应用在裁剪的矩形上。`
  - [drawBitmapMesh()](http://developer.android.com/reference/android/graphics/Canvas.html#drawBitmapMesh(android.graphics.Bitmap, int, int, float[], int, int[], int, android.graphics.Paint))：colors数组被忽略
  - [drawLines()](http://developer.android.com/reference/android/graphics/Canvas.html#drawLines(float[], android.graphics.Paint))：反锯齿不支持
  - [setDrawFilter()](http://developer.android.com/reference/android/graphics/Canvas.html#setDrawFilter(android.graphics.DrawFilter))：可以设置，但无效果
  - 
- Paint
  - [setDither()](http://developer.android.com/reference/android/graphics/Paint.html#setDither(boolean))： 忽略
  - [setFilterBitmap()](http://developer.android.com/reference/android/graphics/Paint.html#setFilterBitmap(boolean))：过滤永远开启
  - [setShadowLayer()](http://developer.android.com/reference/android/graphics/Paint.html#setShadowLayer(float, float, float, int))：只能用在文本上
- ComposeShader
  - [ComposeShader](http://developer.android.com/reference/android/graphics/ComposeShader.html)只能包含不同类型的shader (比如一个[BitmapShader](http://developer.android.com/reference/android/graphics/BitmapShader.html)和一个[LinearGradient](http://developer.android.com/reference/android/graphics/LinearGradient.html)，但不能是两个[BitmapShader](http://developer.android.com/reference/android/graphics/BitmapShader.html)实例)
  - [ComposeShader](http://developer.android.com/reference/android/graphics/ComposeShader.html)不能包含ComposeShader

这里有一点需要非常注意的是**setShadowLayer只有文字绘制阴影支持硬件加速**，其它都不支持硬件加速，所以为了方便起见，我们需要在自定义控件中禁用硬件加速。

```java
//对单独的View在运行时阶段禁用硬件加速
if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.HONEYCOMB) {
            setLayerType(LAYER_TYPE_SOFTWARE, null);
}
```

### Property Animator

​	View Animation 包括 **Tween Animation（补间动画）和 Frame Animation(逐帧动画)；**

​	Property Animator 包括 **ValueAnimator 和 ObjectAnimation；**

##### **view动画和属性动画不同点**

1.  **引入时间不同**

   ​	View Animation 是 API Level 1 就引入的。Property Animation 是 API Level 11 引入的，即 Android 3.0 才开始有 Property Animation 相关的 API

2. **所在包名不同**

    View Animation 在包 android.view.animation 中。而 Property Animation API 在包 android.animation 中

   1. **动画类的命名不同**

      View Animation 中动画类取名都叫 XXXXAnimation,而在 Property Animator 中动画类的取名则叫 XXXXAnimator

##### 为什么引入 Property Animator(属性动画)

  问题：如何利用补间动画来将一个控件的背景色在一分钟内从绿色变为红色？**这个效果想必没办法仅仅通过改变控件的渐入渐出、移动、旋转和缩放来实现吧**，而这个效果是可以通过 Property Animator 完美实现的 ？

**不同点一：Property Animator 能实现补间动画无法实现的功能**

**不同点二**：**View Animation 仅能对指定的控件做动画，而 Property Animator 是通过改变控件某一属性值来做动画的**

​	假设我们将一个按钮从左上角利用补间动画将其移动到右下角，**在移动过程中和移动后，这个按钮都是不会响应点击事件的**。这是为什么呢**？因为补间动画仅仅转变的是控件的显示位置而已，并没有改变控件本身的值**

​	View Animation 的动画实现是通过其 Parent View 实现的，在 View 被 drawn 时 Parents View 改变它的绘制参数，**这样虽然 View 的大小或旋转角度等改变了，但 View 的实际属性没变**，所以有效区域还是应用动画之前的区域

**不同点三：补间动画虽能对控件做动画，但并没有改变控件内部的属性值。而 Property Animator 则是恰恰相反，Property Animator 是通过改变控件内部的属性值来达到动画效果的**

##### ValueAnimator使用

1. **创建 ValueAnimator 实例**

   ```java
   ValueAnimator animator = ValueAnimator.ofInt(0,400);  
   animator.setDuration(1000);  
   animator.start(); 
   ```

   这里我们利用 ValueAnimator.ofInt 创建了一个值从 0 到 400 的动画，动画时长是 1s，然后让动画开始。从这段代码中可以看出，**ValueAnimator 没有跟任何的控件相关联，那也正好说明 ValueAnimator 只是对值做动画运算，而不是针对控件的，我们需要监听 ValueAnimator 的动画过程来自己对控件做操作**。

2. **添加监听** 上面的三行代码，我们已经实现了动画，下面我们就添加监听

   ```java
   ValueAnimator animator = ValueAnimator.ofInt(0,400);  
   animator.setDuration(1000);  
   
   animator.addUpdateListener(new ValueAnimator.AnimatorUpdateListener() {  
       @Override  
       public void onAnimationUpdate(ValueAnimator animation) {  
           int curValue = (int)animation.getAnimatedValue();  
           Log.d("test","curValue:"+curValue);  
       }  
   });  
   animator.start(); 
   ```

   - ValueAnimator 只负责对指定的数字区间进行动画运算
   - 我们需要对运算过程进行监听，然后自己对控件做动画操作

    

   ```java
   private void doAnimation(){  
       ValueAnimator animator = ValueAnimator.ofInt(0,400);  
       animator.setDuration(1000);  
   
       animator.addUpdateListener(new ValueAnimator.AnimatorUpdateListener() {  
           @Override  
           public void onAnimationUpdate(ValueAnimator animation) {  
               int curValue = (int)animation.getAnimatedValue();  
               tv.layout(curValue,curValue,curValue+tv.getWidth(),curValue+tv.getHeight());  
           }  
       });  
       animator.start();  
   }  
   ```

   在监听过程中，通过 layout 函数来改变 textview 的位置。这里注意了，**我们是通过 layout 函数来改变位置的，我们知道 layout 函数在改变控件位置时是永久性的，**即通过更改控件 left,top,right,bottom 这四个点的坐标来改更改坐标位置的，而不仅仅是从视觉上画在哪个位置，所以通过 layout 函数更改位置后，控件在新位置是可以响应点击事件的

##### 常用方法

1. **ofInt 与 ofFloat**

   ```java
   public static ValueAnimator ofInt(int... values)  
   public static ValueAnimator ofFloat(float... values) 
   ```

   他们的参数类型都是可变参数长参数，所以我们可以传入任何数量的值；传进去的值列表，就表示动画时的变化范围**；比如 ofInt(2,90,45)就表示从数值 2 变化到数字 90 再变化到数字 45**；所以我们传进去的数字越多，动画变化就越复杂。从参数类型也可以看出 ofInt 与 ofFloat 的唯一区别就是传入的数字类型不一样，ofInt 需要传入 Int 类型的参数，而 ofFloat 则表示需要传入 Float 类型的参数。 

 2,**常用函数**

```java
/** 
 * 设置动画时长，单位是毫秒 
 */  
ValueAnimator setDuration(long duration)  
/** 
 * 获取 ValueAnimator 在运动时，当前运动点的值 
 */  
Object getAnimatedValue();  
/** 
 * 开始动画 
 */  
void start()  
/** 
 * 设置循环次数,设置为 INFINITE 表示无限循环 
 */  
void setRepeatCount(int value)  
/** 
 * 设置循环模式 
 * value 取值有 RESTART，REVERSE， 
 */  
void setRepeatMode(int value)  
/** 
 * 取消动画 
 */  
void cancel() 
```

**setRepeatCount()、setRepeatMode()、cancel（）**

 setRepeatCount(int value)**用于设置动画循环次数,设置为 0 表示不循环**，设置为 **ValueAnimation.INFINITE** 表示无限循环。 cancel()用于取消动画 我们着重说一下 setRepeatMode：

```java
/** 
 * 设置循环模式 
 * value 取值有 RESTART，REVERSE 
 */  
void setRepeatMode(int value) 
```

setRepeatMode(int value)**用于设置循环模式**，**取值为 ValueAnimation.RESTART 时,表示正序重新开始，当取值为 ValueAnimation.REVERSE 表示倒序重新开始**



##### 两个监听器：

```java
/** 
 * 监听器一：监听动画变化时的实时值 
 */  
public static interface AnimatorUpdateListener {  
    void onAnimationUpdate(ValueAnimator animation);  
}  
//添加方法为：public void addUpdateListener(AnimatorUpdateListener listener)  
/** 
 * 监听器二：监听动画变化时四个状态 
 */  
public static interface AnimatorListener {  
    void onAnimationStart(Animator animation);  
    void onAnimationEnd(Animator animation);  
    void onAnimationCancel(Animator animation);  
    void onAnimationRepeat(Animator animation);  
}  
//添加方法为：public void addListener(AnimatorListener listener)  
```

##### **取消监听**

```java
/** 
 * 移除 AnimatorUpdateListener 
 */  
void removeUpdateListener(AnimatorUpdateListener listener);  
void removeAllUpdateListeners();  
 /** 
  * 移除 AnimatorListener 
  */  
void removeListener(AnimatorListener listener);  
void removeAllListeners();
```

##### 其他函数

```java
/** 
 * 延时多久时间开始，单位是毫秒 
 */  
public void setStartDelay(long startDelay)  
/** 
 * 完全克隆一个 ValueAnimator 实例，包括它所有的设置以及所有对监听器代码的处理 
 */  
public ValueAnimator clone() 
```

setStartDelay(long startDelay)非常容易理解，**就是设置多久后动画才开始**。 但 clone()这个函数就有点难度了；首先是什么叫克隆。就是完全一样！注意是完全一样！就是复制出来一个完全一样的新的 ValueAnimator 实例出来。对原来的那个 ValueAnimator 是怎么处理的，在这个新的实例中也是全部一样的。 我们来看一个例子来看一下，什么叫全部一样.



#### onMeasure





