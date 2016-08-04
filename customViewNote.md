# 课时1：自定义 View #
1. 自定义 View 的初步认识
2. measure 源码分析
3. layout 的实现
4. draw 画各种图形
5. 自定义 View 示例
6. Touch 事件的感性认识
7. onTouch onTouchEvent onClick 的关系
8. disPatchTouchEvent onIntercepteTouchEvent onTouchEvent 源码
9. 弄清楚几个问题
10. 滑动冲突的完整示例

# 课时1-2：常用工具介绍 #
1. Configuration - 
2. ViewConfiguration
3. GestureDetector - 手势监测
4. VelocityTracker - 速度追踪
5. Scroller 
6. ViewDragHelper - View 拖拽

## 1. Configuration ##
### 1. 用来描述设备的配置信息 ###
1. 比如用户的配置信息：locale 和 scaling
2. 比如设备的相关信息：
	1. 输入模式
	2. 屏幕大小
	3. 屏幕方向

### 2. 示例代码 ###
1. 获得一个 Configuration

		/**获取一个 configuration*/
        Configuration configuration = getResources().getConfiguration();
        
        /**获取国家码*/
        int countryCode = configuration.mcc;
        
        /**获取网络码*/
        int netWorkCode = configuration.mnc;
        
        /**判断横竖屏*/
        if (configuration.orientation == configuration.ORIENTATION_PORTRAIT) {

        } else {
            
        } 

## 2. ViewConfiguration ##
### 1. 提供一些自定义控件用到的标准常量： ###
1. 比如：
	1. UI 超时
	2. 尺寸大小
	3. 滑动距离
	4. 敏感度

### 2. ViewConfiguration 的示例获取 ###
1. ViewConfiguration 常用对象方法

		 /**ViewConfiguration 常用对象方法*/
        ViewConfiguration viewConfiguration = ViewConfiguration.get(context);
        
        /**获取 touchSlop
         * - 系统所能识别滑动的最小距离
         * - 手指滑动的距离 > touchSlop 系统认为是滑动
         */
        int touchSlop = viewConfiguration.getScaledTouchSlop();
        
        /**获取 Fling 速度的最小值和最大值*/
        int minimumVelocity = viewConfiguration.getScaledMinimumFlingVelocity();
        int maximumVelocity = viewConfiguration.getScaledMaximumFlingVelocity();
        
        /**判断是否有物理按键
         * - 占用屏幕空间 
         */
        boolean isHavePermanentMenuKey = viewConfiguration.hasPermanentMenuKey();

2. ViewConfiguration 常用静态方法*

		/**ViewConfiguration 常用静态方法*/
        ViewConfiguration viewConfiguration = ViewConfiguration.get(context);
        
        /**双击间隔时间
         * - 在改时间内是双击
         * - 否则是单击
         */
        int doubleTapTimeout = ViewConfiguration.getDoubleTapTimeout();
        
        /**按住状态转变为长按状态需要的时间*/
        int longPressTimeout = ViewConfiguration.getLongPressTimeout();
        
        /**重复按键的时间*/
        int keyRepeatTimeout = ViewConfiguration.getKeyRepeatTimeout();
          

## 3. GestureDetector ##
### 1. 主要作用简化 Touch 操作 ###

### 2. 示例代码 ###
1. 第一步：实现OnGestureListener
2. 第二步：生成GestureDetector对象
3. 第三步：将Touch事件交给GestureDetector处理 

		public class GestureListenenrImpl implements GestureDetector.OnGestureListener {
    
	    /**触摸屏幕时均会调用该方法*/
	    @Override
	    public boolean onDown(MotionEvent motionEvent) {
	        Log.e("TAG", "onDown --- 触摸屏幕");
	        return false;
	    }
	
	    /**手指在屏幕上按下，且未移动和松开时调用该方法*/
	    @Override
	    public void onShowPress(MotionEvent motionEvent) {
	        Log.e("TAG", "onShowPress --- 按下不移动，松开");
	    }
	
	    /**轻击屏幕时调用该方法*/
	    @Override
	    public boolean onSingleTapUp(MotionEvent motionEvent) {
	        Log.e("TAG", "onSingleTapUp --- 轻击屏幕");
	        return false;
	    }
	
	    /**手指在屏幕上滑动时会调用该方法*/
	    @Override
	    public boolean onScroll(MotionEvent motionEvent, MotionEvent motionEvent1, float v, float v1) {
	        Log.e("TAG", "onScroll --- 手指滑动");
	        return false;
	    }
	
	    /**手指长按屏幕时均会调用该方法*/
	    @Override
	    public void onLongPress(MotionEvent motionEvent) {
	        Log.e("TAG", "onLongPress --- 长按屏幕");
	    }
	
	    /**手指在屏幕上拖动时均会调用该方法*/
	    @Override
	    public boolean onFling(MotionEvent motionEvent, MotionEvent motionEvent1, float v, float v1) {
	        Log.e("TAG", "onFling --- 拖动");
	        return false;
	    }

## 4. VelocityTracker ##
### 1. 用于跟踪触摸屏事件的速率 ###

### 2. 示例代码 ###
1. 开始追踪

		private void startVelocityTracker(MotionEvent event) {  
	    if (mVelocityTracker == null) {  
	         mVelocityTracker = VelocityTracker.obtain();  
	     }  
	     mVelocityTracker.addMovement(event);  
		}  
	- 在这里我们初始化VelocityTracker，并且把要追踪的MotionEvent注册到VelocityTracker的监听中。
	
2. 获取速度

		private int getScrollVelocity() {  
	     // 设置VelocityTracker单位.1000表示1秒时间内运动的像素  
	     mVelocityTracker.computeCurrentVelocity(1000);  
	     // 获取在1秒内X方向所滑动像素值  
	     int xVelocity = (int) mVelocityTracker.getXVelocity();  
	     return Math.abs(xVelocity);  
	    } 

3. 停止追踪，释放
	
		private void stopVelocityTracker() {  
		     if (mVelocityTracker != null) {  
		         mVelocityTracker.recycle();  
		         mVelocityTracker = null;  
		      }  
		}  

## 5. Scroller  ##
### 1. 实现View平滑滚动的一个Helper类 ###
1. 通常在自定义的View时使用
2. 在View中定义一个私有成员mScroller = new Scroller(context)。
3. 设置mScroller滚动的位置时，并不会导致View的滚动，
4. 通常是用mScroller记录/计算View滚动的位置，
5. 再重写View的computeScroll()，完成实际的滚动。 

### 2. 示例代码 ###

### 3. scrollTO() 和 scrollBy() 的关系 ###
1. scrollBy() 的源码：

		public void scrollBy(int x, int y) {   
      		scrollTo(mScrollX + x, mScrollY + y);   
		} 
		
	- 在 scrollBy() 中，最终调用 scrollTo() 方法
	- 最终起作用的就是 scrollTo() 方法
	- scrollBy() 对 scrollTo() 进行了封装
	
### 2. scroll 的本质 ###
	1. 滑动的只是滑动 View 的内容
	2. View 的背景是不移动的
	
### 3. scrollTo() 和 scrollBy() 方法的坐标说明 ###
1. 通过 Demo 验证
2. 查看源码


## 6. ViewDragHelper ##
### 1. 主要作用简化 View 的拖拽相关操作 ###
1. 在自定义ViewGroup中，很多效果都包含用户手指去拖动其内部的某个View(eg:侧滑菜单等)，
2. 针对具体的需要去写好onInterceptTouchEvent和onTouchEvent这两个方法是一件很不容易的事，需要自己去处理：多手指的处理、加速度检测等等。

### 2. 示例代码 ###
1. 创建实例
2. 触摸相关方法
3. 实现 ViewDragHelper.CallBack() 

		/**初始化 ViewDragHelper*/
   		private void initViewDragHelper() {
        ViewDragHelper mViewDragHelper = ViewDragHelper.create(this, 0.0f,
                new ViewDragHelper.Callback() {
                    @Override
                    public boolean tryCaptureView(View child, int pointerId) {
                        return true;
                    }

                    /**处理水平方向越界*/
                    @Override
                    public int clampViewPositionHorizontal(View child, int left, int dx) {
                        int fixdLeft;
                        View parent = (View) child.getParent();
                        int leftBound = parent.getPaddingLeft();
                        int rightBound = parent.getWidth() - child.getWidth() - parent.getPaddingRight();

                        if (left < leftBound) {
                            fixdLeft = leftBound;
                        } else if (left > leftBound) {
                            fixdLeft = rightBound;
                        } else {
                            fixdLeft = left;
                        }

                        return fixdLeft;
                    }

                    /**处理竖直方向的越界*/
                    @Override
                    public int clampViewPositionVertical(View child, int top, int dy) {
                        int fixedTop;
                        View parent = (View) child.getParent();
                        int topBound = getPaddingTop();
                        int bottomBound = getHeight() - child.getHeight() - parent.getPaddingBottom();
                        if (top < topBound) {
                            fixedTop = topBound;
                        } else if (top > bottomBound) {
                            fixedTop = bottomBound;
                        } else {
                            fixedTop = top;
                        }

                        return fixedTop;
                    }

                    /**监听拖动状态的改变*/
                    @Override
                    public void onViewDragStateChanged(int state) {
                        super.onViewDragStateChanged(state);
                        switch (state) {
                            case ViewDragHelper.STATE_DRAGGING:
                                Log.d("TAG", "STATE_DRAGGING");
                                break;
                            case ViewDragHelper.STATE_IDLE:
                                Log.d("TAG", "STATE_IDLE");
                                break;
                            case ViewDragHelper.STATE_SETTLING:
                                Log.d("TAG", "STATE_SETTLING");
                                break;

                        }
                    }

                    /**捕获 View*/
                    @Override
                    public void onViewCaptured(View capturedChild, int activePointerId) {
                        super.onViewCaptured(capturedChild, activePointerId);
                        Log.d("TAG", "ViewCaptured");

                    }

                    /**释放 View*/
                    @Override
                    public void onViewReleased(View releasedChild, float xvel, float yvel) {
                        super.onViewReleased(releasedChild, xvel, yvel);
                        Log.d("TAG", "ViewReleased");

                    }
                });
	    }
	
	    /**将拦截事件交给 ViewDragHelper 处理*/
	    @Override
	    public boolean onInterceptHoverEvent(MotionEvent event) {
	        return mViewDragHelper.shouldInterceptTouchEvent(event);
	    }
	
	    /**将 Touch 事件交给 ViewDragHelper 处理*/
	    @Override
	    public boolean onTouchEvent(MotionEvent event) {
	        mViewDragHelper.processTouchEvent(event);
	        return true;
	    }

### 3. ViewDragHelper 小结： ###
1. ViewDragHelper 是作用在 ViewGroup上的(比如：Linearlayout),而不是直接作用到某个被拖动的字 View
	- 子 View，在布局中的位置是其所在的 ViewGroup 决定的

2. 示例代码中 ViewDragHelper 做了如下主要操作
	1. VDH 接管了 ViewGroup 的事件拦截
	2. VDH 接管了 ViewG 的Touch 事件
	3. VDH 处理了拖拽子 View 时的边界越界
	4. VDH 监听了拖拽子 View 时的变化
	5. 还可以实现：抽屉拉伸，拖拽结束后子 View 自动归位