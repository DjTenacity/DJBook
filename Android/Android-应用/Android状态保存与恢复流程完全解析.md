## [Android状态保存与恢复流程完全解析](https://mp.weixin.qq.com/s?__biz=MzA3MjgwNDIzNQ==&mid=2651943472&idx=1&sn=fb927e34db29b98b07a728d71a098f3d&chksm=84fd60d4b38ae9c2850e8fde15bd3f43fb41f5c890a19f22d0179ae93bfecb6e20fa1908ff54&scene=21#wechat_redirect)

​		对于一个Activity或者View来说，状态的保存与恢复是必不可少的，最常见的一种情况是切换屏幕方向了，如果由竖屏切换为横屏，那么必定会经历Activity的摧毁与重建，那么它所对应的View视图也会被摧毁和重建，如果此时没有对View进行状态的保存的话，那么待View重建后，其之前的状态便不复存在。

​		庆幸的是，对于系统自带的View（比如CheckBox等），Android已经帮我们实现了状态的自动保存与恢复，不必我们费心了。但是对于我们自己开发的自定义View，就需要我们去保存状态和恢复状态了，系统提供了两个API方便我们去实现保存和恢复，分别是**onSaveInstanceState**和**onRestoreInstanceState**这两个方法，我们只需要重写这两个方法，然后在里面写需要的逻辑即可。那么接下来就分析整个保存和恢复的流程是怎样的。

## 保存状态流程分析

#### 从Activity的onSaveInstanceState说起

Activity有这样一个方法：**onSaveInstanceState (Bundle outState)**，相信大家都不陌生，我们通过重写这个方法来保存我们需要的数据。先来看看官方文档对这个方法的注释：

> Called to retrieve per-instance state from an activity before being killed so that the state can be restored in onCreate(Bundle) or onRestoreInstanceState(Bundle) (the Bundle populated by this method will be passed to both).
> This method is called before an activity may be killed so that when it comes back some time in the future it can restore its state

可以知道，当一个Activity变得容易被kill的时候会调用这个方法，比如说当旋转屏幕的时候，当前Activity在onDestroy之前会调用onSaveInstanceState来保存状态，又或者说当前Activity被置于后台了，系统内存紧张的时候会有可能杀死这个Activity，那么此时Activity也会保存状态。
那么我们来看看这个方法内部做了些什么，**Activity#onSaveInstanceState**:

```java
protected void onSaveInstanceState(Bundle outState) {
    //1、保存View树状态,key是WINDOW_HIERARCHY_TAG
    outState.putBundle(WINDOW_HIERARCHY_TAG, mWindow.saveHierarchyState());
    //2、保存Fragment状态,key是FRAGMENTS_TAG
    Parcelable p = mFragments.saveAllState();
    if (p != null) {
        outState.putParcelable(FRAGMENTS_TAG, p);
    }
    getApplication().dispatchActivitySaveInstanceState(this, outState);
}
```

显然，数据是存放在一个Bundle里面的。首先，调用了mWindow的saveHierarchyState()方法，遍历Activity的View树来保存数据，接着调用mFragments.saveAllState()对与Activity所关联的Fragment保存数据。下面，我们先来看看怎样保存View树的数据的：

#### 1、保存View树数据

从上面可以看出，调用了Window的saveHierarchyState方法，而Window是一个抽象类，它的实现类是PhoneWindow，那么我们来看**PhoneWindow#saveHierarchyState**:

```
@Override
public Bundle saveHierarchyState() {
    Bundle outState = new Bundle();
    if (mContentParent == null) {
        return outState;
    }
    SparseArray<Parcelable> states = new SparseArray<Parcelable>();
    mContentParent.saveHierarchyState(states);
    outState.putSparseParcelableArray(VIEWS_TAG, states);
    //省略...

    return outState;
}
```

这里先是实例化了一个SparseArray，它和HashMap类似，保存着键值对，接着调用了mContentParent的saveHierarchyState()方法，并把结果放进outState中并返回。这里的mContentParent是DecorView的子元素或者其自身，我之前的文章也有所提及，这里可以把mContentParent看做整个View树的顶层视图，由于mContentParent是一个ViewGroup，但是ViewGroup没有重写saveHierarchyState方法，那么这里调用的便是**View#saveHierarchyState**:

```
public void saveHierarchyState(SparseArray<Parcelable> container) {
     dispatchSaveInstanceState(container);
}
```

从名字可以猜测，接下来就是分发保存事件了，遍历所有子View，让所有的子View去保存数据，我们来看看**ViewGroup#dispatchSaveInstanceState**是否如我们所想的那样：

```java
https://blog.csdn.net/qq_35181209/article/details/66544974@Override
protected void dispatchSaveInstanceState(SparseArray<Parcelable> container) {
    //调用View的dispatchSaveInstanceState方法，目的是保存ViewGroup自身的状态
    super.dispatchSaveInstanceState(container);
    final int count = mChildrenCount;
    final View[] children = mChildren;
    //遍历所有的子View，保存状态，所有状态都放在SparseArray内
    for (int i = 0; i < count; i++) {
        View c = children[i];
        if ((c.mViewFlags &amp; PARENT_SAVE_DISABLED_MASK) != PARENT_SAVE_DISABLED) {
            c.dispatchSaveInstanceState(container);
        }
    }
}
```

从上面源码可以看出，ViewGroup除了把保存事件分发给子View，还会让自身先保存好状态，接着我们看**View#dispatchSaveInstanceState**源码：

```java
protected void dispatchSaveInstanceState(SparseArray<Parcelable> container) {
    if (mID != NO_ID &amp;&amp; (mViewFlags &amp; SAVE_DISABLED_MASK) == 0) {
        mPrivateFlags &amp;= ~PFLAG_SAVE_STATE_CALLED;
        Parcelable state = onSaveInstanceState();

        if (state != null) {
            container.put(mID, state);
        }
    }
}
```

这里首先判断当前View是不是有一个ID以及当前View的标志位的状态等，接着下面便调用到了**View#onSaveInstanceState()**方法，也即我们的自定义View需要重写的方法，这个方法返回Parcelable对象，即可序列化对象，最后把该Parcelable对象放进了SparseArray内，key是该View的id。
**由此可知，如果一个View需要保存状态，那么至少需要以下两个条件**：
**①**该View有着唯一的一个id。可通过setId或xml设置id，如果id不唯一，由于SparseArray是以id为key保存状态的，那么相同的id的View的数据会覆盖。
**②**该View的标志位不是SAVE_DISABLED_MASK。可通过setSaveEnabled(boolean)方法来改变，默认是true，即可保存状态。

那么，到此Activity的View树保存状态流程分析完毕，如果Activity关联着Fragment，那么也会对Fragment进行状态保存，即上面提到的2号代码。

#### 2、保存Fragment数据

上面2号代码调用了mFragment的saveAllState()方法，而这里的mFragment是一个FragmentController，最后调用到了**FragmentManagerImpl#saveAllState**：

```
Parcelable saveAllState() {

    // First collect all active fragments.
    int N = mActive.size();
    FragmentState[] active = new FragmentState[N];
    boolean haveFragments = false;
    for (int i=0; i<N; i++) {
        Fragment f = mActive.get(i);
        if (f != null) {

            haveFragments = true;

            FragmentState fs = new FragmentState(f);
            active[i] = fs;

            if (f.mState > Fragment.INITIALIZING &amp;&amp; fs.mSavedFragmentState == null) {
                fs.mSavedFragmentState = saveFragmentBasicState(f);  // 3
            //省略..
            }        
        }
    }

    //省略...

    FragmentManagerState fms = new FragmentManagerState();
    fms.mActive = active;
    fms.mAdded = added;
    fms.mBackStack = backStack;
    return fms;
}
```

以上是精简后的源码，首先对该Activity内的活跃的Fragment进行遍历（所谓活跃的Fragment，笔者这里理解为未被摧毁的，还有着视图结构的Fragment，如有错误请指正），接着对每一个活跃的Fragment都新建一个FragmentState，顾名思义，这个FragmentState对应着Fragment的状态，我们来简单看看这个**FragmentState**:

```java
final class FragmentState implements Parcelable {
    final String mClassName;
    final int mIndex;
    //若干个属性...

    Bundle mSavedFragmentState;

    Fragment mInstance;

    public FragmentState(Fragment frag) {
        mClassName = frag.getClass().getName();
        mIndex = frag.mIndex;
        //若干个属性...
    }
}
```

在新建一个FragmentState的时候，就已经把Fragment的几个重要属性赋值给FragmentState的属性了，而FragmentState还有一个属性，那就是mSavedFragmentState，这个属性是在上面的③号代码处得到赋值的，这里调用了**saveFragmentBasicState(Fragment)**方法，我们来看看这个方法：

```java
Bundle saveFragmentBasicState(Fragment f) {
    Bundle result = null;

    if (mStateBundle == null) {
        mStateBundle = new Bundle();
    }
    //这里会调用到Fragment的onSaveInstanceState方法
    f.performSaveInstanceState(mStateBundle);
    if (!mStateBundle.isEmpty()) {
        result = mStateBundle;
        mStateBundle = null;
    }

    if (f.mView != null) {
        //保存Fragment的视图结构数据
        saveFragmentViewState(f);
    }
    if (f.mSavedViewState != null) {
        if (result == null) {
            result = new Bundle();
        }
        //将上面保存的视图结构数据放进Bundle内
        result.putSparseParcelableArray(
                FragmentManagerImpl.VIEW_STATE_TAG, f.mSavedViewState);
    }

    return result;
}
```

上面调用了Fragment#performSaveInstanceState方法，里面进而调用了**Fragment#onSaveInstanceState**方法，这个是个空实现，我们可以在我们的Fragment重写该方法，保存我们需要的数据，此时数据都会保存在Bundle内。接着，调用了**saveFragmentViewState方法**，我们来看看这个方法：

```java
void saveFragmentViewState(Fragment f) {
    if (f.mView == null) {
        return;
    }
    if (mStateArray == null) {
        mStateArray = new SparseArray<Parcelable>();
    } else {
        mStateArray.clear();
    }
    f.mView.saveHierarchyState(mStateArray);
    if (mStateArray.size() > 0) {
        f.mSavedViewState = mStateArray;
        mStateArray = null;
    }
}
```

可以看出，里面调用了f.mView.saveHierarchyState(mStateArray)，那么这就跟上面所说的View树保存的套路一样了。可以看出，当调用了saveFragmentBasicState(Fragment)这个方法时，其返回的**Bundle对象**已经携带了两种数据：其一，我们重写的onSaveInstanceState(Bundle)方法所保存的数据；其二，Fragment的View树的视图数据。而这个Bundle对象则是存放在FragmenState内的mSavedFragmentState属性中。**也即是说，在saveAllState()方法内，对于每一个Fragment所生成的FragmentState，不但保存着Fragemnt的基础数据，也保存着Bundle对象。**所以说，FragmentState有着举足轻重的作用。
至此，保存状态的源码分析完毕。各位看官可以站起来放松活动一下，接着就到我们的恢复状态的流程分析了。

## 恢复状态流程分析

#### 从Activity的onRestoreInstanceState说起

通过保存状态的流程分析，我们知道所有的状态都保存在了一个Bundle对象内，那么要恢复状态就必须要拿到这个Bundle对象才行，在Activity的生命周期内，onCreate(Bundle)携带了一个Bundle参数，这正是我们之前保存状态的Bundle，因此我们可以在onCreate(Bundle)方法内进行恢复数据的操作。此外，还有一个方法，与onSaveInstanceState(Bundle)相对的，onRestoreInstanceState(Bundle)方法用于恢复状态。我们看看官方文档对**onRestoreInstanceState(Bundle)**方法的解释：

> This method is called after onStart() when the activity is being re-initialized from a previously saved state, given here in savedInstanceState. Most implementations will simply use onCreate(Bundle) to restore their state, but it is sometimes convenient to do it here after all of the initialization has been done…

从官方文档的注释，我们知道，onRestoreInstanceState(Bundle)会在onStart()方法后面执行，用于重建视图的时候恢复数据，其实调用onCreate(Bundle)方法的时候已经进行了一部分的恢复，此时主要是恢复Fragment的状态，由于onCreate方法内部一般是用于控件的初始化，如果此时就进行View树的数据恢复，就会出错(未初始化完毕)。

#### 1、恢复View树状态

我们来看看**Activity#onRestoreInstanceState**方法：

```java
protected void onRestoreInstanceState(Bundle savedInstanceState) {
    if (mWindow != null) {
        Bundle windowState = savedInstanceState.getBundle(WINDOW_HIERARCHY_TAG);
        if (windowState != null) {
            mWindow.restoreHierarchyState(windowState);
        }
    }
}
```

先是根据WINDOW_HIERARCHY_TAG这个key获取Bundle对应的数据，即View树数据，接着调用mWindow.restoreHierarchyState方法，我们继续看**PhoneWindow#restoreHierarchyState**：

```java
@Override
public void restoreHierarchyState(Bundle savedInstanceState) {

    SparseArray<Parcelable> savedStates
            = savedInstanceState.getSparseParcelableArray(VIEWS_TAG);
    if (savedStates != null) {
        mContentParent.restoreHierarchyState(savedStates);
    }
    //省略...
}
```

从Bundle里面根据VIEWS_TAG来获取SparseArray，这个之前我们也说过了，这就是View树数据所对应的SparseArray，接着调用mContentParent.restoreHierarchyState，到这里我们也知道接下来应该是调用**View#restoreHierarchyState**方法，而就如保存状态一样，恢复状态也需要把事件分发给ViewGroup的所有子View，所以在restoreHierarchyState方法里面又调用到了**ViewGroup#dispatchRestoreInstanceState**:

```java
@Override
protected void dispatchRestoreInstanceState(SparseArray<Parcelable> container) {
    //调用View的dispatchRestoreInstanceState，目的是恢复ViewGroup自身的状态
    super.dispatchRestoreInstanceState(container);
    final int count = mChildrenCount;
    final View[] children = mChildren;
    //遍历所有子View，逐个恢复它们的状态
    for (int i = 0; i < count; i++) {
        View c = children[i];
        if ((c.mViewFlags &amp; PARENT_SAVE_DISABLED_MASK) != PARENT_SAVE_DISABLED) {
            c.dispatchRestoreInstanceState(container);
        }
    }
}
```

可以看出，代码和dispatchOnSaveInstanceState方法基本类似，接着我们看**View#dispatchRestoreInstanceState**:

```java
protected void dispatchRestoreInstanceState(SparseArray<Parcelable> container) {
    if (mID != NO_ID) {
        Parcelable state = container.get(mID);
        if (state != null) {
            mPrivateFlags &amp;= ~PFLAG_SAVE_STATE_CALLED;
            onRestoreInstanceState(state);
        }
    }
}
```

这里根据View的Id在SparseArray中获得对应的Parcelable对象，即视图数据，接着调用了View#onRestoreInstanceState(Parcelable)方法，交给每一个View来自行恢复数据，一般对于自定义View来说，我们会重写onRestoreInstanceState(Parcelable)方法，来处理我们需要恢复的数据。至此，View树的数据恢复解析完毕，我们接着看Fragment的数据恢复。

#### 2、恢复Fragment状态

让我们回到**Activity#onCreate(Bundle)方法**，看看源码：

```java
protected void onCreate(@Nullable Bundle savedInstanceState) {
    //省略部分源码..

    if (savedInstanceState != null) {
        Parcelable p = savedInstanceState.getParcelable(FRAGMENTS_TAG);
        mFragments.restoreAllState(p, mLastNonConfigurationInstances != null
                ? mLastNonConfigurationInstances.fragments : null);
    }
    mFragments.dispatchCreate();
    //...
}
```

这里先根据FRAGMENTS_TAG这个key获取对应的Parcelable对象，接着调用mFragments.restoreAllState方法，即**FragmentManagerImpl#restoreAllState**方法：

```java
void restoreAllState(Parcelable state, List<Fragment> nonConfig) {
    // If there is no saved state at all, then there can not be
    // any nonConfig fragments either, so that is that.
    if (state == null) return;
    FragmentManagerState fms = (FragmentManagerState)state;
    if (fms.mActive == null) return;


    // Build the full list of active fragments, instantiating them from
    // their saved state.
    mActive = new ArrayList<Fragment>(fms.mActive.length);
    if (mAvailIndices != null) {
        mAvailIndices.clear();
    }
    for (int i=0; i<fms.mActive.length; i++) {
        FragmentState fs = fms.mActive[i];
        if (fs != null) {
            Fragment f = fs.instantiate(mHost, mParent);  // 1
            if (DEBUG) Log.v(TAG, "restoreAllState: active #" + i + ": " + f);
            mActive.add(f);
            // Now that the fragment is instantiated (or came from being
            // retained above), clear mInstance in case we end up re-restoring
            // from this FragmentState again.
            fs.mInstance = null;
        } 
        //...
        }
    }

    //省略部分代码...
    //...
}
```

关注上面的①号代码，这里调用了FragmentState的instantiate方法，上面说有FragmentState保存着一个Fragment的状态，那恢复状态也应该是由FragmentState来恢复，我们看**FragmentState#instantiate**:

```java
public Fragment instantiate(FragmentHostCallback host, Fragment parent) {
    //...
    mInstance = Fragment.instantiate(context, mClassName, mArguments);

    if (mSavedFragmentState != null) {
        mSavedFragmentState.setClassLoader(context.getClassLoader());
        mInstance.mSavedFragmentState = mSavedFragmentState;
    }
    mInstance.setIndex(mIndex, parent);
    mInstance.mFromLayout = mFromLayout;
    mInstance.mRestored = true;
    mInstance.mFragmentId = mFragmentId;
    mInstance.mContainerId = mContainerId;
    mInstance.mTag = mTag;
    mInstance.mRetainInstance = mRetainInstance;
    mInstance.mDetached = mDetached;
    mInstance.mFragmentManager = host.mFragmentManager;
    if (FragmentManagerImpl.DEBUG) Log.v(FragmentManagerImpl.TAG,
            "Instantiated fragment " + mInstance);

    return mInstance;
}
```

这里先新建了Fragment，接着把FragmentState保存的状态赋值给Fragment，即新的Fragment“获得”了之前的Fragment的状态。但是这里的Fragment只是恢复的部分的基础数据，另外的一些数据并不是在这里恢复的，这里只是创建了一个新的Fragment，是空的，View视图还没有创建，所以别的数据是在Fragment进入生命周期后再恢复的。让我们先回到**Activity#onCreate**方法，在调用mFragments.restoreAllState方法完毕后，接着调用了mFragments.dispatchCreate()方法，这就是使Fragment开始了它的生命周期，我们看**FragmentManagerImpl#dispatchCreate**:

```java
public void dispatchCreate() {
     mStateSaved = false;
     moveToState(Fragment.CREATED, false);
}
```

这里调用了moveToState()方法，这个方法是贯穿了Fragment生命周期的所有方法，我们来看看moveToState()方法中有关状态恢复部分的源码：

```java
void moveToState(Fragment f, int newState, int transit, int transitionStyle,
            boolean keepActive) {
    //省略...
    //...

    if (f.mState < newState) {
        switch (f.mState) {
            case Fragment.INITIALIZING:
                //省略...

            case Fragment.CREATED:
                if (newState > Fragment.CREATED) {
                    //省略...
                    f.performActivityCreated(f.mSavedFragmentState);
                    if (f.mView != null) {
                        f.restoreViewState(f.mSavedFragmentState);
                    }
                    f.mSavedFragmentState = null;
                }
            case Fragment.ACTIVITY_CREATED:
            case Fragment.STOPPED:
                //...
            case Fragment.STARTED:

        }   
    }     
    f.mState = newState;
}
```

看代码中间，调用了performActivityCreated()方法，这里对应着Fragment生命周期中的onActivityCreated()方法，随后调用了restoreViewState()方法，想必这里便是状态恢复的部分了，同时我们得到一个信息，Fragment的onRestoreViewState()方法是在onActivityCreated()方法之后调用的，当然，由于onActivityCreated也携带Bundle对象，我们也可以在这个方法内恢复状态。我们来看看**Fragment#restoreViewState**:

```java
final void restoreViewState(Bundle savedInstanceState) {
    if (mSavedViewState != null) {
        mView.restoreHierarchyState(mSavedViewState);
        mSavedViewState = null;
    }
    mCalled = false;
    onViewStateRestored(savedInstanceState);
    //...
}
```

看到这里，大家也应该很熟悉了，跟Activity的恢复状态很类似，先是调用mView.restoreHierarchyState，进行Fragment的View树状态恢复，接着调用onViewStateRestored(Bundle)方法，恢复我们需要的数据，这个方法一般是由我们来重写。至此，恢复状态流程分析完毕。

## 总结

上面详细地分析了状态保存与恢复的流程，最后我们总结一下整个保存-恢复流程。首先，在Activity中调用onSaveInstanceState()方法，保存了Activity的View树状态、与Activity关联的Fragment的状态以及我们自己重写的onSaveInstanceState()方法所保存的数据。而Fragment的状态又包括了：Fragment的View树状态、Fragment的基础属性以及我们自己重写的onSaveInstanceState()方法所保存的数据。所保存的所有状态数据均放在Bundle内，这个Bundle可以在Activity的onCreate(Bundle)方法或者onRestoreInstanceState(Bundle)方法内获取。在Activity#onCreate方法中，会对Fragment进行状态恢复，接着Fragment就会进入正常的生命周期，进而对自身的View树进行状态恢复，或者调用我们重写的onRestoreViewState方法来恢复我们需要的数据。而在Activity#onRestoreInstanceState方法内则是对Activity的View树进行状态恢复以及恢复我们需要的数据。

 

**大家都在看**



[2018 年 8 月面试路：6 天 21 家公司](http://mp.weixin.qq.com/s?__biz=MzA3MjgwNDIzNQ==&mid=2651943468&idx=1&sn=9860898a569423e2832b0632e49247b3&chksm=84fd60c8b38ae9de4fc12bb2fb08c593c5cd226b70806e6bee201b677a70cecf8a6e52473a33&scene=21#wechat_redirect)

[众多Android开源控件库收藏分享](http://mp.weixin.qq.com/s?__biz=MzA3MjgwNDIzNQ==&mid=2651943460&idx=1&sn=218ee381f94eab4c90a259dc5c6b755a&chksm=84fd60c0b38ae9d6bb38cfc71ef2b873ea6d97080166aa3e7f38b92c3d3b4db311d306084395&scene=21#wechat_redirect)

[Android插件化原理之Activity插件化](http://mp.weixin.qq.com/s?__biz=MzA3MjgwNDIzNQ==&mid=2651943433&idx=1&sn=539126aa652b9287c7d4a8856d6f88fc&chksm=84fd60edb38ae9fb6f3c6255b4e40fedf0458124c572454b392b13f7252aa6ab6dbc6f23dcc0&scene=21#wechat_redirect)

[Android采用AOP方式封装权限管理](http://mp.weixin.qq.com/s?__biz=MzA3MjgwNDIzNQ==&mid=2651943450&idx=1&sn=215ecf35a0ddee1401623e944c6d5840&chksm=84fd60feb38ae9e82336d87deaed4dcd2f659f9e570b8b5f8e0631c6737d4838cfcd6597f27b&scene=21#wechat_redirect)