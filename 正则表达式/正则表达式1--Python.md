```python
# coding:utf-8
import re

# re.match是用来进行正则匹配检查的方法，若字符串匹配正则表达式，则match方法返回匹配对象（Match Object），否则返回None（注意不是空字符串""）。
# 匹配对象Macth Object具有group方法，用来返回字符串的匹配部分。

#正则表达式通常被用来检索、替换那些匹配某个模式的文本。

# 匹配以itcast开头的语句   
ret = re.match("itcast","itcast.cn")
print(ret.group())		# 		--->  itcast
a
# .  匹配任意字符 
ret = re.match(".", "aaa")
print(ret.group())		# 		--->  a

# [ ]  匹配[ ]中的 字符 
# \d   匹配数字，即0-9
# \D   匹配非数字，即不是数字
# \s   匹配空白，即 空格，tab键
# \S   匹配非空白
# \w   匹配单词字符，即a-z、A-Z、0-9、_
# \W   匹配非单词字符 

ret = re.match(".", "a")
print(ret.group())		# 		--->  a

# 如果hello的首字符小写，那么正则表达式需要小写的h  
ret = re.match("h", "hello Python")
print(ret.group())		#		---> h
print(ret)  			#		---> <_sre.SRE_Match object; span=(0, 1), match='h'>

# 如果hello的首字符大写，那么正则表达式需要大写的H  	--->H
ret = re.match("H", "Hello Python")
print(ret.group())

# 大小写h都可以的情况  							--->h
ret = re.match("[hH]", "hello Python")
print(ret.group())
# --->H
ret = re.match("[hH]", "Hello Python")
print(ret.group())

# 匹配0到9第一种写法 							---->7
ret = re.match("[0123456789]","7Hello Python")
print(ret.group())

# 匹配0到9第二种写法 							---->7
ret = re.match("[0-9]","7Hello Python")
print(ret.group())

# 普通的匹配方式 								---->嫦娥1号
ret = re.match("嫦娥1号", "嫦娥1号发射成功")
print(ret.group())

# 使用\d进行匹配 								---->嫦娥1号
ret = re.match("嫦娥\d号", "嫦娥1号发射成功")
print(ret.group())

print("*********************************************原始字符*********************************************")
#
# Python中字符串前面加上 r 表示原生字符串，

# 与大多数编程语言相同，正则表达式里使用"\"作为转义字符，这就可能造成反斜杠困扰。假如你需要匹配文本中的字符"\"，
# 那么使用编程语言表示的正则表达式里将需要4个反斜杠"\\"：前两个和后两个分别用于在编程语言里转义成反斜杠，
# 转换成两个反斜杠后再在正则表达式里转义成一个反斜杠。
#
# Python里的原生字符串很好地解决了这个问题，有了原始字符串，你再也不用担心是不是漏写了反斜杠，写出来的表达式也更直观。
mm = "c:\\a\\b\\c 本来就是一个斜杠 ,不过是使用转义字符罢了,然后在正则表达式里面再次转义"
print(mm) 							# --->c:\a\b\c 本来就是一个斜杠 ,不过是使用转义字符罢了,然后在正则表达式里面再次转义
re.match("c:\\\\",mm).group() 
ret = re.match("c:\\\\",mm).group()
print(ret) 							# --->c:\
ret = re.match("c:\\\\a",mm).group()
print(ret) 							# --->c:\a
ret = re.match(r"c:\\a",mm).group()
print(ret) 							# --->c:\a

print("************数量************")
# *    匹配前一个字符出现0次或者无限次，即可有可无
# +    匹配前一个字符出现1次或者无限次，即至少有1次
# ?    匹配前一个字符出现1次或者0次，即要么有1次，要么没有
# {m}  匹配前一个字符出现m次
# {m,} 匹配前一个字符至少出现m次
# {m,n}匹配前一个字符出现从m到n次

# 一个字符串第一个字母为大小字符，后面都是小写字母并且这些小写字母可有可无
ret = re.match("[A-Z][a-z]*","Mm")
print(ret.group()) 							# --->Mm
ret = re.match("[A-Z][a-z]*","Aabcdef")
print(ret.group()) 							# --->Aabcdef

# 匹配出，变量名是否有效
ret = re.match("[a-zA-Z_]+[\w_]*","name1")
print(ret.group())							# --->name1
ret = re.match("[a-zA-Z_]+[\w_]*","_name")
print(ret.group())							# --->_name
ret = re.match("[a-zA-Z_]+[\w_]*","2_name")
# print(ret.group())            AttributeError: 'NoneType' object has no attribute 'group'

# 匹配出，0到99之间的数字 ? 匹配前一个字符出现1次或者0次
ret = re.match("[1-9]?[0-9]","7")
print(ret.group()) 							# --->7

ret = re.match("[1-9]?[0-9]","33")
print(ret.group()) 							# --->33

ret = re.match("[1-9]?[0-9]","09")
print(ret.group()) 							# --->0

# 匹配出，8到20位的密码，可以是大小写英文字母、数字、下划线
ret = re.match("[a-zA-Z0-9_]{6}","12a3g45678")
print(ret.group()) 							# 6次--->12a3g4

ret = re.match("[a-zA-Z0-9_]{8,20}","1ad12f23s34455ff66")
print(ret.group()) 							# 8,20次--->1ad12f23s34455ff66 
```

```python
print("***********边界**********")
# ^    匹配字符串开头
# $    匹配字符串结尾
# \b   匹配一个单词的边界
# \B   匹配非单词边界

# 需求：匹配163.com的邮箱地址
# 正确的地址 字符4-20遍   				--->xiaoWang@163.com
ret = re.match("[\w]{4,20}@163\.com", "xiaoWang@163.com")
print(ret.group())

# 不正确的地址		   				 --->xiaoWang@163.com
ret = re.match("[\w]{4,20}@163\.com", "xiaoWang@163.comheihei")
print(ret.group())

# 通过$来确定末尾 ,com 是结尾
ret = re.match("[\w]{4,20}@163\.com$", "xiaoWang@163.comheihei")
# print(ret.group())            AttributeError: 'NoneType' object has no attribute 'group'

# 看ver左右的空格和字符 , ver就是我们说的单词		  --->None
re.match(r".*\bver\b", "ho ver abc").group()
print(ret)

re.match(r".*\bver\b", "ho verabc")
# AttributeError: 'NoneType' object has no attribute 'group'
re.match(r".*\bver\b", "hover abc")
# AttributeError: 'NoneType' object has no attribute 'group'

re.match(r".*\Bver\B", "hoverabc").group() 		  #--->None
print(ret)

re.match(r".*\Bver\B", "ho verabc")
# AttributeError: 'NoneType' object has no attribute 'group'

re.match(r".*\Bver\B", "hover abc")
# AttributeError: 'NoneType' object has no attribute 'group'

re.match(r".*\Bver\B", "ho ver abc")
# AttributeError: 'NoneType' object has no attribute 'group'

print("********************匹配分组************************")
# |    匹配左右任意一个表达式
# (ab) 将括号中字符作为一个分组
# \num 引用分组num匹配到的字符串
# (?P<name>)   分组起别名
# (?P=name)    引用别名为name分组匹配到的字符串

ret = re.match("[1-9]?\d","8").group() 		  #		--->8
print(ret)

ret = re.match("[1-9]?\d","78").group() 	  #		--->78
print(ret)

# 不正确的情况
ret = re.match("[1-9]?\d","08").group() 	  #		--->0
print(ret)

# 修正之后的
ret = re.match("[1-9]?\d$","08")			  #	 --->None

# 添加|
ret = re.match("[1-9]?\d$|100","8").group() 	#	 --->8
print(ret)

ret = re.match("[1-9]?\d$|100","78").group() 	#	 --->78
print(ret)

ret = re.match("[1-9]?\d$|100","08")			#	 --->None
print(ret)
ret = re.match("[1-9]?\d$|100","100").group() 	#	 --->100
print(ret)


# 需求：匹配出163、126、qq邮箱之间的数字		   ------>test@163.com
ret = re.match("\w{4,20}@163\.com", "test@163.com")
print(ret.group())
#											------>test@126.com
ret = re.match("\w{4,20}@(163|126|qq)\.com", "test@126.com")
print(ret.group())
#											------>test@qq.com
ret = re.match("\w{4,20}@(163|126|qq)\.com", "test@qq.com")
print(ret.group())

ret = re.match("\w{4,20}@(163|126|qq)\.com", "test@gmail.com")


# 需求：匹配出<html>hh</html>


# 能够完成对正确的字符串的匹配  						------> <html>hh</html>
ret = re.match("<[a-zA-Z]*>\w*</[a-zA-Z]*>", "<html>hh</html>")
print(ret.group())


# 如果遇到非正常的html格式字符串，匹配出错		  	------> <html>hh</htmlbalabala>
ret = re.match("<[a-zA-Z]*>\w*</[a-zA-Z]*>", "<html>hh</htmlbalabala>")
print(ret.group())


# 正确的理解思路：如果在第一对<>中是什么，按理说在后面的那对<>中就应该是什么
#  (ab)	将括号中字符作为一个分组   \num	引用分组num匹配到的字符串
# 通过引用分组中匹配到的数据即可，但是要注意是元字符串，即类似 r""这种格式  
ret = re.match(r"<([a-zA-Z]*)>\w*</\1>", "<html>hh</html>")
print(ret.group())


# 因为2对<>中的数据不一致，所以没有匹配出来
ret = re.match(r"<([a-zA-Z]*)>\w*</\1>", "<html>hh</htmlbalabala>")

# 需求：匹配出<html><h1>www.lovedj.cn</h1></html>

ret = re.match(r"<(\w*)><(\w*)>.*</\2></\1>", "<html><h1>www.lovedj.cn</h1></html>")
print(ret.group())


ret = re.match(r"<(\w*)><(\w*)>.*</\2></\1>", "<html><h1>www.lovedj.cn</h2></html>")
# print(ret.group())

# 需求：匹配出<html><h1>www.lovedj.cn</h1></html>
ret = re.match(r"<(?P<name1>\w*)><(?P<name2>\w*)>.*</(?P=name2)></(?P=name1)>", "<html><h1>www.lovedj.cn</h1></html>")
print(ret.group())


ret = re.match(r"<(?P<name1>\w*)><(?P<name2>\w*)>.*</(?P=name2)></(?P=name1)>", "<html><h1>www.lovedj.cn</h2></html>")
```