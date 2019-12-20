---
title: android-view
date: 2018-10-15 00:07:03
tags:
---

### View遍历

requestLayout()

ViewRootImpl 和 ViewGroup 一样实现了ViewParent接口。但它本身不是View树的一员，主要负责和WMS通信:

-  scheduleTraversals()
-  mChoreographer.postCallback(Choreographer.CALLBACK_TRAVERSAL, mTraversalRunnable, null);
-  doTraversal()
-  performTraversals();

performTraversals 会触发测、布、 绘三种动作:

1. performMeasure( ) -> mView.measure(childWidthMeasureSpec, childHeightMeasureSpec);
2. performDraw( ) -> host.layout(0, 0, host.getMeasuredWidth(), host.getMeasuredHeight());
3. performLayout( ) -> drawSoftware() -> mView.draw(canvas);

invalidate()

回溯到ViewRoot，回溯过程会加上标志位，然后ViewRoot会进行一次遍历。

invalidate() -> parent.invalidateChild(this, damage) ->

不断调用invalidateChildInParent直到返回null，也就是找到ViewRootImpl



回溯完成后会触发一次遍历(performTraversals)，但是不会触发测和布:



1. 调用onDraw()画自己
2. 如果是ViewGroup则调用dispatchDraw画儿子
3. 儿子成父亲，继续遍历


### Touch事件分发

叶子不需要向下遍历,所以叶子的分发函数只有消费的实现。先尝试由OnTouchListener来消费，后面再是在onTouchEvent消费

~~~
public boolean dispatchTouchEvent(MotionEvent ev) {
    // ....
    if (li.mOnTouchListener.onTouch(this, event)) {
        return true;
    }
    if (onTouchEvent(event)) {
        return true;
    }
    // ....
    return false;
}
~~~

ViewGroup的dispatch则要考虑更多：

- 是不是自己拦截来处理？
- 如果不拦截要交给哪个来处理？

不管怎样最终要确定这次dispatch是不是处理了，一般来说，只要有一个节点的dispatch返回true就是整个view树这次事件分发的结束标识

~~~
View mTarget=null;  //保存捕获Touch事件处理的子View

public boolean dispatchTouchEvent(MotionEvent ev) {

    //....

    // 在分发Down事件时确定哪个子View是分发的目标

    if(ev.getAction()==KeyEvent.ACTION_DOWN) {
        if(!onInterceptTouchEvent()) {  //不拦截ACTION_DOWN
        mTarget=null; //目标View置为null
        // 采用深度优先遍历，childs[0]比childs[1]更优先接收
        View[] childs = getChildView();
        for (int i = 0; i < childs.length; i++) {
            if (childs[i].dispatchTouchEvent(ev)) {
                // 找到了分发的目标
                mTarget=childs[i];
                return true;
            }
        }
      }
  }

    //当子View没有捕获down事件时，ViewGroup自身处理。这里处理的Touch事件包含Down、Up和Move
    if(mTarget == null) {
        return super.dispatchTouchEvent(ev);
    }

    if(onInterceptTouchEvent()){
         //...其他处理，在此不管
    }

   // 已经捕获了DOWN事件的子View则直接给他分发事件了
   //这一步在Action_Down中是不会执行到的，只有Move和UP才会执行到。
    return mTarget.dispatchTouchEvent(ev);
}
~~~
