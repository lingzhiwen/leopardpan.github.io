## 一、需求来源：
设计师要求还原设计的阴影，下面是sketch原型参数：
```
圆角：8px

外阴影：                    
Offset:     0px     1px                         
             X       Y              

Effect:     10px    0px                 
            Blur    Spread                  

颜色：      #135B90D0                       
```
## 二、Android本身控件自带阴影效果无法还原
Google的Material Design中对于阴影的定义是：海拔高度是相对深度或距离，是两个表面在 Z 轴上的距离。

实现方式：

- 第一种方式：elevation属性         
View的大小位置都是通过x，y确定的，而现在有了z轴的概念，而这个z值就是View的高度（elevation），而高度决定了阴影（shadow）的大小。

- 第二种方式：CardView自带属性              
```
card_view:cardElevation 阴影的大小
card_view:cardMaxElevation 阴影最大高度
card_view:cardBackgroundColor 卡片的背景色
card_view:cardCornerRadius 卡片的圆角大小
card_view:contentPadding 卡片内容于边距的间隔
```
- 第三种方式：
设置边框为shape，通过layer-list实现层次感
```
<layer-list xmlns:android="http://schemas.android.com/apk/res/android">
<item>
    <shape android:shape="oval">
        <padding
            android:bottom="2dp"
            android:left="2dp"
            android:right="2dp"
            android:top="2dp" />
        <solid android:color="#0DF3F3F3" />
        <corners android:radius="8dp" />
    </shape>
</item>
    <item>
        <shape
            android:layout_width="wrap_content"
            android:shape="oval">
            <padding
                android:bottom="2dp"
                android:left="2dp"
                android:right="2dp"
                android:top="2dp" />
            <solid android:color="#10F3F3F3" />
            <corners android:radius="8dp" />
        </shape>
    </item>
<item>
    <shape android:shape="oval">
        <padding
            android:bottom="2dp"
            android:left="2dp"
            android:right="2dp"
            android:top="2dp" />
        <solid android:color="#15F3F3F3" />
        <corners android:radius="8dp" />
    </shape>
</item>
<item>
    <shape android:shape="oval">
        <padding
            android:bottom="2dp"
            android:left="2dp"
            android:right="2dp"
            android:top="2dp" />
        <solid android:color="#20F3F3F3" />
        <corners android:radius="8dp" />
    </shape>
</item>
<item>
    <shape android:shape="oval">
        <padding
            android:bottom="2dp"
            android:left="2dp"
            android:right="2dp"
            android:top="2dp" />
        <solid android:color="#25F3F3F3" />
        <corners android:radius="8dp" />
    </shape>
</item>
<item>
    <shape android:shape="oval">
        <solid android:color="#FFFFFF" />
    </shape>
</item>
</layer-list>
```

## 三、使用Android画笔Paint来实现
- 第四种方式（属性一一对应设计稿）：
```
/**
 * 使用圆角时，应设置圆角相同的background
 */

public class ShadowRelativeLayout extends RelativeLayout {
    /**
     * 阴影的颜色
     */
    private int shadowColor = Color.argb(90, 0, 0, 0);
    /**
     * 高斯模糊的模糊半径,值越大越模糊，越小越清晰,
     */
    private float shadowBlur = 30;
    /**
     * 阴影的圆角
     */
    private float shadowRadius = 0;

    /**
     * 阴影的偏移
     */
    private float shadowDx = 0;
    private float shadowDy = 0;

    public ShadowRelativeLayout(@NonNull Context context) {
        this(context, null);
    }

    public ShadowRelativeLayout(@NonNull Context context,
                                @Nullable AttributeSet attrs) {
        this(context, attrs, 0);
    }

    public ShadowRelativeLayout(@NonNull Context context, @Nullable AttributeSet attrs,
                                int defStyleAttr) {
        super(context, attrs, defStyleAttr);
        dealAttrs(context, attrs);
        setPaint();
    }

    private Paint mPaint = new Paint(Paint.ANTI_ALIAS_FLAG);

    @Override
    public void draw(Canvas canvas) {
        setInsetBackground();
        canvas.drawRoundRect(getRectF(), shadowRadius, shadowRadius, mPaint);
        super.draw(canvas);
    }

    private boolean setInsetBackground() {
        Drawable background = getBackground();
        if (background == null || background instanceof InsetDrawable) {
            return false;
        }
        InsetDrawable drawable =
                new InsetDrawable(background, getPaddingLeft(), getPaddingTop(),
                        getPaddingRight(), getPaddingBottom());
        setBackground(drawable);
        return true;
    }

    private RectF getRectF() {
        return new RectF(getPaddingLeft() + shadowDx, getPaddingTop() + shadowDy,
                getWidth() - getPaddingRight() + shadowDx,
                getHeight() - getPaddingBottom() + shadowDy);
    }

    private void dealAttrs(Context context, AttributeSet attrs) {
        TypedArray typedArray = context.obtainStyledAttributes(attrs, R.styleable.ShadowRelativeLayout);
        if (typedArray != null) {
            shadowColor = typedArray.getColor(R.styleable.ShadowRelativeLayout_shadow_color, shadowColor);
            shadowRadius =
                    typedArray.getDimension(R.styleable.ShadowRelativeLayout_shadow_radius, shadowRadius);
            shadowBlur =
                    typedArray.getDimension(R.styleable.ShadowRelativeLayout_shadow_blur, shadowBlur);
            shadowDx = typedArray.getDimension(R.styleable.ShadowRelativeLayout_shadow_dx, shadowDx);
            shadowDy = typedArray.getDimension(R.styleable.ShadowRelativeLayout_shadow_dy, shadowDy);
            typedArray.recycle();
        }
    }

    private void setPaint() {
        setLayerType(View.LAYER_TYPE_SOFTWARE, null);  // 关闭硬件加速,阴影才会绘制
        // todo 从AttributeSet获取设置的值
        mPaint.setAntiAlias(true);
        mPaint.setColor(shadowColor);
        mPaint.setMaskFilter(new BlurMaskFilter(shadowBlur, BlurMaskFilter.Blur.NORMAL));
    }

    @Override
    public boolean isOpaque() { //纯色或图片做背景时,draw之前会有黑底色，
        return false;
    }
}
```

attrs.xml
```
<?xml version="1.0" encoding="utf-8"?>
<resources>
    <declare-styleable name="ShadowRelativeLayout">
        <attr format="color" name="shadow_color"/>
        <attr format="dimension" name="shadow_radius"/>
        <attr format="dimension" name="shadow_blur"/>
        <attr format="dimension" name="shadow_dx"/>
        <attr format="dimension" name="shadow_dy"/>
    </declare-styleable>
</resources>
```

使用范例（属性完美映射设计稿）：
```
<ShadowRelativeLayout
    android:layout_width="180dp"
    android:layout_height="100dp"
    android:background="@drawable/shape_round_white_dp8"
    android:padding="10dp"
    bind:shadow_blur="10dp"
    bind:shadow_color="#135B90D0"
    bind:shadow_dx="0dp"
    bind:shadow_dy="1dp"
    bind:shadow_radius="8dp">
    <ImageView
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_centerInParent="true"
        android:layout_gravity="center"
        android:scaleType="center"
        android:src="@drawable/light" />
</ShadowRelativeLayout>

```