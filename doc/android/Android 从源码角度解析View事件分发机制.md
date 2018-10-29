###读书笔记--从源码角度剖析Android View事件分发机制

>&#160;&#160;&#160;&#160;&#160;&#160;&#160;在开始描述问题之前先说点题外话，写这篇文章的初衷一方面为了构建Android知识体系，另一方面是真心觉得这个是Android面试必问的知识点。网上这方面的博客和书籍讲解这方面的知识也不少，讲的也很到位。正所谓只有自己理解了才是自己的，所以在阅读了他们的文章后，加上自己的理解特此记录一篇～，以便加深理解和记忆！如理解有误的地方请留言说明，我们一起探讨，谢谢！
>
>联系方式：邮箱（ixiyan.li@gmail.com）

### 1.必备知识点
&#160;&#160;&#160;&#160;&#160;&#160;&#160;事件的分发说白了，就是用户与应用的交互过程（手指与屏幕接触）中，发生的一系列事件传递与处理过程。

#### 1.1 事件分发涉及的对象--MotionEvent

典型事件类型：

```
ACTION_DOWN——手指刚触碰屏幕那一刻（按下）
ACTION_MOVE——手指在屏幕上移动（移动）
ACTION_UP——手指抬起那一刻（抬起）
```
一个事件序列：就是从手指按下 View 开始直到手指离开 View 产生的一系列事件。

```
ACTION_DOWN-> ACTION_UP
ACTION_DOWN->...ACTION_MOVE...->ACTION_UP
``` 
#### 1.2 事件分发涉及的方法

##### 1. dispatchTouchEvent(MotionEvent ev) 
用来进行事件分发。返回结果受当前 View 的 onTouchEvent 和子 View 的 dispatchTouchEvent 方法的影响，表示是否消耗当前事件。

##### 2. onInterceptTouchEvent(MotionEvent ev)
在上述`dispatchTouchEvent `方法内部调用，用来进行当前事件是否拦截校验。这里有一点要注意的地方就是如果当前View拦截了某个事件（一般指ACTION_DOWN），那么在同一个`事件序列`（上面讲过这个概念）当中,此方法不会被再次调用——即不会做二次拦截校验。
注：Activity和View内部没有此方法

##### 3. onTouchEvent(MotionEvent ev)
在上述`dispatchTouchEvent `方法内部调用，返回结果表示是否消耗当前事件。这里同上也有一点要注意，如果当前方法返回`false`（不消耗）,那么同一个`事件序列`中，当前View无法再次接收到事件。

上述方法的关系可用下面的一段伪代码表示：

```
public boolean dispatchTouchEvent(MotionEvetn e){ 
	if(onInterceptTouchEvent(ev)){//是否拦截
		return onTouchEvent(e);//拦截事件处理：是否消耗
	}
	return child.dispatchTouchEvent(e);//不拦截：子类View分发
}
```
通过上面的伪代码可以大致了解到事件的传递规则：对于一个根`ViewGroup`来说，点击事件产生后，首先会传递给它，这时它的`dispatchTouchEvent`就会被调用，如果这个`ViewGroup`的`onInterceptTouchEvent`方法返回`true`,就说明拦截当前事件，接着事件就会交给这个`ViewGroup`的`onTouchEvent`方法处理。反之`onInterceptTouchEvent`方法返回`false`，就不拦截当前事件，这时当前事件就会传递给它的子`View`，接着`View`的`dispatchTouchEvent`方法就会调用，如此反复直到事件最终被处理。

#### 1.3 事件传递过程遵循如下过程 
```
Activity -> Windown(PhoneWindow) -> DecorView(FrameLayout) -> contentView(setContentView) ->..ViewGroup..->View
``` 

### 2. 事件分发源码解析
根据上面了解到的事件传递的过程分析，下面我们就一步一步撕开它神秘的面纱，从内部了解它的调用关系。
#### 2.1 Activity对点击事件的分发过程
点击事件用`MontionEvent`表示，当一个点击操作发生时，最先传递给当前`Activity`,由`Activity`的`dispatchTouchEvent`方法进行事件分发，具体的工作由`Window`来完成。`Window`会将事件传递给`DecorView`，`DecorView`一般就是当前界面的底层容器（即setContentView所设置的 `View` 的父容器），通过Activity.getWindow().getDecorView()可以获得。因此我们先从`Activity`的`dispatchTouchEvent`开始分析。

**源码-1：Activity#dispatchTouchEvent**

```
/**
* Called to process touch screen events.  You can override this to
* intercept all touch screen events before they are dispatched to the
* window.  Be sure to call this implementation for touch screen events
* that should be handled normally.
*
* @param ev The touch screen event.
*
* @return boolean Return true if this event was consumed.
*/
public boolean dispatchTouchEvent(MotionEvent ev) {
    if (ev.getAction() == MotionEvent.ACTION_DOWN) {
            onUserInteraction();
        }
        if (getWindow().superDispatchTouchEvent(ev)) {
            return true;
        }
        return onTouchEvent(ev);
    }
    
public boolean onTouchEvent(MotionEvent event) {
    if (mWindow.shouldCloseOnTouch(this, event)) {
        finish();
        return true;
    }

    return false;
}    
```

*现在分析上述代码，通过源码了解到事件交给Activity所附属的Window进行分发，如果`getWindow().superDispatchTouchEvent(ev)`返回`true`，事件到此结束，返回`false `，说明下级所有View的`onTouchEvent`都返回了`false`,则Activity的`onTouchEvent`将会被调用（如上）*

通过上面了解到`getWindow().superDispatchTouchEvent(ev)`这个才是分发的关键，看源码：

**源码-2：Window#superDispatchTouchEvent**

```

/**
 * Abstract base class for a top-level window look and behavior policy.  An
 * instance of this class should be used as the top-level view added to the
 * window manager. It provides standard UI policies such as a background, title
 * area, default key processing, etc.
 *
 * <p>The only existing implementation of this abstract class is
 * android.view.PhoneWindow, which you should instantiate when needing a
 * Window.
 */
public abstract class Window {
	/**
	 * Used by custom windows, such as Dialog, to pass the touch screen event
	 * further down the view hierarchy. Application developers should
	 * not need to implement or call this.
	 *
	 */
	public abstract boolean superDispatchTouchEvent(MotionEvent event);
	...
}
```
*看上面贴的源码发现贴了好多注释说明，因为这里`Window`是个抽象类，那么它的实现类是什么呢，是`PhoneWindow`，为什么呢？到这里您可以详细阅读下上面`Window`类的说明，发现此处已经指明了`Window`的唯一实现就是`android.view.PhoneWindow`，好家伙，隐藏的够深的，那么请移驾，谢谢～*

**源码-3：PhoneWindow#superDispatchTouchEvent相关代码**

```
// This is the top-level view of the window, containing the window decor.
private DecorView mDecor;
@Override
public boolean superDispatchTouchEvent(MotionEvent event) {
    return mDecor.superDispatchTouchEvent(event);
}
```
*到这里逻辑就清晰了吧！虽然代码只有一行，但已经足以说明问题了，此处具体逻辑移交给`DecorView`(这就是我们前面说的窗口的顶级View-->ViewGroup)，即`Activity#setContentView`设置的`View`就是`DecorView`的子View。目前事件传递到了`DecorView`这里，由于`DecorVieW`即成自`FrameLayout`且是父`View`，那么得出结论--最终事件会传递给`View`，到这一步并不是我们的重点，事件如何通过顶级`View`进行传递消费才是我们的重头戏，请继续，谢谢～*

#### 2.2 顶级View对点击事件的分发过程
关于点击事件如何在`View`中进行分发，上面已经做了描述，这里就直接上`ViewGroup`源码，源码如下：

`dispatchTouchEvent`方法内容较多分如下几个片段说明：

**源码-4：ViewGroup#dispatchTouchEvent——拦截逻辑处理**

```
// Handle an initial down.
if (actionMasked == MotionEvent.ACTION_DOWN) {
    // Throw away all previous state when starting a new touch gesture.
    // The framework may have dropped the up or cancel event for the previous gesture
    // due to an app switch, ANR, or some other state change.
    cancelAndClearTouchTargets(ev);
    resetTouchState();
}
// Check for interception.
final boolean intercepted;
if (actionMasked == MotionEvent.ACTION_DOWN
        || mFirstTouchTarget != null) {
    final boolean disallowIntercept = (mGroupFlags & FLAG_DISALLOW_INTERCEPT) != 0;
    if (!disallowIntercept) {
        intercepted = onInterceptTouchEvent(ev);
        ev.setAction(action); // restore action in case it was changed
    } else {
        intercepted = false;
    }
} else {
    // There are no touch targets and this action is not an initial down
    // so this view group continues to intercept touches.
    intercepted = true;
}
```
1. 是否拦截条件：事件类型为`ACTION_DOWN || mFirstTouchTarget != null`;
2. mFirstTouchTarget：每次开始(`ACTION_DOWN`)都会被初始化为`null`,当事件由`ViewGroup`的子元素成功处理时，它指向子元素；
3. 当事件由`ViewGroup`拦截时，条件`mFirstTouchTarget != null`不成立，即当`ACTION_MOVE`和 `ACTION_UP`事件到来时，由于第一条拦截条件不满足，则`onInterceptTouchEvent`不再调用：应证了一旦当前View拦截事件，那么同一事件序列的其它事件都不再进行拦截校验，直接交给它处理。
4. `FLAG_DISALLOW_INTERCEPT`标记位：这个标记位一旦设置后(`requestDisallowInterceptTouchEvent`)，`ViewGroup`将无法拦截除了`ACTION_DOWN`以外的其它点击事件（`ACTION_DOWN`事件会重置此标记位,将导致子View中设置的这个标记位无效）。
5. 面对`ACTION_DOWN`事件时，`ViewGroup`总是会调用自己的`onInterceptTouchEvent`方法来询问自己是否要拦截事件，这一点从上面的源码中可以看出来。

**源码-5：ViewGroup#dispatchTouchEvent——初始化**

```
// Handle an initial down.
if (actionMasked == MotionEvent.ACTION_DOWN) {
    // Throw away all previous state when starting a new touch gesture.
    // The framework may have dropped the up or cancel event for the previous gesture
    // due to an app switch, ANR, or some other state change.
    cancelAndClearTouchTargets(ev);//重置 mFirstTouchTarget = null
    resetTouchState();//重置FLAG_DISALLOW_INTERCEPT标记位
}
```

*从上面的代码可以看出，`ViewGroup`会在`ACTION_DOWN`事件到来时会做重置状态的操作，因此子`View`调用`requestDisallowInterceptTouchEvent`并不能影响`ViewGroup`对`ACTION_DOWN`事件的处理。*

总结：

1. `ViewGroup`决定拦截事件（`ACTION_DOWN`）后，那么后续的点击事件将会默认交给它处理且不再调用它的`onInterceptTouchEvent`方法。 
2. <font color="#ff0000">`FLAG_DISALLOW_INTERCEPT`这个标志的作用是让`ViewGroup`不再拦截事件，当然前提是`ViewGroup`不拦截`ACTION_DOWN`事件</font>。
3. `FLAG_DISALLOW_INTERCEPT`为解决滑动冲突解决提供了新的思路。


**源码-6：ViewGroup#dispatchTouchEvent——不拦截，遍历子View**

```
final int childrenCount = mChildrenCount;
if (newTouchTarget == null && childrenCount != 0) {
    final float x = ev.getX(actionIndex);
    final float y = ev.getY(actionIndex);
    // Find a child that can receive the event.
    // Scan children from front to back.
    final ArrayList<View> preorderedList = buildTouchDispatchChildList();
    final boolean customOrder = preorderedList == null
            && isChildrenDrawingOrderEnabled();
    final View[] children = mChildren;
    for (int i = childrenCount - 1; i >= 0; i--) {
        final int childIndex = getAndVerifyPreorderedIndex(
                childrenCount, i, customOrder);
        final View child = getAndVerifyPreorderedView(
                preorderedList, children, childIndex);

        // If there is a view that has accessibility focus we want it
        // to get the event first and if not handled we will perform a
        // normal dispatch. We may do a double iteration but this is
        // safer given the timeframe.
        if (childWithAccessibilityFocus != null) {
            if (childWithAccessibilityFocus != child) {
                continue;
            }
            childWithAccessibilityFocus = null;
            i = childrenCount - 1;
        }

        if (!canViewReceivePointerEvents(child)
                || !isTransformedTouchPointInView(x, y, child, null)) {
            ev.setTargetAccessibilityFocus(false);
            continue;
        }

        newTouchTarget = getTouchTarget(child);
        if (newTouchTarget != null) {
            // Child is already receiving touch within its bounds.
            // Give it the new pointer in addition to the ones it is handling.
            newTouchTarget.pointerIdBits |= idBitsToAssign;
            break;
        }

        resetCancelNextUpFlag(child);
        if (dispatchTransformedTouchEvent(ev, false, child, idBitsToAssign)) {//*子元素调用dispatchTouchEvent方法*
            // Child wants to receive touch within its bounds.
            mLastTouchDownTime = ev.getDownTime();
            if (preorderedList != null) {
                // childIndex points into presorted list, find original index
                for (int j = 0; j < childrenCount; j++) {
                    if (children[childIndex] == mChildren[j]) {
                        mLastTouchDownIndex = j;
                        break;
                    }
                }
            } else {
                mLastTouchDownIndex = childIndex;
            }
            mLastTouchDownX = ev.getX();
            mLastTouchDownY = ev.getY();
            //保存当前子View:mFirstTouchTarget
            newTouchTarget = addTouchTarget(child, idBitsToAssign);
            alreadyDispatchedToNewTouchTarget = true;
            break;
        }

        // The accessibility focus didn't handle the event, so clear
        // the flag and do a normal dispatch to all children.
        ev.setTargetAccessibilityFocus(false);
    }
    if (preorderedList != null) preorderedList.clear();
    //...
}
```
**源码-7：ViewGroup#dispatchTouchEvent——子View下发主要逻辑调用**

```
/**
 * Transforms a motion event into the coordinate space of a particular child view,
 * filters out irrelevant pointer ids, and overrides its action if necessary.
 * If child is null, assumes the MotionEvent will be sent to this ViewGroup instead.
 */
private boolean dispatchTransformedTouchEvent(MotionEvent event, boolean cancel,
        View child, int desiredPointerIdBits) {
    final boolean handled;

    // Canceling motions is a special case.  We don't need to perform any transformations
    // or filtering.  The important part is the action, not the contents.
    final int oldAction = event.getAction();
    if (cancel || oldAction == MotionEvent.ACTION_CANCEL) {
        event.setAction(MotionEvent.ACTION_CANCEL);
        if (child == null) {
            handled = super.dispatchTouchEvent(event);
        } else {
            handled = child.dispatchTouchEvent(event);
        }
        event.setAction(oldAction);
        return handled;
    }
    //...
}
```
子`View`是否能够接收点击事件有以下两点衡量：

* 子元素是否在播放动画
* 点击事件的坐标是否落在子元素的区域内


*上面这部分代码说明的是`ViewGroup`不拦截情况下，事件向子`View`下发的过程.即主要调用方法为`dispatchTransformedTouchEvent`，它的内部实际上调用的就是子元素的`dispatchTouchEvent`方法(可通过上面的**源码-7**看得出来).通过具体分析可看出，如果`child.dispatchTouchEvent(event)`返回`true`，那么`mFirstTouchTarget`(`addTouchTarget`方法内部操作)就会被赋值同时跳出for循环,这里是否对`mFirstTouchTarget`赋值，将会影响`ViewGroup`的拦截策略，如下所示：*

**源码-8：ViewGroup#dispatchTouchEvent——赋值mFirstTouchTarget**

```
/**
 * Adds a touch target for specified child to the beginning of the list.
 * Assumes the target child is not already present.
 */
private TouchTarget addTouchTarget(@NonNull View child, int pointerIdBits) {
    final TouchTarget target = TouchTarget.obtain(child, pointerIdBits);
    target.next = mFirstTouchTarget;
    mFirstTouchTarget = target;
    return target;
}
```
*`mFirstTouchTarget`如果为`null`,将会默认拦截接下来同一序列的所有事件。（不做二次拦截校验）*

遍历所有子元素，都没有处理包含两种情况：

1. `ViewGroup`没有子元素;
2. 子元素处理了点击事件，但是所有的子元素都没有消耗事件。

此时`ViewGroup`将会调用`super.dispatchTouchEvent(evet)`,这一点可以从上述**源码-8**可以看出，很显然这里`ViewGroup`继承自`View`,所以这里就转到`View`的`dispatchTouchEvent`方法，即点击事件交由`View`处理，那么请继续看下面的分析。


#### 2.3 View对点击事件的处理过程
View（不包含ViewGroup）对点击事件的处理稍微简单，它没有`onInterceptTouchEvent`方法且无法向下传递事件，只能自己处理，请看它的`dispatchTouchEvent`方法，如下：

**源码-9：View#dispatchTouchEvent——View点击事件处理**

```
/**
* Pass the touch screen motion event down to the target view, or this
* view if it is the target.
*
* @param event The motion event to be dispatched.
* @return True if the event was handled by the view, false otherwise.
*/
public boolean dispatchTouchEvent(MotionEvent event) {
// If the event should be handled by accessibility focus first.
//...
boolean result = false;
//...
if (onFilterTouchEventForSecurity(event)) {
    if ((mViewFlags & ENABLED_MASK) == ENABLED && handleScrollBarDragging(event)) {
        result = true;
    }
    //noinspection SimplifiableIfStatement
    ListenerInfo li = mListenerInfo;
    if (li != null && li.mOnTouchListener != null
            && (mViewFlags & ENABLED_MASK) == ENABLED
            && li.mOnTouchListener.onTouch(this, event)) {
        result = true;
    }

    if (!result && onTouchEvent(event)) {
        result = true;
    }
}
//...
return result;
}
```
*从上面的代码可以看出：`OnTouchListener的onTouch`比`onTouchEvent(event)`优先级高，如果设置了`OnTouchListener`且`mOnTouchListener.onTouch`返回`true`那么`onTouchEvent(event)`将不会调用,反之将会调用`onTouchEvent(event)`,见下文：*


**源码-10：View#onTouchEvent——点击事件具体处理**

```
public boolean onTouchEvent(MotionEvent event) {
final float x = event.getX();
final float y = event.getY();
final int viewFlags = mViewFlags;
final int action = event.getAction();
	
final boolean clickable = ((viewFlags & CLICKABLE) == CLICKABLE
    || (viewFlags & LONG_CLICKABLE) == LONG_CLICKABLE)
    || (viewFlags & CONTEXT_CLICKABLE) == CONTEXT_CLICKABLE;
	
if ((viewFlags & ENABLED_MASK) == DISABLED) {
	if (action == MotionEvent.ACTION_UP && (mPrivateFlags & PFLAG_PRESSED) != 0) {
	    setPressed(false);
	}
mPrivateFlags3 &= ~PFLAG3_FINGER_DOWN;
	// A disabled view that is clickable still consumes the touch
	// events, it just doesn't respond to them.
	return clickable;
}
if (mTouchDelegate != null) {
	if (mTouchDelegate.onTouchEvent(event)) {
	    return true;
	}
}
	
	if (clickable || (viewFlags & TOOLTIP) == TOOLTIP) {
	switch (action) {
	    case MotionEvent.ACTION_UP:
	        mPrivateFlags3 &= ~PFLAG3_FINGER_DOWN;
	        if ((viewFlags & TOOLTIP) == TOOLTIP) {
	            handleTooltipUp();
	        }
	        if (!clickable) {
	            removeTapCallback();
	            removeLongPressCallback();
	            mInContextButtonPress = false;
	            mHasPerformedLongPress = false;
	            mIgnoreNextUpEvent = false;
	            break;
	        }
	        boolean prepressed = (mPrivateFlags & PFLAG_PREPRESSED) != 0;
	        if ((mPrivateFlags & PFLAG_PRESSED) != 0 || prepressed) {
	            // take focus if we don't have it already and we should in
	            // touch mode.
	            boolean focusTaken = false;
	            if (isFocusable() && isFocusableInTouchMode() && !isFocused()) {
	                focusTaken = requestFocus();
	            }
	
	            if (prepressed) {
	                // The button is being released before we actually
	                // showed it as pressed.  Make it show the pressed
	                // state now (before scheduling the click) to ensure
	                // the user sees it.
	                setPressed(true, x, y);
	            }
	
	            if (!mHasPerformedLongPress && !mIgnoreNextUpEvent) {
	                // This is a tap, so remove the longpress check
	                removeLongPressCallback();
	
	                // Only perform take click actions if we were in the pressed state
	                if (!focusTaken) {
	                    // Use a Runnable and post this rather than calling
	                    // performClick directly. This lets other visual state
	                    // of the view update before click actions start.
	                    if (mPerformClick == null) {
	                        mPerformClick = new PerformClick();
	                    }
	                    if (!post(mPerformClick)) {
	                        performClick();
	                    }
	                }
	            }
	
	            if (mUnsetPressedState == null) {
	                mUnsetPressedState = new UnsetPressedState();
	            }
	
	            if (prepressed) {
	                postDelayed(mUnsetPressedState,
	                        ViewConfiguration.getPressedStateDuration());
	            } else if (!post(mUnsetPressedState)) {
	                // If the post failed, unpress right now
	                mUnsetPressedState.run();
	            }
	
	            removeTapCallback();
	        }
	        mIgnoreNextUpEvent = false;
	        break;
	
	//...
	   	}
	
	return true;
	}
	
	return false;
}
```
*从上面的代码看出:影响事件的消耗因素有两个：`CLICKABLE`和`LONG_CLICKABLE`只要有一个为`true`,那么它就会消耗这个事件，即`onTouchEvent`方法返回`true`，实际调用方法为`performClick();`,在其内部调用`OnClickListener#onClick`方法。*

到此点击事件的分发机制的源码分析就完了，但是Android
的学习才刚开始，还有很长的路要走，下面附上从别处盗来的图，觉得不错可以看下

#### 2.4View事件分发流程图示例图
![事件分发流程图](images/view-dispatch)

### 参考文章与相关书籍

[Android 事件分发](https://juejin.im/post/5b6bb82e6fb9a04fab45399a)

[Android事件分发机制解析](https://www.jianshu.com/p/153031fb9e7f)

书籍：任玉刚的《Android开发艺术探索》