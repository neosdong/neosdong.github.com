---
layout: post
category : program
tags : [Android, 动作响应, 滑动]

---


左右滑动动作是基于Fling动作（快速连续滑动，不区分方向）的。

Fling动作是基于Touch动作的，所以首先触发响应函数onTouch，然后在onFling中使用GestureDetector.onTouchEvent(event ev)分析动作，判断是否要触发OnGestureListener的响应函数。

OnGestureListener的响应函数：

* onFling：快速连续滑动，不区分方向。


## 左右滑动动作响应的实现

### 运行顺序
Activity implements OnTouchListener

    /*设置滑动响应*/
    someView.setOnTouchListener(this);

OnTouchListener

    /*触发onTouch响应函数*/
    @Override
	public boolean onTouch(View v, MotionEvent event) {
		/*使用GestureDetector.onTouchEvent分析动作，并给OnGestureListener做深度响应*/
		return mGestureDetector.onTouchEvent(event);
	}  

GestureDetector.SimpleOnGestureListener

    /*触发onFling响应函数*/
    onFling(){
    	/*定义Swipe Left and Right的条件*/
    }
    
### 定义Swipe动作

GestureDetector.SimpleOnGestureListener接口定义了Fling动作的标准。Swipe动作是基于Fling动作的，所以实现该接口的时候，要复写onFling响应函数。

            @Override
        public boolean onFling(MotionEvent e1, MotionEvent e2, float velocityX,
                               float velocityY) {

            float off_path_dis = e1.getY()-e2.getY();
            Log.d(TAG, "onFling:"+String.valueOf(off_path_dis));

            if (SWIPE_MAX_OFF_PATH < Math.abs(off_path_dis)){
                Log.d(TAG,"off path");
                return false;
            }
            /**
             * 判断左右
             */
            //右
            if ((e2.getX()-e1.getX()>SWIPE_MIN_DISTANCE && SWIPE_THRESHOLD_VELOCITY< velocityX)){
                Log.d(TAG,"swipe right:"+Math.abs(e2.getX()-e1.getX()));
            }
            //左
            else if ((e1.getX()-e2.getX()>SWIPE_MIN_DISTANCE && SWIPE_THRESHOLD_VELOCITY<Math.abs(velocityX))){
                Log.d(TAG,"swipe left:" + Math.abs(e1.getX()-e2.getX()));

            }

            return super.onFling(e1, e2, velocityX, velocityY);
        }

### Demo

* [MOTION-touch_and_swipe](http://pan.baidu.com/s/1jG7lbPO)




### 参考
* GestureDetector.SimpleOnGestureListener - Android Developers
http://developer.android.com/reference/android/view/GestureDetector.SimpleOnGestureListener.html
* Android tutorial on Event Listeners - Edureka
http://www.edureka.co/blog/android-tutorials-event-listeners
