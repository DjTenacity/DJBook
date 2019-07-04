### android 中 系统日期时间的获取

```java
import java.text.SimpleDateFormat;       
SimpleDateFormat formatter = new SimpleDateFormat ("yyyy年MM月dd日 HH:mm:ss");       
Date curDate = new Date(System.currentTimeMillis());//获取当前时间       
String str = formatter.format(curDate); 
```

#### 可以获取当前的年月时分,也可以分开写:

```java
SimpleDateFormat sDateFormat = new SimpleDateFormat("yyyy-MM-dd hh:mm:ss");       
String date = sDateFormat.format(new java.util.Date()); 
```

#### 如果想获取当前的年月,则可以这样写(只获取时间或秒种一样):

 ```java
SimpleDateFormat sdf = new SimpleDateFormat("yyyy-MM");
String date = sdf.format(new  Java.util.Date());
 ```

#### 当然还有就是可以指定时区的时间(待):

```java
df=DateFormat.getDateTimeInstance(DateFormat.FULL,DateFormat.FULL,Locale.CHINA);  
System.out.println(df.format(new Date())); 
```

#### **如何获取Android系统时间是24小时制还是12小时制**

```java
ContentResolver cv = this.getContentResolver();  
String  strTimeFormat=Android.provider
.Settings.System.getString(cv,android.provider.Settings.System.TIME_12_24);  
if(strTimeFormat.equals("24"))  {}
```

```
Calendar c = Calendar.getInstance();
取得系统日期:year = c.get(Calendar.YEAR)
               month = c.grt(Calendar.MONTH)
               day = c.get(Calendar.DAY_OF_MONTH)
取得系统时间：hour = c.get(Calendar.HOUR_OF_DAY);
                  minute = c.get(Calendar.MINUTE)
```

#### **利用Calendar获取**

```
Calendar c = Calendar.getInstance();
取得系统日期:year = c.get(Calendar.YEAR)
               month = c.grt(Calendar.MONTH)
               day = c.get(Calendar.DAY_OF_MONTH)
取得系统时间：hour = c.get(Calendar.HOUR_OF_DAY);
                  minute = c.get(Calendar.MINUTE)
                    Calendar c = Calendar.getInstance();
取得系统日期:year = c.get(Calendar.YEAR)
                   month = c.grt(Calendar.MONTH)
                   day = c.get(Calendar.DAY_OF_MONTH)
取得系统时间：hour = c.get(Calendar.HOUR_OF_DAY);
                     minute = c.get(Calendar.MINUTE)
```

#### **利用Time获取**

```
Time t=new Time(); // or Time t=new Time("GMT+8"); 加上Time Zone资料。
t.setToNow(); // 取得系统时间。
int year = t.year;
int month = t.month;
int date = t.monthDay;
int hour = t.hour; // 0-23
int minute = t.minute;
int second = t.second;
```

唯一不足是取出时间只有24小时模式.