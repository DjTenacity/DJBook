```python
# coding:utf-8
import re

# 需求：匹配出文章阅读的次数  + 匹配前一个字符出现1次或者无限次，即至少有1次

ret = re.search(r"\d+", "阅读次数为 9999")      #  ---->9999
print(ret.group())

# 统计出python、c、c++相应文章阅读的次数    		 #  ---->['9999', '7890', '12345']
ret = re.findall(r"\d+", "python = 9999, c = 7890, c++ = 12345")
print(ret)

# sub 将匹配到的数据进行替换
ret = re.sub(r"\d+", '998', "python = 991")      #  ---->python = 998
print(ret)


def add(temp):
    strNum = temp.group()
    num = int(strNum) + 1
    return str(num)


ret = re.sub(r"\d+", add, "python = 997")      #  ---->python = 998
print(ret)

ret = re.sub(r"\d+", add, "python = 99")       #  ---->python = 100
print(ret)

# split 根据匹配进行切割字符串，并返回一个列表
# 需求：切割字符串“info:xiaoZhang 33 shandong”
						
ret = re.split(r":| ","info:xiaoZhang 33 shandong")
print (ret)						#  --->['info', 'xiaoZhang', '33', 'shandong']


# Python里数量词默认是贪婪的（在少数语言里也可能是默认非贪婪），总是尝试匹配尽可能多的字符；
#
# 非贪婪则相反，总是尝试匹配尽可能少的字符。
#
# 在"*","?","+","{m,n}"后面加上？，使贪婪变成非贪婪。

s="This is a number 234-235-22-423"
r=re.match(".+(\d+-\d+-\d+-\d+)",s)
print(r.group(1)) 								# 			---->4-235-22-423
# 关闭“.+”贪婪模式
r=re.match(".+?(\d+-\d+-\d+-\d+)",s)
print(r.group(1)) 								# 			---->234-235-22-423

# 正则表达式模式中使用到通配字，那它在从左到右的顺序求值时，会尽量“抓取”满足匹配最长字符串，在我们上面的例子里面，
# “.+”会从字符串的启始处抓取满足模式的最长字符，其中包括我们想得到的第一个整型字段的中的大部分，
# “\d+”只需一位字符就可以匹配，所以它匹配了数字“4”，而“.+”则匹配了从字符串起始到这个第一位数字4之前的所有字符。
#
# 解决方式：非贪婪操作符“？”，这个操作符可以用在"*","+","?"的后面，要求正则匹配的越少越好。

print(re.match(r"aa(\d+)","aa2343ddd").group(1))		# 			---->2343
#关掉 + 的贪婪,所以是2
print(re.match(r"aa(\d+?)","aa2343ddd").group(1))		# 			---->2 

print(re.match(r"aa(\d+)ddd","aa2343ddd").group(1))		# 			---->2343
# 优先符合aa  ddd 的条件
print(re.match(r"aa(\d+?)ddd","aa2343ddd").group(1))	# 			---->2343
```

