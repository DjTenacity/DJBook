# 文本控件

### Android中TextView划线

- textView.getPaint().setFlags(Paint. UNDERLINE_TEXT_FLAG ); //下划线

- textView.getPaint().setAntiAlias(true);//抗锯齿

- textview.getPaint().setFlags(Paint. STRIKE_THRU_TEXT_FLAG); //中划线

- textview.setFlags(Paint. STRIKE_THRU_TEXT_FLAG|Paint.ANTI_ALIAS_FLAG);  // 设置中划线并加清晰 

- textView.getPaint().setFlags(0);  // 取消设置的的划线

```java
tvLogin.setText(Html.fromHtml("<u>"+"立即登录"+"</u>"));//下划线 

<resources>
 <string name="hello"><u>phone:0123456</u></string>
    <string name="app_name">MyLink</string>
</resources>
```

### Android中TextView实现分段显示不同颜色的字符串

**一般有三种实现方式**

- 直接根据不同的需要分段字符串，然后分别使用多个TextView来显示
- 使用spannablestring
- 使用Html

**下面分别来简单介绍下三种方法**

**多个TextVew**

- 这种方式简单粗暴，颜色样式控制灵活
- 如果需要显示的文本需要分多个段的话，那就需要很多个TextView，而且布局不好控制
- 实现方式简单，就不写例子了

**使用SpannableString**

想必用过的人都知道，比较好的一点是SpannableString可以精确控制一个长长的字符串中第几个到第几个字符的样式

```java
SpannableString spannableString = new SpannableString("jakjfkajfjaj");
//设置颜色
spannableString.setSpan(new ForegroundColorSpan(Color.parseColor("#FE6026")), 3, 6, Spanned.SPAN_EXCLUSIVE_EXCLUSIVE);
//设置字体大小，true表示前面的字体大小20单位为dip
spannableString.setSpan(new AbsoluteSizeSpan(20, true), 0, 5, Spanned.SPAN_EXCLUSIVE_EXCLUSIVE);
//设置链接
spannableString.setSpan(new URLSpan("www.baidu.com"), 0, 5, Spanned.SPAN_EXCLUSIVE_EXCLUSIVE);
//设置字体，BOLD为粗体
spannableString.setSpan(new StyleSpan(android.graphics.Typeface.BOLD), 0, 5, Spanned.SPAN_EXCLUSIVE_EXCLUSIVE);
```

以上只是列举了几个常见的用法，更多的可以参考android.text.style包下面的几个类

![](C:\Users\admin\Desktop\DJGitBook\img\android\android.text.style.jpg)

SpannableString的优点在于控制得精细，缺点也是在这。我们使用SpannableString的时候必须指定样式使用的字符下标，那如果我们的字符串不是固定长度的呢？

**使用Html**

如果使用场景是字符串长度不固定的，比如需要服务端的数据来填充的

"首付："+ data1 + "月供："+ data2

服务端返回的数据长度不固定的话，用SpannableString就尴尬了，这时候可以考虑用Html

Html使用格式比较简单，需要用到font标签，具体的话看下面的例子

```java
//首先是拼接字符串
String content = "<font color=\"#FE6026\">" + data + "</font>"
//然后直接setText()
TextView tvContent = (TextView) view.findViewById(R.id.tvContent);
tvContent.setText(Html.fromHtml(content));
```

### EditText maxLines  显示密码  光标位置

- ```
  //显示密码
  etInputPassword.setInputType(InputType.TYPE_TEXT_VARIATION_VISIBLE_PASSWORD);
  //光标在结尾处     etInputPassword.setSelection(etInputPassword.getText().length());
  //密码隐藏
  etInputPassword.setInputType(InputType.TYPE_CLASS_TEXT|InputType.TYPE_TEXT_VARIATION_PASSWORD);
  
  //CheckBox
  	android:button="@null"
      android:drawableRight="@drawable/selector_registry_cb"
  ```

- maxLines对应着EditText 的最大高度 , 控制的是外部边框,而不是内部文本的行数

- 首先设置只能输入数字：android:digits="1234567890."

- 再者代码里面设置输入法类型

```
editText.setInputType(EditorInfo.TYPE_CLASS_PHONE); 
editText.setInputType(EditorInfo.TYPE_CLASS_PHONE);
```

- 则如果该EditText获得焦点，会弹出数字输入法的模拟键盘

 ```
  android:focusable="false"
  android:inputType="none" 
 ```

```
我的解决方法大概是：让EditText所在的layout获得焦点
`etSearchStoreHome.setFocusable(false);`给layout注册OnTouchListener监听器
直接使用  .requestFocus()   无法获取焦点，焦点依然在EditTtext上
先调用下面这两个方法：
.setFocusable(true);
.setFocusableInTouchMode(true);
再调用  .requestFocus() 就可获取焦点

relative.setOnTouchListener(new OnTouchListener() {
    public boolean onTouch(View v, MotionEvent event) {
          relative.setFocusable(true);
          relative.setFocusableInTouchMode(true);
          relative.requestFocus();
          return false;
        }
     });
```

 ```
- 需要注意的是 setOnEditorActionListener这个方法，并不是在我们点击EditText的时候触发，也不是在我们对EditText进行编辑时触发，而是在我们编辑完之后点击软键盘上的回车键才会触发。**﻿﻿

### 关于如何将ListView绑定到AlertDialog上

可以用以下几种方法实现：

​       一、 AlertDialog.Builder.setItems(CharSequence[] items, OnClickListenerlistener) . items就是你想要添加到AlertDialog的一个list，listener是为list设置的监听器，你只需在里面添加自己想要的动作即可。该方法比较简单。

​       二、[AlertDialog.Builder](http://developer.android.com/reference/android/app/AlertDialog.Builder.html).setAdapter ([ListAdapter](http://developer.android.com/reference/android/widget/ListAdapter.html) adapter,[DialogInterface.OnClickListener](http://developer.android.com/reference/android/content/DialogInterface.OnClickListener.html) listener)。这里你需要先定义一个ListAdapter，Adapter可以说是将数据绑定到UI的桥梁，功能很强大，listener与第一种方法里的一样。而且对于我这种菜鸟来说掌握它的用法与比较难。

​       关于如何自定义AlertDialog样式：

​       一、AlertDialog.Builder.setView(View view)。这个view是事先从定义好的XML文件里获取的，关于如何获取可以用下面代码实现：

​```java
LayoutInflater inflater = LayoutInflater.from(Context context);

View view = inflater.inflate(R.layout.alertdialog, null);//这里的R.layout.alertdialog即为你自定义的布局文件 
 ```

二、view.Window.setContentView(View view)。主要代码如下：

```
AlertDialog mAlertDialog = builder.create();

mAlertDialog.show();

mAlertDialog.getWindow().setContentView(view);
```

​      关于两者的区别，大家可以看这个链接：

http://www.x2x1.com/show/6040883.aspx

  大致就是setView()只会覆盖AlertDialog的Title与Button之间的那部分，而setContentView()则会覆盖全部。但是否真的是这样呢？还有待验证。

​      还有一点要注意的是：setContentView()必须放在show()的后面，不然会报错。如果你要在代码里修改AlertDialog的大小，可以用以下代码实现：

```
mAlertDialog.getWindow().setLayout(150, 320);
```

​      但是这段代码同样需要放在

show()的后面，不然你的改动会没有效果。

​      如果可以成功地完成上述步骤，你差不多就可以自定义一个自己想要的AlertDialog了。但遗憾的是，我始终不知道怎么自定义AlertDialog的Title。最后我在<http://blog.csdn.net/chuekup/article/details/8018513>找到了一些思路：

​     1. 先自定义一个AlertDialog布局alertdialog.xml，包括两部分：一个textView 用来显示Title；一个ListView显示相关的选项:

### android获取屏幕的高度和宽度

```
android获取屏幕的高度和宽度用到WindowManager这个类，两种方法： 
 
1、WindowManager wm = (WindowManager) getContext()
                    .getSystemService(Context.WINDOW_SERVICE);
 
     int width = wm.getDefaultDisplay().getWidth();
     int height = wm.getDefaultDisplay().getHeight();
 
2、WindowManager wm = this.getWindowManager();
 
     int width = wm.getDefaultDisplay().getWidth();
     int height = wm.getDefaultDisplay().getHeight();
```

### 修改project所在路径的文件夹

1. 关闭Android Studio 
2. 修改project所在路径的文件夹名字为[NewName]
3. 打开Android Stuido，import新的[NewName]路径工程(很重要,重新import工程，Android Studio会自动修改部分相关的project名字引用)
4. 修改根目录下的.iml文件名为[NewName].iml，及该文件中的external.linked.project.id=[NewName]
5. 修改.idea/modules.xml里面的
<module fileurl="file://$PROJECT_DIR$/[NewName].iml" filepath="$PROJECT_DIR$/[NewName].iml" />