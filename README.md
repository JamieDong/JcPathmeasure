
# PathMeasure迷径追踪动画
### 简介
- 顾名思义，PathMeasure是一个用来测量Path的类。
- 初始化  
`PathMeasure mPathMeasure = mPathMeasure = new PathMeasure();`——创建PathMeasure对象
`mPathMeasure.setPath(mPath, true);`——设置关联Path
`mPathMeasure.setPath(Path path, boolean forceClosed);`——设置关联Path
- forceClosed参数
forceClosed参数对绑定的Path不会产生任何影响。
从字面意思上去理解，forceClosed就是代表这样一个路径是否强制闭合。
forceClosed参数对PathMeasure的测量结果有影响。
如果是一个未封闭的矩形，forceClosed设置为true强行闭合起来，那么PathMeasure测量结果就是一个完整的矩形周长。
- 常用API
getLength()——获取计算的路径长度
getSegment()——从字面意思来看，就是获取到一个路径的片段
getPosTan()——用于获取路径上某点的坐标及其切线的坐标
### 简单的Loading加载动画效果
![](https://upload-images.jianshu.io/upload_images/11184437-9a8e43710474c8d2.gif?imageMogr2/auto-orient/strip)

创建一个自定view`PathTracingView`
定义几个变量
```
    private Path mDst;
    private Path mPath;//
    private Paint mPaint;//画笔对象
    private float mLength;///路径长度
    private float mAnimValue;
```
首先先画出一个圆
`mPath.addCircle(400, 400, 100, Path.Direction.CW);`

然后与PathMeasure进行关联
`mPathMeasure.setPath(mPath, true);`

然后通过这样一个方法来获取路径长度，后面对这样一个长度做属性动画，就可以达到一个路径的绘制效果。
`mLength = mPathMeasure.getLength();`

创建属性动画
要add一个Listener，用来获取数值发生器从0到100每次给出的数值，每次获取到这个值，就让这个自定VIEW同步刷新，调用onDraw()去绘制出这样一个路径片段。
```
ValueAnimator animator = ValueAnimator.ofFloat(0, 1);
        animator.setDuration(1000);
        animator.setInterpolator(new LinearInterpolator());
        animator.setRepeatCount(ValueAnimator.INFINITE);
        animator.addUpdateListener(new ValueAnimator.AnimatorUpdateListener() {
            @Override
            public void onAnimationUpdate(ValueAnimator valueAnimator) {
                mAnimValue = (float) valueAnimator.getAnimatedValue();
                invalidate();
            }
        });
        animator.start();
```
onDraw()
让绘制圆的时候，终点的坐标从0到整个长度进行变化，而让start坐标始终为零。然后通过调用getSegment()方法，第二个参数从0一直截取到stop这个点，由于stop是从0到100递增的，所以每次截取出来的东西就会越来越长，最终完成一次，完整的路径的绘制。
```
@Override
    protected void onDraw(Canvas canvas) {
        super.onDraw(canvas);
        mDst.reset();
        mDst.lineTo(0, 0);

        float stop = mLength * mAnimValue;
        float start = (float) (stop - ((0.5 - Math.abs(mAnimValue - 0.5)) * mLength));

        mPathMeasure.getSegment(start, stop, mDst, true);
        canvas.drawPath(mDst, mPaint);
    }
```
### getPosTan(); API的使用

![](https://upload-images.jianshu.io/upload_images/11184437-d01b4cf6ef756611.gif?imageMogr2/auto-orient/strip)


这个API着重是去除路径上点的坐标，以及这个点的运动趋势。

用到的主要变量
```
    private Path mPath;
    private float[] mPos;//用来存放取出的具体点的坐标
    private float[] mTan;//用来存放取出当前点的运动趋势
    private Paint mPaint;
    private PathMeasure mPathMeasure;
    private ValueAnimator mAnimator;
    private float mCurrentValue;
```
### onDraw()
首先需要计算出点的坐标
`mPathMeasure.getPosTan(mCurrentValue * mPathMeasure.getLength(), mPos, mTan);`
第一个参数，就是路径具体的长度比例，从0变换到100的路径长度。后面俩个参数，用于输出具体坐标的数组。
先来绘制路径上的点
然后获取切面的角度`float degree = (float) (Math.atan2(mTan[1], mTan[0]) * 180 / Math.PI);`
```
canvas.save();
        canvas.translate(400, 400);
        canvas.drawPath(mPath, mPaint);
        canvas.drawCircle(mPos[0], mPos[1], 10, mPaint);
        canvas.rotate(degree);
        canvas.drawLine(0, -200, 300, -200, mPaint);
        canvas.restore();
```
完整的onDraw()代码
```
@Override
    protected void onDraw(Canvas canvas) {
        super.onDraw(canvas);
        mPathMeasure.getPosTan(mCurrentValue * mPathMeasure.getLength(), mPos, mTan);
        float degree = (float) (Math.atan2(mTan[1], mTan[0]) * 180 / Math.PI);

        canvas.save();
        canvas.translate(400, 400);
        canvas.drawPath(mPath, mPaint);
        canvas.drawCircle(mPos[0], mPos[1], 10, mPaint);
        canvas.rotate(degree);
        canvas.drawLine(0, -200, 300, -200, mPaint);
        canvas.restore();
    }
```