# [为什么禁止开发人员使用isSuccess作为变量名](https://www.jianshu.com/p/4121df2ae4b9)

一般情况下，我们可以有以下四种方式来定义一个布尔类型的成员变量：

```java
boolean success
boolean isSuccess
Boolean success
Boolean isSuccess
```

前两种和后两种的主要区别是变量的类型不同，前者使用的是boolean，后者使用的是Boolean。

另外，第一种和第三种在定义变量的时候，变量命名是success，而另外两种使用isSuccess来命名的。

首先，我们来分析一下，到底应该是用Boolean来定义，还是使用boolean更好一点。 

我们知道，boolean是基本数据类型，而Boolean是包装类型。

那么，在定义一个成员变量的时候到底是使用包装类型更好还是使用基本数据类型呢？

我们来看一段简单的代码

```java
public class BooleanMainTest {
   public static void main(String[] args) {
       Model model1 = new Model();
       System.out.println("default model : " + model1);
   }
}

class Model {
   /** 定一个Boolean类型的success成员变量*/
   private Boolean success;
   /** 定一个boolean类型的failure成员变量*/
   private boolean failure;

   /** 覆盖toString方法，使用Java 8 的StringJoiner*/
   @Override
   public String toString() {
       return new StringJoiner(", ", Model.class.getSimpleName() + "[","]")
           .add("success=" + success)
           .add("failure=" + failure)
           .toString();
   }
}
```

以上代码输出结果为：

```java
default model : Model[success=null, failure=false]
```

可以看到，当我们没有设置Model对象的字段的值的时候，Boolean类型的变量会设置默认值为`null`，而boolean类型的变量会设置默认值为`false`。

即对象的默认值是`null`，boolean基本数据类型的默认值是`false`。

我建议，在代码中，尽量避免出现和处理null。 

当我们在设计一个接口的时候，对于接口的返回值的定义，尽量避免使用Boolean类型来定义。大多数情况下，别人使用我们的接口返回值时可能用`if(response.isSuccess){}else{}`的方式，如果我们由于忽略没有设置`success`字段的值，就可能导致NPE，这明显是我们不希望看到的。

所以，**当我们要定义一个布尔类型的成员变量时，尽量选择boolean**，而不是Boolean。当然，编程中并没有绝对。

 那么，到底应该是用success还是isSuccess来给变量命名呢。

从语义上面来讲，两种命名方式都可以讲的通，并且也都没有歧义。那么还有什么原则可以参考来让我们做选择呢。

在阿里巴巴Java开发手册中关于这一点，有过一个『强制性』规定：

![](https://upload-images.jianshu.io/upload_images/14326004-23c983ebbadcdb09?imageMogr2/auto-orient/strip%7CimageView2/2/w/1000/format/webp)

那么，为什么会有这样的规定呢？我们看一下POJO中布尔类型变量不同的命名有什么区别吧。

```java
class Model1  {
   private Boolean isSuccess;
   public void setSuccess(Boolean success) {
       isSuccess = success;
   }
   public Boolean getSuccess() {
       return isSuccess;
   }
}

class Model2 {
   private Boolean success;
   public Boolean getSuccess() {
       return success;
   }
   public void setSuccess(Boolean success) {
       this.success = success;
   }
}

class Model3 {
   private boolean isSuccess;
   public boolean isSuccess() {
       return isSuccess;
   }
   public void setSuccess(boolean success) {
       isSuccess = success;
   }
}

class Model4 {
   private boolean success;
   public boolean isSuccess() {
       return success;
   }
   public void setSuccess(boolean success) {
       this.success = success;
   }
}
```

以上代码的setter/getter是使用Intellij IDEA自动生成的，仔细观察以上代码，你会发现以下规律：

- 基本类型自动生成的getter和setter方法，名称都是`isXXX()`和`setXXX()`形式的。
- 包装类型自动生成的getter和setter方法，名称都是`getXXX()`和`setXXX()`形式的。

既然，我们已经达成一致共识使用基本类型boolean来定义成员变量了，那么我们再来具体看下Model3和Model4中的setter/getter有何区别。

我们可以发现，虽然Model3和Model4中的成员变量的名称不同，一个是success，另外一个是isSuccess，但是他们自动生成的getter和setter方法名称都是`isSuccess`和`setSuccess`。

Java Bean中关于setter/getter的规范

关于Java Bean中的getter/setter方法的定义其实是有明确的规定的，根据JavaBeans(TM) Specification规定，如果是普通的参数，命名为propertyName，需要通过以下方式定义其setter/getter：

```java
public <PropertyType> get<PropertyName>();
public void set<PropertyName>(<PropertyType> a);
```

但是，布尔类型的变量propertyName则是另外一套命名原则的：

```java
public boolean is<PropertyName>();
public void set<PropertyName>(boolean m);
```

 

![](https://upload-images.jianshu.io/upload_images/14326004-2d87c1e1ca260dad?imageMogr2/auto-orient/strip%7CimageView2/2/w/1000/format/webp)

通过对照这份JavaBeans规范，我们发现，在Model4中，变量名为isSuccess，如果严格按照规范定义的话，他的getter方法应该叫isIsSuccess。但是很多IDE都会默认生成为isSuccess。

那这样做会带来什么问题呢。

在一般情况下，其实是没有影响的。但是有一种特殊情况就会有问题，那就是发生序列化的时候。

### 序列化带来的影响

关于序列化和反序列化请参考[Java对象的序列化与反序列化](https://links.jianshu.com/go?to=http%3A%2F%2Fmp.weixin.qq.com%2Fs%3F__biz%3DMzI3NzE0NjcwMg%3D%3D%26mid%3D2650120836%26idx%3D1%26sn%3Dc83a980c0871faf607ae613092c69760%26chksm%3Df36bbfa5c41c36b317c103f27b9d99c26aecba52e4bf614bd73dcadc1e4bc5ab8f99fb082eba%26scene%3D21%23wechat_redirect)。我们这里拿比较常用的JSON序列化来举例，看看常用的fastJson、jackson和Gson之间有何区别：

```java
public class BooleanMainTest {

   public static void main(String[] args) throws IOException {
       //定一个Model3类型
       Model3 model3 = new Model3();
       model3.setSuccess(true);

       //使用fastjson(1.2.16)序列化model3成字符串并输出
       System.out.println("Serializable Result With fastjson :" + JSON.toJSONString(model3));

       //使用Gson(2.8.5)序列化model3成字符串并输出
       Gson gson =new Gson();
       System.out.println("Serializable Result With Gson :"+gson.toJson(model3));

       //使用jackson(2.9.7)序列化model3成字符串并输出
       ObjectMapper om = new ObjectMapper();
       System.out.println("Serializable Result With jackson :" +om.writeValueAsString(model3));
   }
}

class Model3 implements Serializable {

   private static final long serialVersionUID = 1836697963736227954L;
   private boolean isSuccess;
   public boolean isSuccess() {
       return isSuccess;
   }
   public void setSuccess(boolean success) {
       isSuccess = success;
   }
   public String getHollis(){
       return "hollischuang";
   }
}
```

以上代码的Model3中，只有一个成员变量即isSuccess，三个方法，分别是IDE帮我们自动生成的isSuccess和setSuccess，另外一个是作者自己增加的一个符合getter命名规范的方法。

以上代码输出结果：

```java
Serializable Result With fastjson :{"hollis":"hollischuang","success":true}
Serializable Result With Gson :{"isSuccess":true}
Serializable Result With jackson :{"success":true,"hollis":"hollischuang"}
```

在fastjson和jackson的结果中，原来类中的isSuccess字段被序列化成success，并且其中还包含hollis值。而Gson中只有isSuccess字段。

我们可以得出结论：**fastjson和jackson在把对象序列化成json字符串的时候，是通过反射遍历出该类中的所有getter方法**，得到getHollis和isSuccess，然后根据JavaBeans规则，他会认为这是两个属性hollis和success的值。直接序列化成json:

```java
{“hollis”:”hollischuang”,”success”:true}
```

但是**Gson并不是这么做的，他是通过反射遍历该类中的所有属性，并把其值序列化成json**:

```java
{“isSuccess”:true}
```

**可以看到，由于不同的序列化工具，在进行序列化的时候使用到的策略是不一样的，所以，对于同一个类的同一个对象的序列化结果可能是不同的。**

前面提到的关于对getHollis的序列化只是为了说明fastjson、jackson和Gson之间的序列化策略的不同，我们暂且把他放到一边，我们把他从Model3中删除后，重新执行下以上代码，得到结果：

```java
Serializable Result With fastjson :{"success":true}
Serializable Result WithGson :{"isSuccess":true}
Serializable Result With jackson :{"success":true}
```

现在，不同的序列化框架得到的json内容并不相同，如果对于同一个对象，我使用**fastjson进行序列化，再使用Gson反序列化会发生什么？** 

```java
public class BooleanMainTest {
   public static void main(String[] args) throws IOException {
       Model3 model3 = new Model3();
       model3.setSuccess(true);
       Gson gson =new Gson();
       System.out.println(gson.fromJson(JSON.toJSONString(model3),Model3.class));
   }
}

class Model3 implements Serializable {
   private static final long serialVersionUID = 1836697963736227954L;
   private boolean isSuccess;
   public boolean isSuccess() {
       return isSuccess;
   }
   public void setSuccess(boolean success) {
       isSuccess = success;
   }
   @Override
   public String toString() {
       return new StringJoiner(", ", Model3.class.getSimpleName() + "[","]")
           .add("isSuccess=" + isSuccess)
           .toString();
   }
}
```

以上代码，输出结果：

```
Model3[isSuccess=false]
```

这和我们预期的结果完全相反，原因是因为JSON框架通过扫描所有的getter后发现有一个isSuccess方法，然后根据JavaBeans的规范，解析出变量名为success，把model对象序列化城字符串后内容为`{"success":true}`。

根据`{"success":true}`这个json串，Gson框架在通过解析后，通过反射寻找Model类中的success属性，但是Model类中只有isSuccess属性，所以，最终反序列化后的Model类的对象中，isSuccess则会使用默认值false。

但是，**一旦以上代码发生在生产环境，这绝对是一个致命的问题。**

所以，作为开发者，我们应该想办法尽量避免这种问题的发生，对于POJO的设计者来说，只需要做简单的一件事就可以解决这个问题了，那就是把isSuccess改为success。

这样，该类里面的成员变量时success，getter方法是isSuccess，这是完全符合JavaBeans规范的。无论哪种序列化框架，执行结果都一样。就从源头避免了这个问题。

引用一下R大关于阿里巴巴Java开发手册中这条规定的评价

（[https://www.zhihu.com/question/55642203](https://links.jianshu.com/go?to=https%3A%2F%2Fwww.zhihu.com%2Fquestion%2F55642203)）：



![img](https:////upload-images.jianshu.io/upload_images/14326004-a0c3d6d8259ba80e?imageMogr2/auto-orient/strip%7CimageView2/2/w/1000/format/webp)

所以，**在定义POJO中的布尔类型的变量时，不要使用isSuccess这种形式，而要直接使用success！**

文围绕布尔类型的变量定义的类型和命名展开了介绍，最终我们可以得出结论，在定义一个布尔类型的变量，尤其是一个给外部提供的接口返回值时，要使用`boolean success`来定义：

```
public class BaseResponse implements Serializable {
   private boolean success;
   private String errorCode;
   private String errorMessage;
}
```

 阿里巴巴java开发手册中有条强制规定: **所有的 POJO 类属性必须使用包装数据类型。 本文中观念与之完全相反。** (因为基本数据类型有默认值，失去了pojo类的意义)

### POJO（Plain Ordinary Java Object）

简单的Java对象，实际就是普通JavaBeans，是为了避免和EJB混淆所创造的简称。

使用POJO名称是为了避免和[EJB](https://baike.baidu.com/item/EJB)混淆起来, 而且简称比较直接. 其中有一些属性及其getter setter方法的类,没有业务逻辑，有时可以作为[VO](https://baike.baidu.com/item/VO)(value -object)或[dto](https://baike.baidu.com/item/dto/16016821)(Data Transform Object)来使用.当然,如果你有一个简单的运算属性也是可以的,但不允许有业务方法,也不能携带有connection之类的方法。