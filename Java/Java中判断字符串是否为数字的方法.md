# [Java中判断字符串是否为数字的方法](https://www.cnblogs.com/EasonJim/p/8030430.html)

```java
//方法一：用JAVA自带的函数
public static boolean isNumeric(String str){
    for (int i = str.length();--i>=0;){  
        if (!Character.isDigit(str.charAt(i))){
            return false;
        }
    }
    return true;
}

/*方法二：推荐，速度最快
  * 判断是否为整数 
  * @param str 传入的字符串 
  * @return 是整数返回true,否则返回false 
*/
public static boolean isInteger(String str) {  
        Pattern pattern = Pattern.compile("^[-\\+]?[\\d]*$");  
        return pattern.matcher(str).matches();  
}

//方法三：
public static boolean isNumeric(String str){
    Pattern pattern = Pattern.compile("[0-9]*");
    return pattern.matcher(str).matches();   
}

//方法四：
public final static boolean isNumeric(String s) {
    if (s != null && !"".equals(s.trim()))
        return s.matches("^[0-9]*$");
    else
        return false;
}        

//方法五：用ascii码 
public static boolean isNumeric(String str){
    for(int i=str.length();--i>=0;){
        int chr=str.charAt(i);
        if(chr<48 || chr>57)
            return false;
    }
   return true;
}

```

二、org.apache.commons.lang

org.apache.commons.lang.StringUtils;

```java
StringUtils.isNumeric(null)   = false
 StringUtils.isNumeric("")     = true
 StringUtils.isNumeric("  ")   = false
 StringUtils.isNumeric("123")  = true
 StringUtils.isNumeric("12 3") = false
 StringUtils.isNumeric("ab2c") = false
 StringUtils.isNumeric("12-3") = false
 StringUtils.isNumeric("12.3") = false
```

三、try...catch

最笨的一种方式，通过比如Double.pareOf方法，然后抛出异常时判断为字符串。不推荐这种方式！