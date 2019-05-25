##  [Android之使用本地缓存数据](https://blog.csdn.net/loverleslie/article/details/84100646)

前言： 

在通常做项目的时候，需要存储数据，会使用GreenDAO数据库,bmob后端云，或者其他方法，以及本篇文章所讲解的本地缓存，也就是通过SharedPreferences,来进行缓存

### 第一部分：

+ 1.那么首先呢需要创建一个缓存数据的类CarStorage：

+ 2.主要是创建了一个sparseArray的集合，那么sparseArray的性能存储方面，比hashmap更加适合存储数据.。

+ 3.我们所需要呈现的效果是，当运行项目的时候，就要加载好数据，所以在application初始化类中，获取到context，创建一个单例模式，进行初始化。
```
/**
 * 购物车选择的物品存储类：
 */
public class CarStorage {
    private Context context;
    private static CarStorage instance;//购物车实例对象：
    public static final String JSON_CART = "json_cart";
    private SparseArray<goodsBean> sparseArray;//存储商品创建一个优于hashmap的集合
 
    /**
     * 第二步：构造方法+创建存储集合;
     * @param context
     */
    private CarStorage(Context context){
        this.context=context;
        sparseArray=new SparseArray<>(100);
        listToSparse();
    }
 
    /**
     * 第一步：单例模式(懒汉式)：第一次调用的时候，实例购物车：
     * @return
     */
    public static CarStorage getInstance(){
        if (instance==null){
            instance=new CarStorage(MyNews.getContext());
        }
        return instance;
    }
 
```
### 第二部分：读取到本地的数据后，把数据添加到创建好的集合中去：
+ 1.创建一个工具类CacheUtils，来缓存数据：

+ 2.读取本地的数据

+ 3.读取到后，就添加到集合中

 

CacheUtils类：

```
/**
 * 缓工具类：
 * 得到资源后   资源，进行本地缓存：
 */
public class CacheUtils {
    //得到String类型数据：
    public static String getString(Context context, String key) {
        SharedPreferences sp=context.getSharedPreferences("cache",Context.MODE_PRIVATE);
        return sp.getString(key,"");
    }
 
    //保存String类型数据：
    public static void saveString(Context context, String key,String value) {
        //得到String类型数据：
        SharedPreferences sp=context.getSharedPreferences("cache",Context.MODE_PRIVATE);
        sp.edit().putString(key,value).commit();
    }
}
```

读取存储 ：

```
  /**
     * 第三步：1.读取本地数据
     *         2.添加到集合当中：
     */
    private void listToSparse() {
        List<goodsBean> goodsBeanList=getAllData();
        Log.i("zzzz",goodsBeanList.toString());
        //2.把list数据转换成SparseArray；
        for (int i = 0; i < goodsBeanList.size(); i++) {
            goodsBean goodsBean=goodsBeanList.get(i);
            sparseArray.put(Integer.parseInt(goodsBean.getProduct_id()),goodsBean);
        }
    }
 
    //1.读取本地数据:
    public List<goodsBean> getAllData() {
        List<goodsBean> goodsBeanList=new ArrayList<>();
        //1.存本地获取：
        String json=CacheUtils.getString(context,JSON_CART);
        //2.使用gson转成列表：(判断不为空是就执行)
        if (!TextUtils.isEmpty(json)){
            //直接把String转换成list
            goodsBeanList=new Gson().fromJson(json,new TypeToken< List<goodsBean>>(){}.getType());
            Log.i("zzzzz",goodsBeanList.toString());
        }else{
            Log.i("zzzzz","nonononononono");
        }
        return goodsBeanList;
    }
 
```

 第三部分：进行增删改：

```
  /**
     * 添加数据：
     * @param goodsBean
     */
    public void addDate(goodsBean goodsBean){
        //1.添加到内存中——如果商品已存在，就修改数据numer递增：
        goodsBean tempData=sparseArray.get(Integer.parseInt(goodsBean.getProduct_id()));
        if (tempData !=null){
            tempData.setNumber(tempData.getNumber()+1);
        }else{
            tempData=goodsBean;
            tempData.setNumber(1);
        }
        //同步到内存中：
        sparseArray.put(Integer.parseInt(tempData.getProduct_id()),tempData);
        Log.i("aaaa",sparseArray.toString());
        //2.同步到本地
        saveLocal();
    }
    public void deleteDate(goodsBean goodsBean){
        //1.内存中删除
        sparseArray.delete(Integer.parseInt(goodsBean.getProduct_id()));
        //2.同步到本地
        saveLocal();
    }
    public void updateDate(goodsBean goodsBean){
        //1.内存中更新
        sparseArray.put(Integer.parseInt(goodsBean.getProduct_id()),goodsBean);
        //2.同步到本地
        saveLocal();
    }
 
  
```

 第四部分：增删改需要同步本地的操作：

 传数据的时候直接可以调用CarStorage，进行调用方法，增删改查等操作！

```
  /**
     * 同步到本地的方法：
     */
    private void saveLocal() {
        //sparseArray转成list
            List<goodsBean> goodsBeanList=sparseToList();
            //把列表转换成————String类型
            String json=new Gson().toJson(goodsBeanList);
            //把数据储存：
            CacheUtils.saveString(context,JSON_CART,json);
        }
 
    private List<goodsBean> sparseToList() {
        List<goodsBean> goodsBeanList=new ArrayList<>();
        for (int i = 0; i < sparseArray.size(); i++) {
            goodsBean goodsBeans=sparseArray.valueAt(i);
            Log.i("zzzz",sparseArray.toString());
            goodsBeanList.add(goodsBeans);
        }
        return goodsBeanList;
    }
}
```

