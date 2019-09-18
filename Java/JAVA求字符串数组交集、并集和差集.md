### JAVA求字符串数组交集、并集和差集

```java
package string;   
  
import java.util.HashMap;   
import java.util.HashSet;   
import java.util.LinkedList;   
import java.util.Map;   
import java.util.Map.Entry;   
import java.util.Set;   
  
public class StringArray {   
    public static void main(String[] args) {   
        //测试union   
        String[] arr1 = {"abc", "df", "abc"};   
        String[] arr2 = {"abc", "cc", "df", "d", "abc"};   
        String[] result_union = unionSet(arr1, arr2);   
        System.out.println("求并集的结果如下：");   
        for (String str : result_union) {   
            System.out.println(str);   
        }   
        System.out.println("---------------------可爱的分割线------------------------");   
  
        //测试insect   
        String[] result_insect = intersect(arr1, arr2);   
        System.out.println("求交集的结果如下：");   
        for (String str : result_insect) {   
            System.out.println(str);   
        }   
  
         System.out.println("---------------------疯狂的分割线------------------------");   
          //测试minus   
        String[] result_minus = minus(arr1, arr2);   
        System.out.println("求差集的结果如下：");   
        for (String str : result_minus) {   
            System.out.println(str);   
        }   
    }   
  
    //求两个字符串数组的并集，利用set的元素唯一性   
    public static String[] unionSet(String[] arr1, String[] arr2) {   
        Set<String> set = new HashSet<String>();   
        for (String str : arr1) {   
            set.add(str);   
        }   
        for (String str : arr2) {   
            set.add(str);   
        }   
        String[] result = {};   
        return set.toArray(result);   
    }   
  
    //求两个数组的交集   
    public static String[] intersect(String[] arr1, String[] arr2) {   
        Map<String, Boolean> map = new HashMap<String, Boolean>();   
        LinkedList<String> list = new LinkedList<String>();   
        for (String str : arr1) {   
            if (!map.containsKey(str)) {   
                map.put(str, Boolean.FALSE);   
            }   
        }   
        for (String str : arr2) {   
            if (map.containsKey(str)) {   
                map.put(str, Boolean.TRUE);   
            }   
        }   
  
        for (Entry<String, Boolean> e : map.entrySet()) {   
            if (e.getValue().equals(Boolean.TRUE)) {   
                list.add(e.getKey());   
            }   
        }   
  
        String[] result = {};   
        return list.toArray(result);   
    }   
  
    //求两个数组的差集   
    public static String[] minus(String[] arr1, String[] arr2) {   
        LinkedList<String> list = new LinkedList<String>();   
        LinkedList<String> history = new LinkedList<String>();   
        String[] longerArr = arr1;   
        String[] shorterArr = arr2;   
        //找出较长的数组来减较短的数组   
        if (arr1.length > arr2.length) {   
            longerArr = arr2;   
            shorterArr = arr1;   
        }   
        for (String str : longerArr) {   
            if (!list.contains(str)) {   
                list.add(str);   
            }   
        }   
        for (String str : shorterArr) {   
            if (list.contains(str)) {   
                history.add(str);   
                list.remove(str);   
            } else {   
                if (!history.contains(str)) {   
                    list.add(str);   
                }   
            }   
        }   
  
        String[] result = {};   
        return list.toArray(result);   
    }  
    // 合并数组
      public static void arrayCopy(String[] a,String[] b){
        String[] c=new String[a.length+b.length];
        System.arraycopy(a, 0, c, 0, a.length);
        System.arraycopy(b, 0, c, a.length, b.length);
        System.out.println("---------合并后的数组为：------------");
        for (String string : c) {
            System.out.print(string+",");
        }
    }
    
    
    //按照中文第一个字母升序排列的排序
    public static void sort(){
        Comparator<Object> com=Collator.getInstance(java.util.Locale.CHINA);  
        String[] newArray={"中山","汕头","广州","安庆","阳江","南京","武汉","北京","安阳","北方"};  
        Arrays.sort(newArray,com);  
        for(String i:newArray){  
            System.out.print(i+"  ");  
        }  
    }

}
```

# [Java中遍历Set集合的方法](https://www.cnblogs.com/magicya/p/6683052.html)

```java
对 set 的遍历  
  
1.迭代遍历：  
Set<String> set = new HashSet<String>();  
Iterator<String> it = set.iterator();  
while (it.hasNext()) {  
  String str = it.next();  
  System.out.println(str);  
}  
  
2.for循环遍历：  
for (String str : set) {  
      System.out.println(str);  
}  
  
  
优点还体现在泛型 假如 set中存放的是Object  
  
Set<Object> set = new HashSet<Object>();  
for循环遍历：  
for (Object obj: set) {  
      if(obj instanceof Integer){  
                int aa= (Integer)obj;  
             }else if(obj instanceof String){  
               String aa = (String)obj  
             }  
              ........  
}
```





# [Java写入文件的几种方法及性能对比](https://www.cnblogs.com/wanying521/p/5179256.html)

```java
// Java写入文件的几种方法及性能对比
   public void fileOutPutTime(){
        FileOutputStream out = null;   
        FileOutputStream outSTr = null;   
        BufferedOutputStream Buff=null;   
        FileWriter fw = null;   
        int count=50000;//写文件行数   

        try {   
            //****************************FileOutputStream保存**************************************
            out = new FileOutputStream(new File("E:/add0.html"));   
            long begin = System.currentTimeMillis();   
            for (int i = 0; i < count; i++) {   
                out.write("测试java 文件操作".getBytes());   
            }   
            out.close();   
            long end = System.currentTimeMillis();   
            System.out.println("FileOutputStream执行耗时:" + (end - begin) + " 豪秒");   
            
            //****************************BufferedOutputStream保存***********************************
            outSTr = new FileOutputStream(new File("E:/add1.html"));   
             Buff=new BufferedOutputStream(outSTr);   
            long begin0 = System.currentTimeMillis();   
            for (int i = 0; i < count; i++) {   
                Buff.write("测试java 文件操作".getBytes());   
            }   
            Buff.flush();   
            Buff.close();   
            long end0 = System.currentTimeMillis();   
            System.out.println("BufferedOutputStream执行耗时:" + (end0 - begin0) + " 豪秒");   

            //****************************fileWriter保存*********************************************
            fw = new FileWriter("E:/add2.html");   
            long begin3 = System.currentTimeMillis();   
            for (int i = 0; i < count; i++) {   
                fw.write("测试java 文件操作");   
            }   
            fw.close();   
            long end3 = System.currentTimeMillis();   
            System.out.println("FileWriter执行耗时:" + (end3 - begin3) + " 豪秒");   
            
            //****************************outPutStreamWriter保存**************************************
            FileOutputStream os = new FileOutputStream(new File("E:/add3.html"));
            OutputStreamWriter osWriter = new OutputStreamWriter(os, "UTF-8");
            long begin4 = System.currentTimeMillis();  
            for (int i = 0; i < count; i++) {
                osWriter.write("测试java 文件操作");
            }
            osWriter.close();
            long end4 = System.currentTimeMillis();   
            os.close();
            System.out.println("outPutStreamWriter执行耗时:" + (end4 - begin4) + " 豪秒");  
            
            FileWriter fileWritter = new FileWriter(new File("E:/add4.html"));
            BufferedWriter bufferWritter = new BufferedWriter(fileWritter);
            long begin5 = System.currentTimeMillis();
            for (int i = 0; i < count; i++) {
            bufferWritter.write("测试java 文件操作");
            }
            bufferWritter.close();
            long end5 = System.currentTimeMillis();   
            System.out.println("BufferedWriter执行耗时:" + (end5 - begin5) + " 豪秒");
        } catch (Exception e) {   
            e.printStackTrace();   
        } finally {   
            try {   
                fw.close();   
                Buff.close();   
                outSTr.close();   
                out.close();   
            } catch (Exception e) {   
                e.printStackTrace();   
            }   
        }   
    }   
    }
```

