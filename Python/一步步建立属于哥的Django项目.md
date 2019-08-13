## 一步步建立属于哥的Django项目

### 第一天

1. 在 setting.py 中的配置模板信息
	`TEMPLATES`添加`'DIRS': [os.path.join(BASE_DIR, 'templates')]`
	
2. 在 setting.py 中的配置数据库信息

```python
	DATABASES = {
	    'default': {
	        # 'ENGINE': 'django.db.backends.sqlite3',
        	# 'NAME': os.path.join(BASE_DIR, 'db.sqlite3'),
        	'ENGINE': 'django.db.backends.mysql',
	        'HOST': 'localhost',
	        'PORT': '3306',
	        'USER': 'lovedj',
	        'PASSWORD': 'jianchi1314',
	        'NAME': 'DjGoShop', #  数据库名字
	    }
	}
```

```mysql
net start mysql
mysql -u lovedj -p
create database DjGoShop charset=utf8;
use DjGoShop;
```

3. 配置静态文件路径 , 图片 ,css , js 等

```python
STATIC_URL = '/static/'

STATICFILES_DIRS=[
    os.path.join(BASE_DIR,'static')
]
```

4. 创建应用

   `python manage.py startapp df_user`

![](D:\DJGitBook\Python\img\django创建应用.jpg)

setting.py 中添加应用

```python
INSTALLED_APPS = [
    ...
    'df_user',
]
```

5. 设计模型类

   刚刚创建的 `df_user` 文件夹中的 `models.py` 中 ,  好烦,我忘了加一个字段 , 好麻烦

   ```python
   from django.db import models
   
   class UserInfo(models.Model):
       uname = models.CharField(max_length=20)
       upwd = models.CharField(max_length=40)
       # 加密
       uemail =models.CharField(max_length=30)
       # 多个收货地址的话 , 创建一个专门的地址表 , 跟人一对多的关系
       # 收件人
       uAddressee = models.CharField(max_length=40)
       # 收件地址
       uReceivingAddress = models.CharField(max_length=40)
       # 邮政编码
       uPostalCode = models.CharField(max_length=6)
       # 手机号
       uphone = models.CharField(max_length=11)
   ```

   声明  `python manage.py makemigrations`

   迁移 `python manage.py migrate`

   (1)  python 报错 ModuleNotFoundError: No module named 'MySQLdb'

   ​	原因是： 在python2.x中用mysqldb，但是在python3.x中已经不支持那个组件了。
   取而代之的是：

   ```python
   `import pymysql`
   ```

   　1、在项目文件夹下的_init_.py中导入pymysq包

   ```python
   `import pymysql``pymysql.install_as_MySQLdb()`
   ```

   　　

   ​    2、在settings.py设置数据库

   ```python
   `DATABASES = {``'default'``: {``'ENGINE'``: ``'django.db.backends.mysql'``,``'NAME'``: ``'dbname'``,``'USER'``:``'dbUser'``,``'PASSWORD'``:``'dbPwd'``,``'HOST'``:``''``,#默认本地``'PORT'``:``''``}``}`
   ```

   　　

      3、必须先在mysql创建你的数据库

      4、使用命令建立数据库数据

   ```
   `manage.py migrate`
   ```




### 第二天

1. templates文件夹中创建df_user文件夹 , 将对应模块的html 文件放入里面
2. 打开df_user组件中的 views .py定义视图 , 创建register方法

```python
from django.shortcuts import render

# Create your views here.
def register(request):
    return render(request,'df_user/register.html')
```

在根组件 DjangoShop 的urls.py 文件中



