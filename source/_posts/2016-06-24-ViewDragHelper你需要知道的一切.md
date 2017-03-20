title: ViewDragHelper你需要知道的一切
date: 2016-06-24 10:17:06
tags: [Android, View]
---

> ViewDragHelper is a utility class for writing custom ViewGroups. It offers a number of useful operations and state tracking for allowing a user to drag and reposition views within their parent ViewGroup.

<!--more-->

## 参考文章
- [**Android ViewDragHelper完全解析 自定义ViewGroup神器**](http://blog.csdn.net/lmj623565791/article/details/46858663)
- [**ViewDragHelper详解**](http://www.jcodecraeer.com/a/anzhuokaifa/androidkaifa/2014/0911/1680.html)
- [**一个详细的示例**](https://github.com/wangjia55/ViewDragHelper)，汇聚多个丰富多彩的Demo，让你从浅入深学会如何使用ViewDragHelper

## 几点说明
- `ViewDragHelper.Callback`是连接ViewDragHelper与view之间的桥梁（这个view一般是指拥子view的容器即parentView）；
- ViewDragHelper的实例是通过静态工厂方法创建的；
- 能够指定拖动的方向；
- ViewDragHelper可以检测到是否触及到边缘；
- ViewDragHelper并不是直接作用于要被拖动的View，而是使其控制的视图容器中的子View可以被拖动，如果要指定某个子view的行为，需要在Callback中想办法；
- ViewDragHelper的本质其实是分析`onInterceptTouchEvent`和`onTouchEvent`的MotionEvent参数，然后根据分析的结果去改变一个容器中被拖动子View的位置（ 通过offsetTopAndBottom(int offset)和offsetLeftAndRight(int offset)方法 ），他能在触摸的时候判断当前拖动的是哪个子View；

## 使用示例

### 自定义ViewGroup
``` java
public class VDHLayout extends FrameLayout {
    private ViewDragHelper dragHelper;

    public VDHLayout(Context context) {
        this(context, null);
    }

    public VDHLayout(Context context, AttributeSet attrs) {
        this(context, attrs, 0);
    }

    public VDHLayout(Context context, AttributeSet attrs, int defStyleAttr) {
        super(context, attrs, defStyleAttr);
        dragHelper = ViewDragHelper.create(this, 1.0f, new DragHelperCallback());
    }

    class DragHelperCallback extends ViewDragHelper.Callback {

        @Override
        public boolean tryCaptureView(View child, int pointerId) {
            return true;
        }

        @Override
        public int clampViewPositionHorizontal(View child, int left, int dx) {
            return super.clampViewPositionHorizontal(child, left, dx);
        }

        @Override
        public int clampViewPositionVertical(View child, int top, int dy) {
            return super.clampViewPositionVertical(child, top, dy);
        }
    }


    @Override
    public boolean onInterceptTouchEvent(MotionEvent ev) {
        return dragHelper.shouldInterceptTouchEvent(ev);
    }

    @Override
    public boolean onTouchEvent(MotionEvent event) {
        dragHelper.processTouchEvent(event);
        return true;
    }
}
```

自定义ViewGroup分为了3个步骤：
- 创建ViewDragHelper实例:

``` java
dragHelper = ViewDragHelper.create(this, 1.0f, new DragHelperCallback());
```
可以看到创建实例需要3个参数：当前的ViewGroup，sensitivity和ViewDragHelper.Callback。其中sensitivity主要适用于设置touchSlop，sensitivity越大，touchSlop越小：

``` java
helper.mTouchSlop = (int) (helper.mTouchSlop * (1 / sensitivity));
```

- 触摸相关方法

``` java
@Override
public boolean onInterceptTouchEvent(MotionEvent ev) {
    return dragHelper.shouldInterceptTouchEvent(ev);
}

@Override
public boolean onTouchEvent(MotionEvent event) {
    dragHelper.processTouchEvent(event);
    return true;
}
```

`onInterceptTouchEvent`中通过使用`mDragger.shouldInterceptTouchEvent(event)`来决定我们是否应该拦截当前的事件。
`onTouchEvent`中通过`mDragger.processTouchEvent(event)`处理事件。

- 实现ViewDragHelper.Callback相关方法

``` java
class DragHelperCallback extends ViewDragHelper.Callback {
    @Override
    public boolean tryCaptureView(View child, int pointerId) {
        return true;
    }

    @Override
    public int clampViewPositionHorizontal(View child, int left, int dx) {
        return super.clampViewPositionHorizontal(child, left, dx);
    }

    @Override
    public int clampViewPositionVertical(View child, int top, int dy) {
        return super.clampViewPositionVertical(child, top, dy);
    }
}
```

注意callback中复写的3个方法：
`tryCaptureView`:返回ture则表示可以捕获该view，可根据传入的view参数决定哪些可以捕获
`clampViewPositionHorizontal`和`clampViewPositionVertical`:对child移动的边界进行控制,`left` , `top` 分别为即将移动到的位置。

**注**：使View在ViewGroup内移动：
``` java
@Override
public int clampViewPositionHorizontal(View child, int left, int dx) {
   final int leftBound = getPaddingLeft();
   final int rightBound = getWidth() - mDragView.getWidth() - getPaddingRight();
   final int newLeft = Math.min(Math.max(left, leftBound), rightBound);
   return newLeft;
}
```

### 功能展示
ViewDragHelper能做的可不仅仅是拖动view，除此之外，他还能做以下的一些操作：
- 边界检测、加速度检测
边界检测时,首先记得添加`dragHelper.setEdgeTrackingEnabled(ViewDragHelper.EDGE_LEFT);`,重写`onEdgeDragStarted`方法，在方法中可以绕过`tryCaptureView`,主动通过`captureChildView`对其进行捕获,
- 回调Drag Release
- 移动到某个指定的位置

``` java
dragHelper.settleCapturedViewAt(mAutoBackOriginPos.x, mAutoBackOriginPos.y);
invalidate();

@Override
public void computeScroll() {
   if (dragHelper.continueSettling(true)) {
       invalidate();
   }
}
```
移动到指定位置注意调用`settleCapturedViewAt`方法，紧接着`invalidate();`,配合`computeScroll()`使用。

修改之后的VHDLayout:
``` java
public class VDHLayout extends LinearLayout {
    private ViewDragHelper dragHelper;

    private View mDragView;
    private View mAutoBackView;
    private View mEdgeTrackerView;

    private Point mAutoBackOriginPos = new Point();

    public VDHLayout(Context context) {
        this(context, null);
    }

    public VDHLayout(Context context, AttributeSet attrs) {
        this(context, attrs, 0);
    }

    public VDHLayout(Context context, AttributeSet attrs, int defStyleAttr) {
        super(context, attrs, defStyleAttr);
        dragHelper = ViewDragHelper.create(this, 1.0f, new DragHelperCallback());
        dragHelper.setEdgeTrackingEnabled(ViewDragHelper.EDGE_LEFT);
    }

    class DragHelperCallback extends ViewDragHelper.Callback {

        @Override
        public boolean tryCaptureView(View child, int pointerId) {
            //mEdgeTrackerView禁止直接移动
            return child == mDragView || child == mAutoBackView;
        }

        @Override
        public int clampViewPositionHorizontal(View child, int left, int dx) {
            final int leftBound = getPaddingLeft();
            final int rightBound = getWidth() - mDragView.getWidth() - getPaddingRight();
            final int newLeft = Math.min(Math.max(left, leftBound), rightBound);
            return newLeft;
        }

        @Override
        public int clampViewPositionVertical(View child, int top, int dy) {
            return top;
        }

        @Override
        public void onViewReleased(View releasedChild, float xvel, float yvel) {
            //mAutoBackView手指释放时可以自动回去
            if (releasedChild == mAutoBackView) {
                dragHelper.settleCapturedViewAt(mAutoBackOriginPos.x, mAutoBackOriginPos.y);
                invalidate();
            }
        }

        @Override
        public void onEdgeDragStarted(int edgeFlags, int pointerId) {
            dragHelper.captureChildView(mEdgeTrackerView, pointerId);
        }
    }

    @Override
    public void computeScroll() {
        if (dragHelper.continueSettling(true)) {
            invalidate();
        }
    }

    @Override
    public boolean onInterceptTouchEvent(MotionEvent ev) {
        return dragHelper.shouldInterceptTouchEvent(ev);
    }

    @Override
    public boolean onTouchEvent(MotionEvent event) {
        dragHelper.processTouchEvent(event);
        return true;
    }

    @Override
    protected void onLayout(boolean changed, int l, int t, int r, int b) {
        super.onLayout(changed, l, t, r, b);

        mAutoBackOriginPos.x = mAutoBackView.getLeft();
        mAutoBackOriginPos.y = mAutoBackView.getTop();
    }

    @Override
    protected void onFinishInflate() {
        super.onFinishInflate();

        mDragView = getChildAt(0);
        mAutoBackView = getChildAt(1);
        mEdgeTrackerView = getChildAt(2);
    }
```

### 效果演示
### 冲突事件处理
- 当子View不消耗点击事件的时候，整个手势操作会直接进入onTouchEvent，然后会调用`dragHelper.processTouchEvent(event);`来进行拖动操作的处理
- 当子View消耗点击事件的时候，则会先走`onInterceptTouchEvent`方法，判断能否捕获，这个过程取决于另外两个回调方法：`getViewHorizontalDragRange`,`getViewVerticalDragRange`,因此，如果Button和设置了clickable=true的View的时候，记得重写两个方法，如下：

``` java
@Override
public int getViewHorizontalDragRange(View child) {
     return getMeasuredWidth() - child.getMeasuredWidth();
}

@Override
public int getViewVerticalDragRange(View child) {
     return getMeasuredHeight() - child.getMeasuredHeight();
}
```
## 源码分析
### ViewDragHelper实例的创建
ViewDragHelper重载了两个create()静态方法：

``` java
/**
  * Factory method to create a new ViewDragHelper.
  *
  * @param forParent Parent view to monitor
  *                  自定义的ViewGroup
  * @param cb Callback to provide information and receive events
  *           控制子View拖拽需要的回调对象
  * @return a new ViewDragHelper instance
  */
public static ViewDragHelper create(ViewGroup forParent, Callback cb) {
    return new ViewDragHelper(forParent.getContext(), forParent, cb);
}

/**
  * Factory method to create a new ViewDragHelper.
  *
  * @param forParent Parent view to monitor
  * @param sensitivity Multiplier for how sensitive the helper should be about detecting
  *                    the start of a drag. Larger values are more sensitive. 1.0f is normal.
  * @param cb Callback to provide information and receive events
  * @return a new ViewDragHelper instance
  */
public static ViewDragHelper create(ViewGroup forParent, float sensitivity, Callback cb) {
    final ViewDragHelper helper = create(forParent, cb);
    helper.mTouchSlop = (int) (helper.mTouchSlop * (1 / sensitivity));
    return helper;
}
```

create()直接调用了ViewDragHelper唯一的私有构造方法：

``` java

/**
  * Apps should use ViewDragHelper.create() to get a new instance.
  * This will allow VDH to use internal compatibility implementations for different
  * platform versions.
  *
  * @param context Context to initialize config-dependent params from
  * @param forParent Parent view to monitor
  */
private ViewDragHelper(Context context, ViewGroup forParent, Callback cb) {
    if (forParent == null) {
        throw new IllegalArgumentException("Parent view may not be null");
    }
    if (cb == null) {
        throw new IllegalArgumentException("Callback may not be null");
    }

    mParentView = forParent;
    mCallback = cb;

    // ViewConfiguration类里定义了View相关的一系列时间、大小、距离等常量
    final ViewConfiguration vc = ViewConfiguration.get(context);
    final float density = context.getResources().getDisplayMetrics().density;
    // 边缘触摸的范围
    mEdgeSize = (int) (EDGE_SIZE * density + 0.5f);
    // 系统所能识别出的可以被认为是滑动的最小距离
    mTouchSlop = vc.getScaledTouchSlop();
    // fling时的最大速率，单位像素每秒
    mMaxVelocity = vc.getScaledMaximumFlingVelocity();
    // fling时的最小速率
    mMinVelocity = vc.getScaledMinimumFlingVelocity();
    // mScroller : View滚动的辅助类
    mScroller = ScrollerCompat.create(context, sInterpolator);
}

```

### 对Touch事件的处理
当`mParentView`（自定义ViewGroup）被触摸时，首先会调用mParentView的`onInterceptTouchEvent(MotionEvent ev)`， 接着就调用`shouldInterceptTouchEvent(MotionEvent ev)`:

``` java
/**
  * Check if this event as provided to the parent view's onInterceptTouchEvent should
  * cause the parent to intercept the touch event stream.
  *
  * @param ev MotionEvent provided to onInterceptTouchEvent
  * @return true if the parent view should return true from onInterceptTouchEvent
  */
public boolean shouldInterceptTouchEvent(MotionEvent ev) {
    final int action = MotionEventCompat.getActionMasked(ev);
    final int actionIndex = MotionEventCompat.getActionIndex(ev);

    if (action == MotionEvent.ACTION_DOWN) {
        // Reset things for a new event stream, just in case we didn't get
        // the whole previous stream.
        cancel();
    }

    if (mVelocityTracker == null) {
        // mVelocityTracker记录下的是本次ACTION_DOWN事件直至ACTION_UP事件发生后 （下次ACTION_DOWN事件发生前）的所有触摸点的信息
        mVelocityTracker = VelocityTracker.obtain();
    }
    mVelocityTracker.addMovement(ev);

    switch (action) {
        case MotionEvent.ACTION_DOWN: {
            final float x = ev.getX();
            final float y = ev.getY();
            final int pointerId = MotionEventCompat.getPointerId(ev, 0);
            // 保存手势的初始信息
            saveInitialMotion(x, y, pointerId);
            // 调用findTopChildUnder((int) x, (int) y); 来获取当前触摸点下最顶层的子View
            final View toCapture = findTopChildUnder((int) x, (int) y);

            // Catch a settling view if possible.
            // mDragState共有三种取值:
            // STATE_IDLE：所有的View处于静止空闲状态
            // STATE_DRAGGING：某个View正在被用户拖动（用户正在与设备交互）
            // STATE_SETTLING：某个View正在安置状态中（用户并没有交互操作），就是自动滚动的过程中
            if (toCapture == mCapturedView && mDragState == STATE_SETTLING) {
                tryCaptureViewForDrag(toCapture, pointerId);
            }

            // 向外部通知mParentView的某些边缘被触摸到了，mInitialEdgesTouched是在刚才调
            // 用过的saveInitialMotion方法里进行赋值的
            final int edgesTouched = mInitialEdgesTouched[pointerId];
            if ((edgesTouched & mTrackingEdges) != 0) {
                mCallback.onEdgeTouched(edgesTouched & mTrackingEdges, pointerId);
            }
            break;
        }

        case MotionEventCompat.ACTION_POINTER_DOWN: {
            final int pointerId = MotionEventCompat.getPointerId(ev, actionIndex);
            final float x = MotionEventCompat.getX(ev, actionIndex);
            final float y = MotionEventCompat.getY(ev, actionIndex);
            
            saveInitialMotion(x, y, pointerId);

            // A ViewDragHelper can only manipulate one view at a time.
            if (mDragState == STATE_IDLE) {
                final int edgesTouched = mInitialEdgesTouched[pointerId];
                if ((edgesTouched & mTrackingEdges) != 0) {
                    mCallback.onEdgeTouched(edgesTouched & mTrackingEdges, pointerId);
                }
            } else if (mDragState == STATE_SETTLING) {
                // Catch a settling view if possible.
                final View toCapture = findTopChildUnder((int) x, (int) y);
                if (toCapture == mCapturedView) {
                    tryCaptureViewForDrag(toCapture, pointerId);
                }
            }
            break;
        }

        case MotionEvent.ACTION_MOVE: {
            // First to cross a touch slop over a draggable view wins. Also report edge drags.
            final int pointerCount = MotionEventCompat.getPointerCount(ev);
            for (int i = 0; i < pointerCount; i++) {
                final int pointerId = MotionEventCompat.getPointerId(ev, i);
                final float x = MotionEventCompat.getX(ev, i);
                final float y = MotionEventCompat.getY(ev, i);
                final float dx = x - mInitialMotionX[pointerId];
                final float dy = y - mInitialMotionY[pointerId];

                final View toCapture = findTopChildUnder((int) x, (int) y);
                final boolean pastSlop = toCapture != null && checkTouchSlop(toCapture, dx, dy);
                if (pastSlop) {
                    // check the callback's
                    // getView[Horizontal|Vertical]DragRange methods to know
                    // if you can move at all along an axis, then see if it
                    // would clamp to the same value. If you can't move at
                    // all in every dimension with a nonzero range, bail.
                    final int oldLeft = toCapture.getLeft();
                    final int targetLeft = oldLeft + (int) dx;
                    final int newLeft = mCallback.clampViewPositionHorizontal(toCapture,
                            targetLeft, (int) dx);
                    final int oldTop = toCapture.getTop();
                    final int targetTop = oldTop + (int) dy;
                    final int newTop = mCallback.clampViewPositionVertical(toCapture, targetTop,
                            (int) dy);
                    final int horizontalDragRange = mCallback.getViewHorizontalDragRange(
                                toCapture);
                    final int verticalDragRange = mCallback.getViewVerticalDragRange(toCapture);
                    if ((horizontalDragRange == 0 || horizontalDragRange > 0
                            && newLeft == oldLeft) && (verticalDragRange == 0
                                || verticalDragRange > 0 && newTop == oldTop)) {
                       break;
                    }
                }
                reportNewEdgeDrags(dx, dy, pointerId);
                if (mDragState == STATE_DRAGGING) {
                    // Callback might have started an edge drag
                    break;
                }

                if (pastSlop && tryCaptureViewForDrag(toCapture, pointerId)) {
                    break;
                }
            }
            saveLastMotion(ev);
            break;
        }

        case MotionEventCompat.ACTION_POINTER_UP: {
            final int pointerId = MotionEventCompat.getPointerId(ev, actionIndex);
            clearMotionHistory(pointerId);
            break;
        }

        case MotionEvent.ACTION_UP:
        case MotionEvent.ACTION_CANCEL: {
            cancel();
            break;
        }
    }

    return mDragState == STATE_DRAGGING;
}
```

`findTopChildUnder`:在父View的坐标系统下，根据给定的坐标找到最顶层的子View。子View的顺序是由`getOrderedChildIndex(int)`来决定的。

``` java
/**
  * Find the topmost child under the given point within the parent view's coordinate system.
  * The child order is determined using {@link Callback#getOrderedChildIndex(int)}.
  *
  * @param x X position to test in the parent's coordinate system
  * @param y Y position to test in the parent's coordinate system
  * @return The topmost child view under (x, y) or null if none found.
  */
public View findTopChildUnder(int x, int y) {
    final int childCount = mParentView.getChildCount();
    for (int i = childCount - 1; i >= 0; i--) {
        final View child = mParentView.getChildAt(mCallback.getOrderedChildIndex(i));
        if (x >= child.getLeft() && x < child.getRight() &&
                   y >= child.getTop() && y < child.getBottom()) {
            return child;
        }
    }
    return null;
}
```
如上，当同一位置有2个子View重叠，想要让下层的子View被选中， 那么就要实现Callback里的`getOrderedChildIndex(int index)`方法来改变查找子View的顺序:

``` java
public int getOrderedChildIndex(int index) {
	int indexTop = mParentView.indexOfChild(topView);
	int indexBottom = mParentView.indexOfChild(bottomView);
	if (index == indexTop) {
		return indexBottom;
	}
	return index;
}
```

- `ACTION_DOWN事件的处理`:
`ACTION_DOWN`部分处理完了，跳过switch语句块，剩下的代码就只有`return mDragState == STATE_DRAGGING;`。在`ACTION_DOWN`部分没有对`mDragState`进行赋值，其默认值为`STATE_IDLE`，所以此处返回false,接下来会调用每个子View 的`dispatchTouchEvent()`方法，`dispatchTouchEvent`里一般又会调用`onTouchEvent()`。

**情形一**：如果没有子View消费这次事件（子View的dispatchTouchEvent()返回都是false），会调用mParentView的super.dispatchTouchEvent (ev)，即View中的dispatchTouchEvent(ev)，然后调用mParentView的onTouchEvent()方法， 再调用ViewDragHelper的processTouchEvent(MotionEvent ev)方法。此时（ACTION_DOWN事件发生时）mParentView的onTouchEvent()要返回true， onTouchEvent()才能继续接受到接下来的ACTION_MOVE、ACTION_UP等事件，否则无法完成拖动（除了ACTION_DOWN外的其他事件发生时返回true或false都 不会影响接下来的事件接受），因为拖动的相关代码是写在processTouchEvent()里的ACTION_MOVE部分的。 

``` java
    /**
     * Process a touch event received by the parent view. This method will dispatch callback events
     * as needed before returning. The parent view's onTouchEvent implementation should call this.
     *
     * @param ev The touch event received by the parent view
     */
    public void processTouchEvent(MotionEvent ev) {
        final int action = MotionEventCompat.getActionMasked(ev);
        final int actionIndex = MotionEventCompat.getActionIndex(ev);

        if (action == MotionEvent.ACTION_DOWN) {
            // Reset things for a new event stream, just in case we didn't get
            // the whole previous stream.
            cancel();
        }

        if (mVelocityTracker == null) {
            mVelocityTracker = VelocityTracker.obtain();
        }
        mVelocityTracker.addMovement(ev);

        switch (action) {
            case MotionEvent.ACTION_DOWN: {
                final float x = ev.getX();
                final float y = ev.getY();
                final int pointerId = MotionEventCompat.getPointerId(ev, 0);
                final View toCapture = findTopChildUnder((int) x, (int) y);

                saveInitialMotion(x, y, pointerId);

                // Since the parent is already directly processing this touch event,
                // there is no reason to delay for a slop before dragging.
                // Start immediately if possible.
                tryCaptureViewForDrag(toCapture, pointerId);

                final int edgesTouched = mInitialEdgesTouched[pointerId];
                if ((edgesTouched & mTrackingEdges) != 0) {
                    mCallback.onEdgeTouched(edgesTouched & mTrackingEdges, pointerId);
                }
                break;
            }

            // ...
        }
    }
```

在`processTouchEvent`中的处理和`shouldInterceptTouchEvent()`里`ACTION_DOWN`那部分基本一致，唯一区别就是这里没有约束条件直接调用了`tryCaptureViewForDrag()`方法：

``` java
    /**
     * Attempt to capture the view with the given pointer ID. The callback will be involved.
     * This will put us into the "dragging" state. If we've already captured this view with
     * this pointer this method will immediately return true without consulting the callback.
     *
     * @param toCapture View to capture
     * @param pointerId Pointer to capture with
     * @return true if capture was successful
     */
    boolean tryCaptureViewForDrag(View toCapture, int pointerId) {
        if (toCapture == mCapturedView && mActivePointerId == pointerId) {
            // Already done!
            return true;
        }
        // 在tryCaptureView()中决定是否需要拖动当前触摸到的View
        if (toCapture != null && mCallback.tryCaptureView(toCapture, pointerId)) {
            mActivePointerId = pointerId;
            captureChildView(toCapture, pointerId);
            return true;
        }
        return false;
    }
```

在`tryCaptureViewForDrag()`方法中，调用了`mCallback.tryCaptureView(toCapture, pointerId)`来决定是否需要拖动当前触摸到的View，若`tryCaptureView`返回true，，紧接着就调用了`captureChildView(toCapture, pointerId);`方法：

``` java
/**
     * Capture a specific child view for dragging within the parent. The callback will be notified
     * but {@link Callback#tryCaptureView(android.view.View, int)} will not be asked permission to
     * capture this view.
     *
     * @param childView Child view to capture
     * @param activePointerId ID of the pointer that is dragging the captured child view
     */
    public void captureChildView(View childView, int activePointerId) {
        if (childView.getParent() != mParentView) {
            throw new IllegalArgumentException("captureChildView: parameter must be a descendant " +
                    "of the ViewDragHelper's tracked parent view (" + mParentView + ")");
        }

        mCapturedView = childView;
        mActivePointerId = activePointerId;
        mCallback.onViewCaptured(childView, activePointerId);
        setDragState(STATE_DRAGGING);
    }
```

如上，在`captureChildView(toCapture, pointerId)`中记录下将要拖动的View和触摸的手指编号，并调用Callback的`onViewCaptured(childView, activePointerId)`通知外部有子View被捕获到了， 再调用`setDragState()`设置当前的状态为`STATE_DRAGGING`:

``` java
void setDragState(int state) {
        if (mDragState != state) {
            mDragState = state;
            // 状态改变后会调用Callback的onViewDragStateChanged()通知状态的变化
            mCallback.onViewDragStateChanged(state);
            if (mDragState == STATE_IDLE) {
                mCapturedView = null;
            }
        }
    }
```

假设ACTION_DOWN发生后在mParentView的onTouchEvent()返回了true，接下来就会执行ACTION_MOVE部分：

``` java
public void processTouchEvent(MotionEvent ev) {
        //...

            case MotionEvent.ACTION_MOVE: {
                if (mDragState == STATE_DRAGGING) {
                    final int index = MotionEventCompat.findPointerIndex(ev, mActivePointerId);
                    final float x = MotionEventCompat.getX(ev, index);
                    final float y = MotionEventCompat.getY(ev, index);
                    final int idx = (int) (x - mLastMotionX[mActivePointerId]);
                    final int idy = (int) (y - mLastMotionY[mActivePointerId]);

                    dragTo(mCapturedView.getLeft() + idx, mCapturedView.getTop() + idy, idx, idy);

                    saveLastMotion(ev);
                } else {
                    // Check to see if any pointer is now over a draggable view.
                    final int pointerCount = MotionEventCompat.getPointerCount(ev);
                    for (int i = 0; i < pointerCount; i++) {
                        final int pointerId = MotionEventCompat.getPointerId(ev, i);
                        final float x = MotionEventCompat.getX(ev, i);
                        final float y = MotionEventCompat.getY(ev, i);
                        final float dx = x - mInitialMotionX[pointerId];
                        final float dy = y - mInitialMotionY[pointerId];

                        reportNewEdgeDrags(dx, dy, pointerId);
                        if (mDragState == STATE_DRAGGING) {
                            // Callback might have started an edge drag.
                            break;
                        }

                        final View toCapture = findTopChildUnder((int) x, (int) y);
                        if (checkTouchSlop(toCapture, dx, dy) &&
                                tryCaptureViewForDrag(toCapture, pointerId)) {
                            break;
                        }
                    }
                    saveLastMotion(ev);
                }
                break;
            }

            //...
        }
    }
```

如上，这部分代码首先会判断`mDragState`的状态，如果之前在`ACTION_DOWN`中捕获到了要拖动的View，那么就执行if里面的代码，如果没有捕获到，就执行else里面的代码，这里还有两个方法涉及到了Callback里的方法，需要来解析一下， 分别是`reportNewEdgeDrags()`和`checkTouchSlop()`

``` java
    private void reportNewEdgeDrags(float dx, float dy, int pointerId) {
        int dragsStarted = 0;
        if (checkNewEdgeDrag(dx, dy, pointerId, EDGE_LEFT)) {
            dragsStarted |= EDGE_LEFT;
        }
        if (checkNewEdgeDrag(dy, dx, pointerId, EDGE_TOP)) {
            dragsStarted |= EDGE_TOP;
        }
        if (checkNewEdgeDrag(dx, dy, pointerId, EDGE_RIGHT)) {
            dragsStarted |= EDGE_RIGHT;
        }
        if (checkNewEdgeDrag(dy, dx, pointerId, EDGE_BOTTOM)) {
            dragsStarted |= EDGE_BOTTOM;
        }

        if (dragsStarted != 0) {
            mEdgeDragsInProgress[pointerId] |= dragsStarted;
            mCallback.onEdgeDragStarted(dragsStarted, pointerId);
        }
    }
```

这里对四个边缘都做了一次检查，检查是否在某些边缘产生拖动了，如果有拖动，就将有拖动的边缘记录在mEdgeDragsInProgress中，再调用Callback的onEdgeDragStarted(int edgeFlags, int pointerId)通知某个边缘开始产生拖动了。虽然reportNewEdgeDrags()会被调用很多次（因为processTouchEvent()的ACTION_MOVE部分会执行很多次）， 但mCallback.onEdgeDragStarted(dragsStarted, pointerId)只会调用一次，具体的要看checkNewEdgeDrag()这个方法：

``` java
    private boolean checkNewEdgeDrag(float delta, float odelta, int pointerId, int edge) {
        final float absDelta = Math.abs(delta);
        final float absODelta = Math.abs(odelta);

        if ((mInitialEdgesTouched[pointerId] & edge) != edge  || (mTrackingEdges & edge) == 0 ||
                (mEdgeDragsLocked[pointerId] & edge) == edge ||
                (mEdgeDragsInProgress[pointerId] & edge) == edge ||
                (absDelta <= mTouchSlop && absODelta <= mTouchSlop)) {
            return false;
        }
        if (absDelta < absODelta * 0.5f && mCallback.onEdgeLock(edge)) {
            mEdgeDragsLocked[pointerId] |= edge;
            return false;
        }
        return (mEdgeDragsInProgress[pointerId] & edge) == 0 && absDelta > mTouchSlop;
    }
```