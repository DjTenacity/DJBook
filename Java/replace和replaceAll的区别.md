### replace和replaceAll的区别

 1)replace的参数是char和CharSequence,即可以支持字符的替换,也支持字符串的替换(CharSequence即字符串序列的意思,说白了也就是字符串);
2)replaceAll的参数是regex,即基于规则表达式的替换,比如,可以通过replaceAll("\\d", "*")把一个字符串所有的数字字符都换成星号;

相同点是都是全部替换,即把源字符串中的某一字符或字符串全部换成指定的字符或字符串,如果只想替换[第一次](https://www.baidu.com/s?wd=第一次&tn=SE_PcZhidaonwhc_ngpagmjz&rsv_dl=gh_pc_zhidao)出现的,可以使用 replaceFirst(),这个方法也是基于规则表达式的替换,但与replaceAll()不同的是,只替换[第一次](https://www.baidu.com/s?wd=第一次&tn=SE_PcZhidaonwhc_ngpagmjz&rsv_dl=gh_pc_zhidao)出现的字符串;
另外,如果replaceAll()和replaceFirst()所用的参数据不是基于规则表达式的,则与replace()替换字符串的效果是一样的,即这两者也支持字符串的操作;
还有一点注意:执行了替换操作后,源字符串的内容是没有发生改变的.

举例如下:

String src = new String("ab43a2c43d");

System.out.println(src.replace("3","f"));=>ab4f2c4fd.
System.out.println(src.replace('3','f'));=>ab4f2c4fd.
System.out.println(src.replaceAll("\\d","f"));=>abffafcffd.
System.out.println(src.replaceAll("a","f"));=>fb43fc23d.
System.out.println(src.replaceFirst("\\d,"f"));=>abf32c43d
System.out.println(src.replaceFirst("4","h"));=>abh32c43d.  