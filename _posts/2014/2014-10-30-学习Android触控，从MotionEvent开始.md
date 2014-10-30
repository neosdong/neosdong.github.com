为什么从MotionEvent开始？

## Touch:触控动作的入口

其实，一个触控动作的响应，应该是从**onTouchListener的onTouch**方法开始，或者从**Activity的onTouchEvent**方法开始。
	
	public boolean onTouch(View v, MotionEvent event);
	public boolean onTouchEvent(MotionEvent event);
	 
两者的区别可以从函数原型的参数分辨出来：前者触摸的对象是特定的view，后者是Activity，所以只有MotionEvent参数了。





