# 关于利用SurfaceView绘制抽奖转盘的一些记录

> 前些日子根据公司需求，用Android 实现了一个抽奖大转盘的功能，界面效果为25宫格，十六件奖品（视频后期放入）。经过简单研究后决定采用具有双缓存机制的SurfaceView来实现。整体实现思路也比较简单，废话不多说，进入教程。  

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

2. 绘制基础UI  
	
	因为Canvas中的每一次的绘制行为都会把上一次的清除掉，所以界面呈现的UI,需要在`mHolder.lockCanvas()`之后，` mHolder.unlockCanvasAndPost(canvas)`之前一次完成。

	```java

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

	```

	为了后续便于绘制奖品和奖品动画，我将计算每个绘制每个奖品时需要得到的的Rect对象，提取成一个方法去完成。

	```java
	private Rect getItemRect(int i) {
        	int top = 0, left = 0, right = 0, bottom = 0;
	        if (i >= 16) {
        	    i = i % 16;
	        }
        	if (i < 5) {
	            left = i * (margin + prizeWidth) + margin;
        	    top = margin;
	            right = (i + 1) * (margin + prizeWidth);
        	    bottom = margin + prizeWidth;
	        } else if (i >= 5 && i < 8) {
        	    left = 4 * (margin + prizeWidth) + margin;
	            top = (i - 4) * (margin + prizeWidth) + margin;
        	    right = 5 * (margin + prizeWidth);
	            bottom = (i - 3) * (margin + prizeWidth);
	        } else if (i >= 8 && i < 13) {
        	    left = (12 - i) * (margin + prizeWidth) + margin;
	            top = 4 * (margin + prizeWidth) + margin;
        	    right = (13 - i) * (margin + prizeWidth);
	            bottom = 5 * (margin + prizeWidth);
	        } else if (i >= 13 && i < 16) {
        	    left = margin;
	            top = (16 - i) * (margin + prizeWidth) + margin;
        	    right = margin + prizeWidth;
	            bottom = (17 - i) * (margin + prizeWidth);
	        }
        	return new Rect(left, top, right, bottom);
	}
	```

	绘制奖品时，奖品封装为对象Prize
	```java

	List<Prize> prizeList = new ArrayList();

	public class Prize {
	
	 public Bitmap bitmap;

	}

	/**
	* 绘制奖品     
	*
	* @param canvas
	*/
	private void drawPrizes(final Canvas canvas) {
        	for (int i = 0; i < 16; i++) {
	            if (mSweepstakesInfo == null || mSweepstakesInfo.prizeList == null || mSweepstakesInfo.prizeList.isEmpty())
        	        return;
	            Prize prize = mSweepstakesInfo.prizeList.get(i);
        	    if (prize == null) return;
	            Bitmap bitmap = prize.bitmap;
        	    if (bitmap == null) return;
	            canvas.drawBitmap(bitmap, null, getItemRect(i), null);
	        }
	}

	```

	绘制中心按钮
	```java
 
	private void drawCenter(Canvas canvas) {
        	int left = prizeWidth + margin * 2;
	        int right = (prizeWidth + margin) * 4;
	        Rect rect = new Rect(left, left, right, right);//示例，具体计算方式，详见demo
	        Bitmap bitmap = BitmapFactory.decodeResource(getResources(), R.drawable.sweepstakes_button);
	        canvas.drawBitmap(bitmap, null, rect, null);
	}
	```
3. 生成旋转动画
	> 利用Runnable在绘制基础UI的基础上再去在当前选中奖品上绘制一个选中ui，间隔一定时间绘制一次，达到动画的效果，并且通过控制间隔时间，是动画具有逐渐变慢的效果。	

	自定义Runnable去执行绘制行为
	```java
	class SweepstakesRunnable implements Runnable {

        @Override
        public void run() {
            if (isLottery) {
                Canvas canvas = null;
                try {
                    canvas = mHolder.lockCanvas();
                    if (canvas != null) {
                        drawBg(canvas);
                        drawPrizes(canvas);
                        drawCenter(canvas);
                        drawTransfer(canvas);
                    }
                } catch (Exception e) {
                    e.printStackTrace();
                } finally {
                    if (canvas != null)
                        mHolder.unlockCanvasAndPost(canvas);
                }
            }
            //逐渐变慢的效果
            if (isLottery()) {
                int disCount = Max - count;
                int delayed;
                if (disCount < 3) {
                    delayed = 450;
                } else if (disCount < 8) {
                    delayed = 300;
                } else if (disCount < 16) {
                    delayed = 150;
                } else {
                    delayed = 50;
                }
                mHandler.postDelayed(new SweepstakesRunnable(), delayed);
            } else {
                if (mSweepstakesListener != null) {
                    try {
                        Thread.sleep(1000);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                    if (isError) {
                        isError = false;
                        mSweepstakesListener.onSweepstakesFinish(-1);
                    } else {
                        mSweepstakesListener.onSweepstakesFinish(0);
                    }
                }
            }
         }
	}
	```
	`SweepstakesListener`是为了执行抽奖回调存在的，此处就不附录代码了。

	绘制奖品选中框
	```java
	private void drawTransfer(final Canvas canvas) {
        	if (count <= Max) {
	            Bitmap bitmap = BitmapFactory.decodeResource(getResources(), R.drawable.sweepstakes_checked_icon);
        	    canvas.drawBitmap(bitmap, null, getItemRect(count), null);
	            if (count == Max) {
        	        setLottery(false);
	            }
        	    count++;
	        } else {
        	    setLottery(false);
		}
      	}
	```

4.  通过触摸事件，监听中心区域点击，触发抽奖


	```java

	@Override
	public boolean onTouchEvent(MotionEvent event) {
        	witchPos(event);
	        return super.onTouchEvent(event);
	}

	private void witchPos(MotionEvent event) {
        	// 获取点击屏幕时的点的坐标
       		float x = event.getX();
	        float y = event.getY();
        	int left = (int) ((prizeWidth + margin * 2) + prizeWidth * 0.4);
	        int right = (int) ((prizeWidth + margin) * 4 - prizeWidth * 0.4);
	        int top = (prizeWidth + margin) * 3;
	        int bottom = (int) ((prizeWidth + margin) * 4 - prizeWidth * 0.4);
	        if (x > left && x < right && y > top && y < bottom && event.getAction() == MotionEvent.ACTION_DOWN) {
        	    EventAgent.onEvent(getContext(), EventId.SWEEPSTAKES_APP);
	            startLottery();
	        }
	}
	```

	```java
	private void startLottery() {
        	if (isLottery()) {
	            ToastUtils.show(getContext(), "正在抽奖...");
        	    return;
        	}
	        setLottery(true);
	        count = 0;
	        new SweepstakesRunnable().run();
        	if (mSweepstakesListener != null) {
	            mSweepstakesListener.OnSweepstakesStart();
	        }
	}	

	```
	
5. 在拿到用户抽奖结果后，主动设置抽奖结束位置，结束此次抽奖

	```java
	//absPos 为抽中奖品的绝对位置
	//本篇文章中，默认左上角 position == 1,顺时针旋转 
	public void setEndPos(int absPos) {
        	if (absPos == -1) {
	            isError = true;
        	    setLottery(false);
	            return;
        	}
	        Max = absPos + count + 32 - (count % 16);计算好结束位置后,让它在多转两圈再结束，防止结束过早，略显尴尬。
	}
	```
***
6. End 

好了，到了此处，本片教程就基本结束了，由于写的仓促，示例代码用的是公司代码。便不在此处附录完整代码了。随后有时间我会再次撸出一个demo,上传至GIthub。

