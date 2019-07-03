# [Android 5.0之后隐式声明Intent 启动Service引发的问题](https://blog.csdn.net/l2show/article/details/47421961)

在Android 5.0之后google出于安全的角度禁止了隐式声明Intent来启动Service.也禁止使用Intent filter.否则就会抛个异常出来.

禁止隐式启动应该是防止进程间调service，是避免别人劫持你的服务

(Android 5.0之后修改了HashMap的实现 , hashCode算法修改了 , 也导致了在get之后,在两个系统上同样的数据不同的顺序。如果对存储的数据有顺序需求的话改为使用红黑树构建的TreeMap就OK了.)

```java
    private void validateServiceIntent(Intent service) {
        if (service.getComponent() == null && service.getPackage() == null) {
            if (getApplicationInfo().targetSdkVersion >= Build.VERSION_CODES.LOLLIPOP) {
                IllegalArgumentException ex = new IllegalArgumentException(
                        "Service Intent must be explicit: " + service);
                throw ex;
            } else {
                Log.w(TAG, "Implicit intents with startService are not safe: " + service
                        + " " + Debug.getCallers(2, 3));
            }
        }
    }
```

从源码中的逻辑来看的话,判断一个intent是不是显式声明的点就是component和package,只要这两个有一个生效就不算是隐式声明的,

```java
    public Intent(Context packageContext, Class<?> cls) {
        mComponent = new ComponentName(packageContext, cls);
    }
    public Intent(String action) {
        setAction(action);
    }
    public Intent(String action, Uri uri,
                  Context packageContext, Class<?> cls) {
        setAction(action);
        mData = uri;
        mComponent = new ComponentName(packageContext, cls);
    }
    public Intent setPackage(String packageName) {
        if (packageName != null && mSelector != null) {
            throw new IllegalArgumentException(
                    "Can't set package name when selector is already set");
        }
        mPackage = packageName;
        return this;
    }
```

Intent的源码,可以看到 三种构造方式,设置action来声明Intent是没有构建component的,所以显式声明需要用到第一和第二种构造(还有带packagename或component的拷贝构造),或者后面设置package属性.