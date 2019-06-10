## 减少代码中该死的 if else 嵌套

程序员的脑力不应该花费在无止境的分支语句里的，应该专注于业务本身。所以我们很有必要避免写出多分支嵌套的语句。好的，我们来分析下上面的代码多分支的原因：

- 空值判断
- 业务判断
- 状态判断

几乎所有的业务都离不开这几个判断，从而导致`if else`嵌套过多。那是不是没办法解决了？答案肯定不是的。

上面的代码我是用java写的，对于java程序员来说，空值判断简直使人很沮丧，让人身心疲惫。上面的代码每次回调都要判断一次`listener`是否为空，又要判断用户传入的`ShareItem`是否为空，还要判断`ShareItem`里面的字段是否为空……

对于这种情况，我采用的方法很简单：接口分层。

### 减少 if else 方法一：接口分层

**所谓接口分层指的是：把接口分为外部和内部接口，所有空值判断放在外部接口完成，只处理一次；而内部接口传入的变量由外部接口保证不为空，从而减少空值判断。**

来，看代码更加直观：

```java
public void share(ShareItem item, ShareListener listener) {
    if (item == null) {
        if (listener != null) {
            listener.onCallback(ShareListener.STATE_FAIL, "ShareItem 不能为 null");
        }
        return;
    }

    if (listener == null) {
        listener = new ShareListener() {
            @Override
            public void onCallback(int state, String msg) {
                Log.i("DEBUG", "ShareListener is null");
            }
        };
    }

    shareImpl(item, listener);
}

private void shareImpl (ShareItem item, ShareListener listener) {
    if (item.type == TYPE_LINK) {
        // 分享链接
        if (!TextUtils.isEmpty(item.link) &amp;&amp; !TextUtils.isEmpty(item.title)) {
            doShareLink(item.link, item.title, item.content, listener);
        } else {
            listener.onCallback(ShareListener.STATE_FAIL, "分享信息不完整");
        }
    } else if (item.type == TYPE_IMAGE) {
        // 分享图片
        if (!TextUtils.isEmpty(item.imagePath)) {
            doShareImage(item.imagePath, listener);
        } else {
            listener.onCallback(ShareListener.STATE_FAIL, "分享信息不完整");
        }
    } else if (item.type == TYPE_TEXT) {
        // 分享文本
        if (!TextUtils.isEmpty(item.content)) {
            doShareText(item.content, listener);
        } else {
            listener.onCallback(ShareListener.STATE_FAIL, "分享信息不完整");
        }
    } else if (item.type == TYPE_IMAGE_TEXT) {
        // 分享图文
        if (!TextUtils.isEmpty(item.imagePath) &amp;&amp; !TextUtils.isEmpty(item.content)) {
            doShareImageAndText(item.imagePath, item.content, listener);
        } else {
            listener.onCallback(ShareListener.STATE_FAIL, "分享信息不完整");
        }
    } else {
        listener.onCallback(ShareListener.STATE_FAIL, "不支持的分享类型");
    }
}
```

可以看到，上面的代码分为外部接口`share`和内部接口`shareImpl`，`ShareItem`和`ShareListener`的判断都放在`share`里完成，那么`shareImpl`就减少了`if else`的嵌套了，相当于把`if else`分摊了。这样一来，代码的可读性好很多，嵌套也不超过3层了。

但可以看到，`shareImpl`里还是包含分享类型的判断，也即业务判断，我们都清楚产品经理的脑洞有多大了，分享的类型随时会改变或添加。嗯说到这里相信大家都想到用多态了。多态不但能应付业务改变的情况，也可以用来减少`if else`的嵌套。



### 减少 if else 方法二：多态

利用多态，每种业务单独处理，在接口不再做任何业务判断。把`ShareItem`抽象出来，作为基础类，然后针对每种业务各自实现其子类：

```java
public abstract class ShareItem {
    int type;

    public ShareItem(int type) {
        this.type = type;
    }

    public abstract void doShare(ShareListener listener);
}

public class Link extends ShareItem {
    String title;
    String content;
    String link;

    public Link(String link, String title, String content) {
        super(TYPE_LINK);
        this.link = !TextUtils.isEmpty(link) ? link : "default";
        this.title = !TextUtils.isEmpty(title) ? title : "default";
        this.content = !TextUtils.isEmpty(content) ? content : "default";
    }

    @Override
    public void doShare(ShareListener listener) {
        // do share
    }
}

public class Image extends ShareItem {
    String imagePath;

    public Image(String imagePath) {
        super(TYPE_IMAGE);
        this.imagePath = !TextUtils.isEmpty(imagePath) ? imagePath : "default";
    }

    @Override
    public void doShare(ShareListener listener) {
        // do share
    }
}

public class Text extends ShareItem {
    String content;

    public Text(String content) {
        super(TYPE_TEXT);
        this.content = !TextUtils.isEmpty(content) ? content : "default";
    }

    @Override
    public void doShare(ShareListener listener) {
        // do share
    }
}

public class ImageText extends ShareItem {
    String content;
    String imagePath;

    public ImageText(String imagePath, String content) {
        super(TYPE_IMAGE_TEXT);
        this.imagePath = !TextUtils.isEmpty(imagePath) ? imagePath : "default";
        this.content = !TextUtils.isEmpty(content) ? content : "default";
    }

    @Override
    public void doShare(ShareListener listener) {
        // do share
    }
}
```

（注意：上面每个子类的构造方法还对每个字段做了空值处理，为空的话，赋值`default`，这样如果用户传了空值，在调试就会发现问题。）

实现了多态后，分享接口的就简洁多了：

```java
public void share(ShareItem item, ShareListener listener) {
    if (item == null) {
        if (listener != null) {
            listener.onCallback(ShareListener.STATE_FAIL, "ShareItem 不能为 null");
        }
        return;
    }

    if (listener == null) {
        listener = new ShareListener() {
            @Override
            public void onCallback(int state, String msg) {
                Log.i("DEBUG", "ShareListener is null");
            }
        };
    }

    shareImpl(item, listener);
}

private void shareImpl (ShareItem item, ShareListener listener) {
    item.doShare(listener);
}
```

嘻嘻，怎样，内部接口一个`if else`都没了，是不是很酷~ 如果这个分享功能是自己App里面的功能，不是第三方SDK，到这里已经没问题了。但如果是第三方分享SDK的功能的话，这样暴露给用户的类增加了很多（各`ShareItem`的子类，相当于把`if else`抛给用户了），用户的接入成本提高，违背了“迪米特原则”了。

处理这种情况也很简单，再次封装一层即可。把`ShareItem`的子类的访问权限降低，在暴露给用户的主类里定义几个方法，在内部帮助用户创建具体的分享类型，这样用户就无需知道具体的类了：

```java
public ShareItem createLinkShareItem(String link, String title, String content) {
    return new Link(link, title, content);
}

public ShareItem createImageShareItem(String ImagePath) {
    return new Image(ImagePath);
}

public ShareItem createTextShareItem(String content) {
    return new Text(content);
}

public ShareItem createImageTextShareItem(String ImagePath, String content) {
    return new ImageText(ImagePath, content);
}
```

或者有人会说，这样用户也需额外了解多几个方法。我个人觉得让用户了解多几个方法好过了解多几个类，而已方法名一看就能知道意图，成本还是挺小，是可以接受的。

其实这种情况，更多人想到的是使用工厂模式。嗯，工厂模式能解决这个问题（其实也需要用户额外了解多几个`type`类型），但工厂模式难免又引入分支，我们可以用`Map`消除分支。

### 减少 if else 方法三：使用Map替代分支语句

把所有分享类型预先缓存在`Map`里，那么就可以直接`get`获取具体类型，消除分支：

```java
private Map<Integer, Class<? extends ShareItem>> map = new HashMap<>();

private void init() {
    map.put(TYPE_LINK, Link.class);
    map.put(TYPE_IMAGE, Image.class);
    map.put(TYPE_TEXT, Text.class);
    map.put(TYPE_IMAGE_TEXT, ImageText.class);
}

public ShareItem createShareItem(int type) {
    try {
        Class<? extends ShareItem> shareItemClass = map.get(type);
        return shareItemClass.newInstance();
    } catch (Exception e) {
        return new DefaultShareItem(); // 返回默认实现，不要返回null
    } 
}
```

这种方式跟上面分为几个方法的方式各有利弊，看大家取舍了~

#### 写在最后

讲到这里大家有没收获呢？总结下减少`if else`的方法：

- 把接口分为外部和内部接口，所有空值判断放在外部接口完成；而内部接口传入的变量由外部接口保证不为空，从而减少空值判断。
- 利用多态，把业务判断消除，各子类分别关注自己的实现，并实现子类的创建方法，避免用户了解过多的类。
- 把分支状态信息预先缓存在`Map`里，直接`get`获取具体值，消除分支。