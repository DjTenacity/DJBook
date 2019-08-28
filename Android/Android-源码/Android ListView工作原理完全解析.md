# [Android ListView工作原理完全解析](http://blog.csdn.net/guolin_blog/article/details/44996879)

> 在Android所有常用的原生控件当中，用法最复杂的应该就是ListView了，它专门用于处理那种内容元素很多，手机屏幕无法展示出所有内容的情况。ListView可以使用列表的形式来展示内容，超出屏幕部分的内容只需要通过手指滑动就可以移动到屏幕内了。
>
> 
>
> 另外ListView还有一个非常神奇的功能，我相信大家应该都体验过，即使在ListView中加载非常非常多的数据，比如达到成百上千条甚至更多，ListView都不会发生OOM或者崩溃，而且随着我们手指滑动来浏览更多数据时，程序所占用的内存竟然都不会跟着增长。那么ListView是怎么实现这么神奇的功能的呢？当初我就抱着学习的心态花了很长时间把ListView的源码通读了一遍，基本了解了它的工作原理，在感叹Google大神能够写出如此精妙代码的同时我也有所敬畏，因为ListView的代码量比较大，复杂度也很高，很难用文字表达清楚，于是我就放弃了把它写成一篇博客的想法。那么现在回想起来这件事我已经肠子都悔青了，因为没过几个月时间我就把当初梳理清晰的源码又忘的一干二净。于是现在我又重新定下心来再次把ListView的源码重读了一遍，那么这次我一定要把它写成一篇博客，分享给大家的同时也当成我自己的笔记吧。 

### ListView的继承结构
如下图所示：

首先看一下ListView的继承结构，如下图所示：



![img](https://img-blog.csdn.net/20150704111744498?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)





可以看到，ListView的继承结构还是相当复杂的，它是直接继承自的AbsListView，而AbsListView有两个子实现类，一个是ListView，另一个就是GridView，因此我们从这一点就可以猜出来，ListView和GridView在工作原理和实现上都是有很多共同点的。然后AbsListView又继承自AdapterView，AdapterView继承自ViewGroup，后面就是我们所熟知的了。先把ListView的继承结构了解一下，待会儿有助于我们更加清晰地分析代码。



### Adapter的作用

Adapter相信大家都不会陌生，我们平时使用ListView的时候一定都会用到它。那么话说回来大家有没有仔细想过，为什么需要Adapter这个东西呢？总感觉正因为有了Adapter，ListView的使用变得要比其它控件复杂得多。那么这里我们就先来学习一下Adapter到底起到了什么样的一个作用。

其实说到底，控件就是为了交互和展示数据用的，只不过ListView更加特殊，它是为了展示很多很多数据用的，但是ListView只承担交互和展示工作而已，至于这些数据来自哪里，ListView是不关心的。因此，我们能设想到的最基本的ListView工作模式就是要有一个ListView控件和一个数据源

不过如果真的让ListView和数据源直接打交道的话，那ListView所要做的适配工作就非常繁杂了。因为数据源这个概念太模糊了，我们只知道它包含了很多数据而已，至于这个数据源到底是什么样类型，并没有严格的定义，有可能是数组，也有可能是集合，甚至有可能是数据库表中查询出来的游标。所以说如果ListView真的去为每一种数据源都进行适配操作的话，一是扩展性会比较差，内置了几种适配就只有几种适配，不能动态进行添加。二是超出了它本身应该负责的工作范围，不再是仅仅承担交互和展示工作就可以了，这样ListView就会变得比较臃肿。



那么显然Android开发团队是不会允许这种事情发生的，于是就有了Adapter这样一个机制的出现。顾名思义，Adapter是适配器的意思，它在ListView和数据源之间起到了一个桥梁的作用，ListView并不会直接和数据源打交道，而是会借助Adapter这个桥梁来去访问真正的数据源，与之前不同的是，Adapter的接口都是统一的，因此ListView不用再去担心任何适配方面的问题。而Adapter又是一个接口(interface)，它可以去实现各种各样的子类，每个子类都能通过自己的逻辑来去完成特定的功能，以及与特定数据源的适配操作，比如说ArrayAdapter可以用于数组和List类型的数据源适配，SimpleCursorAdapter可以用于游标类型的数据源适配，这样就非常巧妙地把数据源适配困难的问题解决掉了，并且还拥有相当不错的扩展性。简单的原理示意图如下所示：

![](https://img-blog.csdn.net/20150719202924532?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)



当然Adapter的作用不仅仅只有数据源适配这一点，还有一个非常非常重要的方法也需要我们在Adapter当中去重写，就是getView()方法，这个在下面的文章中还会详细讲到。

 

### RecycleBin机制

那么在开始分析ListView的源码之前，还有一个东西是我们提前需要了解的，就是RecycleBin机制，这个机制也是ListView能够实现成百上千条数据都不会OOM最重要的一个原因。

其实RecycleBin的代码并不多，只有300行左右，它是写在AbsListView中的一个内部类，所以所有继承自AbsListView的子类，也就是ListView和GridView，都可以使用这个机制。那我们来看一下RecycleBin中的主要代码，如下所示 

```java
class RecycleBin {
	private RecyclerListener mRecyclerListener;
 
	/**
	 * The position of the first view stored in mActiveViews.
	 */
	private int mFirstActivePosition;
 
	/**
	 * Views that were on screen at the start of layout. This array is
	 * populated at the start of layout, and at the end of layout all view
	 * in mActiveViews are moved to mScrapViews. Views in mActiveViews
	 * represent a contiguous range of Views, with position of the first
	 * view store in mFirstActivePosition.
	 */
	private View[] mActiveViews = new View[0];
 
	/**
	 * Unsorted views that can be used by the adapter as a convert view.
	 */
	private ArrayList<View>[] mScrapViews;
 
	private int mViewTypeCount;
 
	private ArrayList<View> mCurrentScrap;
 
	/**
	 * Fill ActiveViews with all of the children of the AbsListView.
	 * 
	 * @param childCount
	 *            The minimum number of views mActiveViews should hold
	 * @param firstActivePosition
	 *            The position of the first view that will be stored in
	 *            mActiveViews
	 */
	void fillActiveViews(int childCount, int firstActivePosition) {
		if (mActiveViews.length < childCount) {
			mActiveViews = new View[childCount];
		}
		mFirstActivePosition = firstActivePosition;
		final View[] activeViews = mActiveViews;
		for (int i = 0; i < childCount; i++) {
			View child = getChildAt(i);
			AbsListView.LayoutParams lp = (AbsListView.LayoutParams) child.getLayoutParams();
			// Don't put header or footer views into the scrap heap
			if (lp != null && lp.viewType != ITEM_VIEW_TYPE_HEADER_OR_FOOTER) {
				// Note: We do place AdapterView.ITEM_VIEW_TYPE_IGNORE in
				// active views.
				// However, we will NOT place them into scrap views.
				activeViews[i] = child;
			}
		}
	}
 
	/**
	 * Get the view corresponding to the specified position. The view will
	 * be removed from mActiveViews if it is found.
	 * 
	 * @param position
	 *            The position to look up in mActiveViews
	 * @return The view if it is found, null otherwise
	 */
	View getActiveView(int position) {
		int index = position - mFirstActivePosition;
		final View[] activeViews = mActiveViews;
		if (index >= 0 && index < activeViews.length) {
			final View match = activeViews[index];
			activeViews[index] = null;
			return match;
		}
		return null;
	}
 
	/**
	 * Put a view into the ScapViews list. These views are unordered.
	 * 
	 * @param scrap
	 *            The view to add
	 */
	void addScrapView(View scrap) {
		AbsListView.LayoutParams lp = (AbsListView.LayoutParams) scrap.getLayoutParams();
		if (lp == null) {
			return;
		}
		// Don't put header or footer views or views that should be ignored
		// into the scrap heap
		int viewType = lp.viewType;
		if (!shouldRecycleViewType(viewType)) {
			if (viewType != ITEM_VIEW_TYPE_HEADER_OR_FOOTER) {
				removeDetachedView(scrap, false);
			}
			return;
		}
		if (mViewTypeCount == 1) {
			dispatchFinishTemporaryDetach(scrap);
			mCurrentScrap.add(scrap);
		} else {
			dispatchFinishTemporaryDetach(scrap);
			mScrapViews[viewType].add(scrap);
		}
 
		if (mRecyclerListener != null) {
			mRecyclerListener.onMovedToScrapHeap(scrap);
		}
	}
 
	/**
	 * @return A view from the ScrapViews collection. These are unordered.
	 */
	View getScrapView(int position) {
		ArrayList<View> scrapViews;
		if (mViewTypeCount == 1) {
			scrapViews = mCurrentScrap;
			int size = scrapViews.size();
			if (size > 0) {
				return scrapViews.remove(size - 1);
			} else {
				return null;
			}
		} else {
			int whichScrap = mAdapter.getItemViewType(position);
			if (whichScrap >= 0 && whichScrap < mScrapViews.length) {
				scrapViews = mScrapViews[whichScrap];
				int size = scrapViews.size();
				if (size > 0) {
					return scrapViews.remove(size - 1);
				}
			}
		}
		return null;
	}
 
	public void setViewTypeCount(int viewTypeCount) {
		if (viewTypeCount < 1) {
			throw new IllegalArgumentException("Can't have a viewTypeCount < 1");
		}
		// noinspection unchecked
		ArrayList<View>[] scrapViews = new ArrayList[viewTypeCount];
		for (int i = 0; i < viewTypeCount; i++) {
			scrapViews[i] = new ArrayList<View>();
		}
		mViewTypeCount = viewTypeCount;
		mCurrentScrap = scrapViews[0];
		mScrapViews = scrapViews;
	}
 
} 
```





这里的RecycleBin代码并不全，我只是把最主要的几个方法提了出来。那么我们先来对这几个方法进行简单解读，这对后面分析ListView的工作原理将会有很大的帮助。

+ **fillActiveViews()** 这个方法接收两个参数，第一个参数表示要存储的view的数量，第二个参数表示ListView中第一个可见元素的position值。RecycleBin当中使用mActiveViews这个数组来存储View，调用这个方法后就会根据传入的参数来将ListView中的指定元素存储到mActiveViews数组当中。
+ 
+ **getActiveView()** 这个方法和fillActiveViews()是对应的，用于从mActiveViews数组当中获取数据。该方法接收一个position参数，表示元素在ListView当中的位置，方法内部会自动将position值转换成mActiveViews数组对应的下标值。需要注意的是，mActiveViews当中所存储的View，一旦被获取了之后就会从mActiveViews当中移除，下次获取同样位置的View将会返回null，也就是说mActiveViews不能被重复利用。
+ 
+ addScrapView() 用于将一个废弃的View进行缓存，该方法接收一个View参数，当有某个View确定要废弃掉的时候(比如滚动出了屏幕)，就应该调用这个方法来对View进行缓存，RecycleBin当中使用mScrapViews和mCurrentScrap这两个List来存储废弃View。
+ 
+ getScrapView 用于从废弃缓存中取出一个View，这些废弃缓存中的View是没有顺序可言的，因此getScrapView()方法中的算法也非常简单，就是直接从mCurrentScrap当中获取尾部的一个scrap view进行返回。
+ 
+ setViewTypeCount() 我们都知道Adapter当中可以重写一个getViewTypeCount()来表示ListView中有几种类型的数据项，而setViewTypeCount()方法的作用就是为每种类型的数据项都单独启用一个RecycleBin缓存机制。实际上，getViewTypeCount()方法通常情况下使用的并不是很多，所以我们只要知道RecycleBin当中有这样一个功能就行了。


了解了RecycleBin中的主要方法以及它们的用处之后，下面就可以开始来分析ListView的工作原理了，这里我将还是按照以前分析源码的方式来进行，即跟着主线执行流程来逐步阅读并点到即止，不然的话要是把ListView所有的代码都贴出来，那么本篇文章将会很长很长了。



### 第一次Layout

不管怎么说，ListView即使再特殊最终还是继承自View的，因此它的执行流程还将会按照View的规则来执行，对于这方面不太熟悉的朋友可以参考我之前写的 [Android视图绘制流程完全解析，带你一步步深入了解View(二)](https://blog.csdn.net/guolin_blog/article/details/16330267) 。 

View的执行流程无非就分为三步，

+ onMeasure()用于测量View的大小，
+ onLayout()用于确定View的布局，
+ onDraw()用于将View绘制到界面上。

而在ListView当中，onMeasure()并没有什么特殊的地方，因为它终归是一个View，占用的空间最多并且通常也就是整个屏幕。

onDraw()在ListView当中也没有什么意义，因为ListView本身并不负责绘制，而是由ListView当中的子元素来进行绘制的。

那么ListView大部分的神奇功能其实都是在onLayout()方法中进行的了，因此我们本篇文章也是主要分析的这个方法里的内容。

如果你到ListView源码中去找一找，你会发现ListView中是没有onLayout()这个方法的，这是因为这个方法是在ListView的父类AbsListView中实现的，代码如下所示：

```java
@Override
protected void onLayout(boolean changed, int l, int t, int r, int b) {
	super.onLayout(changed, l, t, r, b);
	mInLayout = true;
	if (changed) {
		int childCount = getChildCount();
		for (int i = 0; i < childCount; i++) {
			getChildAt(i).forceLayout();
		}
		mRecycler.markChildrenDirty();
	}
	layoutChildren();
	mInLayout = false;
}
```

可以看到，onLayout()方法中并没有做什么复杂的逻辑操作，主要就是一个判断，如果ListView的大小或者位置发生了变化，那么changed变量就会变成true，此时会要求所有的子布局都强制进行重绘。除此之外倒没有什么难理解的地方了，不过我们注意到，在第16行调用了layoutChildren()这个方法，从方法名上我们就可以猜出这个方法是用来进行子元素布局的，不过进入到这个方法当中你会发现这是个空方法，没有一行代码。这当然是可以理解的了，因为子元素的布局应该是由具体的实现类来负责完成的，而不是由父类完成。那么进入ListView的layoutChildren()方法，代码如下所示：
```java
@Override
protected void layoutChildren() {
    final boolean blockLayoutRequests = mBlockLayoutRequests;
    if (!blockLayoutRequests) {
        mBlockLayoutRequests = true;
    } else {
        return;
    }
    try {
        super.layoutChildren();
        invalidate();
        if (mAdapter == null) {
            resetList();
            invokeOnItemScrollListener();
            return;
        }
        int childrenTop = mListPadding.top;
        int childrenBottom = getBottom() - getTop() - mListPadding.bottom;
        int childCount = getChildCount();
        int index = 0;
        int delta = 0;
        View sel;
        View oldSel = null;
        View oldFirst = null;
        View newSel = null;
        View focusLayoutRestoreView = null;
        // Remember stuff we will need down below
        switch (mLayoutMode) {
        case LAYOUT_SET_SELECTION:
            index = mNextSelectedPosition - mFirstPosition;
            if (index >= 0 && index < childCount) {
                newSel = getChildAt(index);
            }
            break;
        case LAYOUT_FORCE_TOP:
        case LAYOUT_FORCE_BOTTOM:
        case LAYOUT_SPECIFIC:
        case LAYOUT_SYNC:
            break;
        case LAYOUT_MOVE_SELECTION:
        default:
            // Remember the previously selected view
            index = mSelectedPosition - mFirstPosition;
            if (index >= 0 && index < childCount) {
                oldSel = getChildAt(index);
            }
            // Remember the previous first child
            oldFirst = getChildAt(0);
            if (mNextSelectedPosition >= 0) {
                delta = mNextSelectedPosition - mSelectedPosition;
            }
            // Caution: newSel might be null
            newSel = getChildAt(index + delta);
        }
        boolean dataChanged = mDataChanged;
        if (dataChanged) {
            handleDataChanged();
        }
        // Handle the empty set by removing all views that are visible
        // and calling it a day
        if (mItemCount == 0) {
            resetList();
            invokeOnItemScrollListener();
            return;
        } else if (mItemCount != mAdapter.getCount()) {
            throw new IllegalStateException("The content of the adapter has changed but "
                    + "ListView did not receive a notification. Make sure the content of "
                    + "your adapter is not modified from a background thread, but only "
                    + "from the UI thread. [in ListView(" + getId() + ", " + getClass() 
                    + ") with Adapter(" + mAdapter.getClass() + ")]");
        }
        setSelectedPositionInt(mNextSelectedPosition);
        // Pull all children into the RecycleBin.
        // These views will be reused if possible
        final int firstPosition = mFirstPosition;
        final RecycleBin recycleBin = mRecycler;
        // reset the focus restoration
        View focusLayoutRestoreDirectChild = null;
        // Don't put header or footer views into the Recycler. Those are
        // already cached in mHeaderViews;
        if (dataChanged) {
            for (int i = 0; i < childCount; i++) {
                recycleBin.addScrapView(getChildAt(i));
                if (ViewDebug.TRACE_RECYCLER) {
                    ViewDebug.trace(getChildAt(i),
                            ViewDebug.RecyclerTraceType.MOVE_TO_SCRAP_HEAP, index, i);
                }
            }
        } else {
            recycleBin.fillActiveViews(childCount, firstPosition);
        }
        // take focus back to us temporarily to avoid the eventual
        // call to clear focus when removing the focused child below
        // from messing things up when ViewRoot assigns focus back
        // to someone else
        final View focusedChild = getFocusedChild();
        if (focusedChild != null) {
            // TODO: in some cases focusedChild.getParent() == null
            // we can remember the focused view to restore after relayout if the
            // data hasn't changed, or if the focused position is a header or footer
            if (!dataChanged || isDirectChildHeaderOrFooter(focusedChild)) {
                focusLayoutRestoreDirectChild = focusedChild;
                // remember the specific view that had focus
                focusLayoutRestoreView = findFocus();
                if (focusLayoutRestoreView != null) {
                    // tell it we are going to mess with it
                    focusLayoutRestoreView.onStartTemporaryDetach();
                }
            }
            requestFocus();
        }
        // Clear out old views
        detachAllViewsFromParent();
        switch (mLayoutMode) {
        case LAYOUT_SET_SELECTION:
            if (newSel != null) {
                sel = fillFromSelection(newSel.getTop(), childrenTop, childrenBottom);
            } else {
                sel = fillFromMiddle(childrenTop, childrenBottom);
            }
            break;
        case LAYOUT_SYNC:
            sel = fillSpecific(mSyncPosition, mSpecificTop);
            break;
        case LAYOUT_FORCE_BOTTOM:
            sel = fillUp(mItemCount - 1, childrenBottom);
            adjustViewsUpOrDown();
            break;
        case LAYOUT_FORCE_TOP:
            mFirstPosition = 0;
            sel = fillFromTop(childrenTop);
            adjustViewsUpOrDown();
            break;
        case LAYOUT_SPECIFIC:
            sel = fillSpecific(reconcileSelectedPosition(), mSpecificTop);
            break;
        case LAYOUT_MOVE_SELECTION:
            sel = moveSelection(oldSel, newSel, delta, childrenTop, childrenBottom);
            break;
        default:
            if (childCount == 0) {
                if (!mStackFromBottom) {
                    final int position = lookForSelectablePosition(0, true);
                    setSelectedPositionInt(position);
                    sel = fillFromTop(childrenTop);
                } else {
                    final int position = lookForSelectablePosition(mItemCount - 1, false);
                    setSelectedPositionInt(position);
                    sel = fillUp(mItemCount - 1, childrenBottom);
                }
            } else {
                if (mSelectedPosition >= 0 && mSelectedPosition < mItemCount) {
                    sel = fillSpecific(mSelectedPosition,
                            oldSel == null ? childrenTop : oldSel.getTop());
                } else if (mFirstPosition < mItemCount) {
                    sel = fillSpecific(mFirstPosition,
                            oldFirst == null ? childrenTop : oldFirst.getTop());
                } else {
                    sel = fillSpecific(0, childrenTop);
                }
            }
            break;
        }
        // Flush any cached views that did not get reused above
        recycleBin.scrapActiveViews();
        if (sel != null) {
            // the current selected item should get focus if items
            // are focusable
            if (mItemsCanFocus && hasFocus() && !sel.hasFocus()) {
                final boolean focusWasTaken = (sel == focusLayoutRestoreDirectChild &&
                        focusLayoutRestoreView.requestFocus()) || sel.requestFocus();
                if (!focusWasTaken) {
                    // selected item didn't take focus, fine, but still want
                    // to make sure something else outside of the selected view
                    // has focus
                    final View focused = getFocusedChild();
                    if (focused != null) {
                        focused.clearFocus();
                    }
                    positionSelector(sel);
                } else {
                    sel.setSelected(false);
                    mSelectorRect.setEmpty();
                }
            } else {
                positionSelector(sel);
            }
            mSelectedTop = sel.getTop();
        } else {
            if (mTouchMode > TOUCH_MODE_DOWN && mTouchMode < TOUCH_MODE_SCROLL) {
                View child = getChildAt(mMotionPosition - mFirstPosition);
                if (child != null) positionSelector(child);
            } else {
                mSelectedTop = 0;
                mSelectorRect.setEmpty();
            }
            // even if there is not selected position, we may need to restore
            // focus (i.e. something focusable in touch mode)
            if (hasFocus() && focusLayoutRestoreView != null) {
                focusLayoutRestoreView.requestFocus();
            }
        }
        // tell focus view we are done mucking with it, if it is still in
        // our view hierarchy.
        if (focusLayoutRestoreView != null
                && focusLayoutRestoreView.getWindowToken() != null) {
            focusLayoutRestoreView.onFinishTemporaryDetach();
        }
        mLayoutMode = LAYOUT_NORMAL;
        mDataChanged = false;
        mNeedSync = false;
        setNextSelectedPositionInt(mSelectedPosition);
        updateScrollIndicators();
        if (mItemCount > 0) {
            checkSelectionChanged();
        }
        invokeOnItemScrollListener();
    } finally {
        if (!blockLayoutRequests) {
            mBlockLayoutRequests = false;
        }
    }
} 

```

 

这段代码比较长，我们挑重点的看。首先可以确定的是，ListView当中目前还没有任何子View，数据都还是由Adapter管理的，并没有展示到界面上，因此第19行getChildCount()方法得到的值肯定是0。接着在第81行会根据dataChanged这个布尔型的值来判断执行逻辑，dataChanged只有在数据源发生改变的情况下才会变成true，其它情况都是false，因此这里会进入到第90行的执行逻辑，调用RecycleBin的fillActiveViews()方法。按理来说，调用fillActiveViews()方法是为了将ListView的子View进行缓存的，可是目前ListView中还没有任何的子View，因此这一行暂时还起不了任何作用

接下来在第114行会根据mLayoutMode的值来决定布局模式，默认情况下都是普通模式LAYOUT_NORMAL，因此会进入到第140行的default语句当中。而下面又会紧接着进行两次if判断，childCount目前是等于0的，并且默认的布局顺序是从上往下，因此会进入到第145行的fillFromTop()方法，我们跟进去瞧一瞧：
```java
private View fillFromTop(int nextTop) {
    mFirstPosition = Math.min(mFirstPosition, mSelectedPosition);
    mFirstPosition = Math.min(mFirstPosition, mItemCount - 1);
    if (mFirstPosition < 0) {
        mFirstPosition = 0;
    }
    return fillDown(mFirstPosition, nextTop);
} 
```

