### 打包

#### 1 Android Studio 打release包报错 Expected a name but was STRING at line 1 column 99 path $[0].apkInfo.vers

 + Android Studio 打release包报错

 这个问题看log是json问题但不是代码中的问题，而是apk保存地址中的output.json文件的问题，删除掉就好了

#### 2 FileProvider的使用及应用更新时提示：解析包出错、[失败等问题](<https://blog.csdn.net/XST891205/article/details/79169552>)



#### 3  [解决android textview自动换行问题](http://aichixihongshi.iteye.com/blog/1407853)

半角字符与全角字符混乱所致！一般情况下，我们输入的数字、字母以及英文标点都是半角，所以占位无法确定。它们与汉字的占位大大的不同，由于这个原因，导致很多文字的排版都是参差不齐的。对此找到了两种办法可以解决这个问题： 

1. 将textview中的字符全角化。即将所有的数字、字母及标点全部转为全角字符，使它们与汉字同占两个字节，这样就可以避免由于占位导致的排版混乱问题了。 半角转为全角的代码如下，只需调用即可。  

```
/*** 半角转换为全角  */      
public static String ToDBC(String input) {        
​         char[] c = input.toCharArray();
​         for (int i = 0; i < c.length; i++) {     
​         if (c[i] == 12288) {    
​         c[i] = (char) 32;      
​         continue; }
​          if (c[i] > 65280 && c[i] < 65375)
​             c[i] = (char) (c[i] - 65248);
​         }
​         return new String(c);
​     }  
```





2. 去除特殊字符或将所有中文标号替换为英文标号。利用正则表达式将所有特殊字符过滤，或利用replaceAll（）将中文标号替换为英文标号。则转化之后，则可解决排版混乱问题。 

```
/** * 去除特殊字符或将所有中文标号替换为英文标号  */
 public static String stringFilter(String str) {     
	// 替换中文标号
	str = str.replaceAll("【", "[").replaceAll("】", "]").replaceAll("！", "!").replaceAll("：", ":");
	String regEx = "[『』]"; // 清除掉特殊字符 
	Pattern p = Pattern.compile(regEx);
    Matcher m = p.matcher(str);
    return m.replaceAll("").trim();  }  
```

#### 4 ScheduledExecutorService取代 Timer

多线程并行处理定时任务时，Timer运行多个TimeTask时，只要其中之一没有捕获抛出的异常，其它任务便会自动终止运行，使用ScheduledExecutorService则没有这个问题。 
            

    //org.apache.commons.lang3.concurrent.BasicThreadFactory
    ScheduledExecutorService executorService = new ScheduledThreadPoolExecutor(1,
        new BasicThreadFactory.Builder().namingPattern("example-schedule-pool-%d").daemon(true).build());
    executorService.scheduleAtFixedRate(new Runnable() {
        @Override
        public void run() {
            //do something
        }
    },initialDelay,period, TimeUnit.HOURS);
5 基本常量类型，都是值传递

**依赖冲突**
 compile ('第三方库' ,{
        exclude group: 'com.google.gson'
    })