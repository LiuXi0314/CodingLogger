# 关于利用SurfaceView绘制16宫格抽奖转盘的一些记录

> 前些日子根据公司需求，用Android 实现了一个抽奖大转盘的功能，界面效果为十六宫格，十六件奖品（视频后期放入）。经过简单研究后决定采用具有双缓存机制的SurfaceView来实现。整体实现思路也比较简单，废话不多说，进入教程。  

### 核心思想

1. 绘制**基础UI**,其中分为三步骤 

	- 绘制背景；
	- 绘制奖品；
	- 绘制中心按钮；

2. 绘制**抽奖旋转动画**

	- 绘制基础Ui;
	- 绘制奖品选中Ui；
	- 利用线程休眠机制，间隔绘制，生成动画效果；

3. 旋转抽奖控制逻辑

	利用两个int值和一个boolean来控制抽奖逻辑 

	```java
	int count = 0;
	int max = 9999;
	boolean isLottery;
	```
	>count: 表示旋转动画已经走过的步数，每次抽奖开始前重置；  
	>max: 表示抽奖旋转动画的最大步数，当获取到api返回结果后，通过动态修改max值来达到结束抽奖的目的；  
	>isLottery: 旋转开始时 进行操作`isLottery = true`,在旋转过程中，判断到`count == max`时，执行操作`isLottery = false`,终止抽奖。	  

### 实现步骤

1. 实现自定义View SweepstakesView 继承自SurfaceView,并实现`SurfaceHolder.Callback` `Runnable`两个接口

	```java
	public class SweepstakesView extends SurfaceView implements SurfaceHolder.Callback, Runnable {
	
	private SurfaceHolder mHolder;


	public SweepstakesView(Context context) {
        	this(context, null);
	}

        public SweepstakesView(Context context, AttributeSet attrs) {
    	    this(context, attrs, 0);
        }

	public SweepstakesView(Context context, AttributeSet attrs, int defStyleAttr) {
        	super(context, attrs, defStyleAttr);
	        init();
	}


	private void init(){
	   
        getHolder().addCallback(this);
        mHolder = getHolder();
      	/**
	* 无关代码
	**/	
    }
	}

	@Override   
	public void surfaceCreated(SurfaceHolder holder) {
        
		new Thread(this).run();
	
	}

	@Override
	public void surfaceChanged(SurfaceHolder holder, int format, int width, int height) {

	}

	@Override
	public void surfaceDestroyed(SurfaceHolder holder) {

	}

	@Override
	public void run() {
        	
		if (mSweepstakesInfo == null) return;        
		Canvas canvas = null;
        	
		try {
	            canvas = mHolder.lockCanvas();
        	    if (canvas != null) {
                	drawBg(canvas);
	                drawPrizes(canvas);
	                drawCenter(canvas);
        	    }
	        } catch (Exception e) {
        	    e.printStackTrace();
	        } finally {
        	    if (canvas != null)
                	mHolder.unlockCanvasAndPost(canvas);
		}
	}
	```
	以上代码中，`run()`方法是在子线程中去绘制基础UI的操作，共分为三步去执行。绘制背景`drawBg(canvas)`、绘制奖品 `drawPrizes(canvas)`、绘制中心区域按钮 `drawCenter(canvas)`

2.绘制基础UI  
	
	因为Canvas中的每一次的绘制行为都会把上一次的清除掉，所以界面呈现的UI,需要在`mHolder.lockCanvas()`之后，` mHolder.unlockCanvasAndPost(canvas)`之前一次完成。

	/**
	* 绘制背景
	*
	* @param canvas
	*/
	private void drawBg(Canvas canvas) {
        	canvas.drawColor(ContextCompat.getColor(getContext(), bgcolorId), PorterDuff.Mode.CLEAR);
	        int x = DeviceUtils.getDevWidth();
        	Rect r = new Rect(0, 0, x, x);
	        Paint p = new Paint(Paint.ANTI_ALIAS_FLAG);//Paint.ANTI_ALIAS_FLAG 加上此属性，使绘制出的UI具有抗锯齿效果；
        	p.setColor(ContextCompat.getColor(getContext(), bgcolorId));
	        canvas.drawRect(r, p);
	}

	
	



