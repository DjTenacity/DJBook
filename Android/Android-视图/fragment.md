# fragment 碎片

onViewCreated在onCreateView执行完后立即执行。
onCreateView返回的就是fragment要显示的view。



　　　活动停止时，与它关联的碎片就会进入到停止状态，或者通过调用FragmentTransaction的remove()，replace()方法将碎片从活动中移除，但如果在事务提交之前调用addToBackStack()方法，这时的碎片也会进入到停止状态。总的来说，进入停止状态的碎片对用户来说是完全不可见，有可能会被系统回收。 

4销毁：

​           碎片总是依附于活动存在的，因此当活动被销毁时，与它相关联的碎片就会进入到销毁状态。或者调用FragmentTransaction的remove(),replace()方法将碎片从活动中移除，但在事务提交之前并没有调用addTobackStack()方法，这时的碎片也会进入到销毁状态



Fragment  如果是viewpager中且页数多于,生命周期会再走一遍 ,注意全局变量的更改,需要在生命周期中重置数据!!!

```java
@Nullable
@Override
public View onCreateView(LayoutInflater inflater, @Nullable ViewGroup container, @Nullable Bundle savedInstanceState) {
    View contentView = inflater.inflate(getViewLayout(), container, false);
    ButterKnife.bind(this, contentView);
    return contentView;
}

@Override
public void onViewCreated(View view, @Nullable Bundle savedInstanceState) {
    super.onViewCreated(view, savedInstanceState);
    initView(savedInstanceState);
    loadData(savedInstanceState);
}
```

