# 	Django学习

### http协议

- 请求头和请求体

  ```
  GET / HTTP/1.1
  Host: 127.0.0.1
  Connection: keep-alive
  Cache-Control: max-age=0
  Upgrade-Insecure-Requests: 1
  User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/85.0.4183.121 Safari/537.36
  Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.9
  Sec-Fetch-Site: none
  Sec-Fetch-Mode: navigate
  Sec-Fetch-User: ?1
  Sec-Fetch-Dest: document
  Accept-Encoding: gzip, deflate, br
  Accept-Language: zh-CN,zh;q=0.9,ja;q=0.8
  
  '
  ```

- 响应头和响应体

  ```
  200 OK
  Access-Control-Allow-Origin: *
  cache-control: no-cache
  Content-Security-Policy: script-src 'self' https://www.google-analytics.com; style-src 'self'; img-src 'self' *; object-src 'self';
  Content-Type: text/javascript
  Cross-Origin-Resource-Policy: cross-origin
  ETag: "YImhzN0HNi1WlcdKT9u3HkmCX7M="
  
  
  网页
  ```

  

### web框架本质：(socket)

**接受用户请求，处理用户请求， 响应相关内容**

- ``Http``: 无状态，短连接

- 浏览器（socket客户端）

  网站（socket服务端）

- 自己写网站

  1. socket服务端
  2. 根据URL不同返回不同的内容
     - 路由系统 URL-》函数
  3. 字符串返回给用户
     - 模板引擎渲染
       - HTML充当模板（特殊字符）
     - 自己创造任意数据

socket客户端：

1. 发送IP和端口  http://www.baidu.com/index/

2. 接收响应

   ```
   普通响应：页面直接显示
   重定向响应：再发起一次Http请求
   ```

   

socket服务端：

1. 启动并监听IP和端口，等待用户连接

2. 接收请求进行处理，并返回

   ```
   普通返回：响应头+响应体
   重定向返回：
   LOCATION:"targetServer"
   ```

注意：``django``中没有socket，但是django使用了别人的socket,这些socket都遵循WSGI协议

```python
'cgi': CGIServer,
'flup': FlupFCGIServer,
'wsgiref': WSGIRefServer,##默认使用  所以我们运行的django实际上 就是django+wsgiref
'waitress': WaitressServer,
'cherrypy': CherryPyServer,
'paste': PasteServer,
'fapws3': FapwsServer,
'tornado': TornadoServer,
'gae': AppEngineServer,
'twisted': TwistedServer,
'diesel': DieselServer,
'meinheld': MeinheldServer,
'gunicorn': GunicornServer,
'eventlet': EventletServer,
'gevent': GeventServer,
'geventSocketIO':GeventSocketIOServer,
'rocket': RocketServer,
'bjoern' : BjoernServer,
'auto': AutoServer,
    
```

使用python自带的wsgiref写的一个web框架

```python
#!/usr/bin/env python
#coding:utf-8
from wsgiref.simple_server import make_server
 
def index():
    return 'index'
 
def login():
    return 'login'
 
def routers():
    urlpatterns = (
        ('/index/',index),
        ('/login/',login),
    )
     
    return urlpatterns
 
def RunServer(environ, start_response):
    start_response('200 OK', [('Content-Type', 'text/html')])
    url = environ['PATH_INFO']
    urlpatterns = routers()
    func = None
    #for循环进行路由匹配	
    for item in urlpatterns:
        if item[0] == url:
            func = item[1]
            break
    if func:
        return func()
    else:
        return '404 not found'
     
if __name__ == '__main__':
    httpd = make_server('', 8000, RunServer)
    print "Serving HTTP on port 8000..."
    httpd.serve_forever()
```

### MTV/MVC

models(数据库)    templates（html模板）    views（业务逻辑处理）     ->      MTV

models（数据库）     views（html模板）       controlles（业务逻辑处理）->   MVC

### Http请求生命周期：

1. 请求-> 中间件->请求头-> 提取url->路由关系匹配->函数（模板+数据渲染)->返回用户（响应头+响应体）

   ![image-20210214220446764](image/image-20210214220446764.png)

### Web框架种类

- a, b, c                               Tornado
- [第三方a], b, c                Django
- [第三方a], b, [第三方c]  flask

### Django

```python
pip install django
django-admin startproject my-site
```

#### 目录结构

--mysite
	--mysite
			--setting.py   #Django配置文件
			--urls.py         #路由系统
			--wsgi.py		#用于定义Django用socket, wsgiref, 	uwsgi   

--manage.py  #对当前django程序所有操作可以基于 此文件
--app01
	--admin.py            #Django自带后台管理相关数据配置
	--modal.py			#写类，根据类创建数据库表
	--test.py				#单元测试
	--views.py			 #业务处理

#### 路由系统

- url ->   函数

  1. 静态/login/           				    ->              def login(request):

  2. 动态/add-user/(\w+)/             ->              def login(request, v1):

  3. 动态/add-user/(?P<v1>\w+)/(?P<v2>\w+)             ->              def login(request, v2, v1):

  4. 别名/add-user/(?P<v1>\w+)    name="n1"            ->              def login(request, v2, v1): (可以根据名称反向生成路由)

     ```python
     from django.urls import reverse
     v = reverse('n1', simple={"v1": "asd", "v2": "das"})
         print(v)
     
     {% url "n1" i%}
     ```

  5. 路由分发

     urls.py
     		url('app01', include('app01.urls'))
     app01.urls.py
     		url('index/', views.index)

#### ORM操作

- 创建数据库

- 配置数据库

  ```python
  DATABASES = {
      'default': {
          'ENGINE': 'django.db.backends.mysql',
          'NAME': "my_site_2",
          'USER': 'root',
          'PASSWORD': '123456',
          'HOST': '127.0.0.1',
          'PORT': '3306'
         
         ,
      }
  ```

- models.py

  ```python
  from django.db import models
  class UserInfo(models.Model):
      nid = models.AutoField(primary_key=True)
      username = models.CharField(max_length=32)
      password = models.CharField(max_length=64)
      ug = models.ForeignKey("UserGroup", null=True, on_delete=True)
  
  class UserGroup(models.Model):
      title = models.CharField(max_length=32)
      
  2、django建立多对多关系
  手动建表
  class Boy(models.Model):
      name = models.CharField(max_length=32)
      m = models.ManyToManyField('Girl')
  class Girl(models.Model):
      nick = models.CharField(max_lengh=32)
  class Love(models.Model):
      b = models.ForeignKey('boy', on_delete=True)
      g = models.ForeignKey('girl', on_delete=True)
     
  	class Meta:
          #建立联合主键
          unique_together =[
              'g', 'b'
          ]
  使用many
  class Boy(models.Model):
      name = models.CharField(max_length=32)
      m = models.ManyToManyField('Girl')
  class Girl(models.Model):
      nick = models.CharField(max_lengh=32)
  3、不同字段
  ```

- 注册app

- 创建数据表

  ```pyhton manage.py makemigrations```

  ``python manage.py  migrate``

- 增删改查：

  ```python
  # 创建
      # models.UserGroup.objects.create(title="运营部")
      # 查找
      # group_list = models.UserGroup.objects.all()
      group_list = models.UserGroup.objects.filter(id=3)
      group_list = models.UserGroup.objects.filter(id__gt=1)
      #删除
      # models.UserGroup.objects.filter(id=3).delete()
      #更新
      models.UserGroup.objects.filter(id=3).update(title="公关部")
  ```

- 查询多条数据 

  ```python
  # 获取多个数据
  # all() QuerySet([obj, obj])
  # 获取到的是一个个对象列表
  obj = models.UserGroup.objects.all()
  for item in obj:
      print(item.id, item.title)
  #values() OuerySet([{'nid': "1", "username": "fads"}]
   #获取到的是一个个字典，想要连表需要用双下划线
  obj = models.UserInfo.objects.values('nid', 'username', 'ug__title')
  for item in obj:
      print(item['nid'], item['username'], item['ug__title'])
  # values_list() OuerySet([(1, "sfs"), (2, 'asd')]
  obj = models.UserInfo.objects.values_list('nid', 'username')
  ```

- 跨表操作中的正向反向操作

  ```python
  #UserInfo  ug是FK字段 正向操作获得关联表的信息 一个用户有一个用户类型
  obj = models.UserInfo.objects.all()
  for item in obj:
      print(item.nid, item.username, item.password, item.ug.title)//这里会去再执行一次sql
  obj = models.UserInfo.objects.values('nid', 'username', 'ug__title')
  #UserGroup 表名小写_set 反向操作   一个用户类型对应很多用户
  #当ForeignKey字段设置下列属性
  #	related_name=xxx时    表名小写_set == xxx
  #	related_query_name=xxx时  表名小写_set == xxx_set		
  1、小写的表名_set
  	obj = models.UserGroup.objects.all().first()
      result = obj.userinfo_set.all() #[userInfo对象， userInfo对象]
  2、小写的表名
  	obj = models.UserGroup.objects.values('id','title', '小写的表名__nid')
  ```

- 进行连表查询

  ```sql
  --通过FK连表，会进行多次查询，每用row.ug.title都是一次查询
  q = models.userInfo.objects.all()
  select * from userinfo
  for row in q:
  	print(row.name, row.ug.title)
  --select_related: 查询主动做连表
  models.UserInfo.objects.all().select_related('ug')
  select * from userInfo left join userType on()
  --prefetch_related: 不做连表，做多次查询
  models.UserInfo.objects.all().prefetch_related('ug')   ug_id = [2, 4]
  1、select * from userinfo
  2、select * from usertype where id in [2, 4]
  for row in q:
      print(row.id, row.ut.title)
  ```

- 多对多操作

  ```python
  1、自己建表
  love_list = models.boy.objects.filter(name = "Jenny").first().love_set.all()
  love_list = models.Love.objects.filter(b__name="Jenny")
  love_list = models.Love.objects.filter(b__name="Jenny").select_related('g')
  for row in love_list:
      print(row.g.nike)
  love_list = models.Love.objects.filter(b__name="Jenny").values('g__nike')
  for row in love_list:
      print(row['g__nike'])
  ```

- 多对多中的自关联

  ```python
  两种创建子关联的方式
  1、自己创建额外的表来实现自关联
  class User(models.Model):
      username = models.CharField(max_length=32)
      password = models.CharField(max_length=32)
      gender_choice = ((1, "男"), (2, '女'))
      gender = models.IntegerField(choices=gender_choice)
  
  
  class U2U(models.Model):
      g = models.ForeignKey("User", on_delete=models.CASCADE, related_name="boys")
      b = models.ForeignKey("User", on_delete=models.CASCADE, related_name="girls")
      
  查询：
  wayne = models.User.objects.filter(id=1).first()
  wayne.girls.all()#查到与wayne有关联的女生
  2、使用models.Manytomany字段创建
  class User(models.Model):
      username = models.CharField(max_length=32)
      password = models.CharField(max_length=32)
      gender_choice = ((1, "男"), (2, '女'))
      gender = models.IntegerField(choices=gender_choice)
  
      m = models.ManyToManyField("User")
  查询：
  obj = models.User.object.filter(id=1).first()
  #前面列
  obj.m      => select xx from xx where from_user_id = 1
  后面列
  obj.userinfo_sest => select xx from xx where to_user_id = 1
  ```

- FK自关联

  ![image-20210213220002974](image/image-20210213220002974.png)

  ```python
  class comment(models.Model):
  	news_id = 
  ```

  

- 分页操作

  1. django自带（适合只有上一页下一页的标签， 不适用于其他python框架）

     ```python
     from django.core.paginator import Paginator
     def Pagination(request):
         pNum = request.GET.get('pNum')
         user_list = models.UserInfo.objects.all()
         paginator = Paginator(user_list,10)
         try:
         	posts = paginator.page(pNum)
       	except EmptyPage as e:
             posts=paginator.page(1)
         except PageNotAnInteger as e:
             posts = paginator.page(1)
         return render(request, "PaginatorTest.html", {"posts":posts})
     ```

     ```
     paginator中的方法：
         per_page:每页显示条目数量
         count：  数据总个数
         num_pages:总页数
         page_range: 总页数的索引范围
         page:     page对象
     page对象中常用的方法：
     	has_next: 是否有下一页
     	next_page_number: 下一页页码
     	has_previous: 是否有上一页
     	previous_page_number: 上一页页码
     	number：      当前页页码
     	paginator:    paginator对象
     ```

     

  2. 自制的分页组件

     ```python
     class Pager:
         def __init__(self, per_page, cur_page, total_num, page_view, url):
             '''
     
             :param per_page:    每一页的数据数
             :param cur_page:    当前页码
             :param total_num:   数据库总行数
             :param page_view:   一次显示出的页码
             :param url          url
             '''
             self.per_page = per_page
             try:
                 self.cur_page = int(cur_page)
             except Exception as e:
                 self.cur_page = 1
             # page_num 总页码
             a, b = divmod(total_num, 10)
             if b:
                 self.page_num = a + 1
             else:
                 self.page_num = a
     
             self.page_views = page_view
             self.url = url
     
         def begin(self):
             return (self.cur_page - 1) * self.per_page
     
         def end(self):
             return self.cur_page * self.per_page
     
         def pager(self):
             page = []
             half = int(self.page_views / 2)
             if self.cur_page <= half:
                 begin = 1
                 end = self.page_views + 1
             elif self.cur_page >= self.page_num - half:
                 begin = self.page_num - self.page_views
                 end = self.page_num + 1
             else:
                 begin = self.cur_page - half
                 end = self.cur_page + half + 1
             if self.cur_page <= 1:
                 page.append('''
                 <li>
                      <a href="#" aria-label="Previous">
                         <span aria-hidden="true">&laquo;</span>
                     </a>
                  </li>''')
             else:
                 page.append('''
                 <li>
                     <a href="%s?page=%s" aria-label="Previous">
                          <span aria-hidden="true">&laquo;</span>
                      </a>
                  </li>''' % (self.url, self.cur_page - 1))
             for i in range(begin, end):
                 if i == self.cur_page:
                     page.append("<li class='active'><a href='%s?page=%s'>%s</a><li>" % (self.url, i, i))
                 else:
                     page.append("<li><a href='%s?page=%s'>%s</a><li>" % (self.url, i, i))
     
             if self.cur_page >= self.page_num:
                 page.append('''
                 <li>
                      <a href="#" aria-label="Next">
                         <span aria-hidden="true">&laquo;</span>
                     </a>
                  </li>''')
             else:
                 page.append('''
                 <li>
                     <a href="%s/?page=%s" aria-label="Next">
                          <span aria-hidden="true">&raquo;</span>
                      </a>
                  </li>''' % (self.url, self.cur_page + 1))
             return "".join(page)
     
     ```

- 筛选使用的双下滑线语法：

  ```python
  obj=models.UserInfo.objects.filter(nid__gt=10)                #`nid` > 10
  obj=models.UserInfo.objects.filter(nid__lt=10)                #`nid` < 10
  obj=models.UserInfo.objects.filter(nid__lte=10)               #`nid` <= 10
  obj=models.UserInfo.objects.filter(nid__gte=10)               #`nid` >= 10
  obj=models.UserInfo.objects.filter(nid__in=[1, 2, 3])         #`nid` IN (1, 2, 3)
  obj=models.UserInfo.objects.filter(nid__range=[1, 10])        #BETWEEN 1 AND 10
  obj=models.UserInfo.objects.filter(username__startswith='ha') #LIKE BINARY hao%
  obj=models.UserInfo.objects.filter(username__startswith='ha') #NOT(`nid` = 1)
  ```

  

- 其他（order by, group by, F, Q）

  ```sql
  //order by
  obj = models.UserInfo.objects.all().order_by('-nid') 相当于
  SELECT * FROM `app01_userinfo` ORDER BY `nid` DESC
  
  //group by
  obj = models.UserInfo.objects.values('ug_id').annotate(xxx=Count('ug_id')).filter(xxx__gt=10)相当于
  SELECT `ug_id`, COUNT(`ug_id`) AS `xxx` FROM `app01_userinfo` GROUP BY ``ug_id` HAVING COUNT(.`ug_id`) > 10
  
  //F：An object capable of resolving references to existing query objects
  obj = models.UserInfo.objects.all().update(password=F("username"))
  
  //Q :用于构造复杂的查询条件 对象方式，和直接添加
  1、直接添加
  	obj = models.UserInfo.objects.filter(Q(nid__gt=1) | Q(nid__lt=10))
  2、对象添加
  	conn = Q()
      conn.connector = "OR"
      conn.children.append((nid__gt=1))
      conn.children.append(("nid", 2))
      conn.children.append(("nid", 3))
  
      conn1 = Q()
      conn1.connector = "OR"
      conn1.children.append(("nid", 4))
      conn1.children.append(("nid", 5))
      conn1.children.append(("nid", 6))
  
      conn2 = Q()
      conn2.add(conn, "OR")
      conn2.add(conn1, "OR")
      obj = models.UserInfo.objects.filter(conn2)
  
  ```

- extra,额外的查询条件以及相关的表

  ```sql
  extra(self, select=None, where=None, params=None, tables=None,
                order_by=None, select_params=None)
  1、select select_params=None, 在select子句中添加select查询
  2、where=None, params=None, 在from子句中添加查询条件
  3、tables   : select * from 表， talbes
  4、order_by : 排序
      
  exp:
      models.UserInfo.objects.estra(
      select={‘id': 'select count(1) from %s},
      select_params=["xxxx", ], 
      where = ['age>%s'],
      params = [19, ], 
      order_by=['-age']
      tables = ['userType'])
          相当于
      select userInfo.id, (select count(1) from xxxx) as id
      from userInfo, userType
      where userInfo.age>19
      order by userInfo.age desc
      
  ```

- 原生的SQL语句

  ```python
  1、
  from django.db import connection, connections
  
  cursor = connection.cursor() #connection=default数据
  cursor = connections['db'].cursor()
  cursor.execute("select * from xxx where id=%s",[1])
  
  row = cursor.fetchone()
  row = cursor.fetchall()
  2、extra
  3、
  models.userInfo.objects.row("select * from userInfo")
  ```

  



#### models操作：

1. 不考虑``django admin``	：

   ```sqlite
   1、字段类型
       CharField()
       IntegerField()
       DecimalField()
   
       DateTimeField()
       TimeField()
       sex_list = ((1, '男'), (2, '女'))
           sex = models.IntegerField(choices=sex_list, default=1)
   2、参数
   	null = True
   	default = "xxxx"
   	db_index = True
   	unique = True
   	primay_key = True
   	max_length = 
   	
   	class Meta:
   		unique_togeter = (
   		 ('xxx', 'sdfs'))
   		 index_togeter = (
   		 ('xxx', 'sds'))	
   ```

2. 考虑``django admin``

   ```python
   1、字段类型
   	正则验证
       邮箱
       IP
       URL
       UUID
       ...
   2、参数
   	blank
       error_message={}
       validators=[
           自定义正则
       ]
   ```


#### 模板语言（模板的渲染是在后台进行）

- for， if ， {{}}   {%%}

- 母版（整个页面去继承）``{% extends "xx.html" ``

- include(导入小组件)``{% include "xxx.html"%}``

- 模板自定义函数(simple_filter,  simple_tag)

  1、在``app``中创建``tempaltestag``模块

  2、创建任意``py``文件

  ```python
  from django import template
  register = template.Library()
  #最多两个参数 {{ 第一个参数 | my_upper:"第二个"}}
  @register.filter
  def my_upper(values):
      return values.upper()
  #无限制  {% 函数名 参数 参数%}
  @register.simple_tag
  def my_lower(values):
      return values.lower()
  ```

  3、在模板中使用

  ​		``{% load xx %}``

  ​        ``{{ myStr| my_upper }}``

  ​		``{% my_lower "SFDSF" "DSFSD"%}``

  4、注册app

django母版：

- 母版

  ```
  {%block s1%} {%endblock%}
  {%block s2%} {%endblock%}
  ```

- 子板

  ```
  {%extend "layout.html"%}
  {%block s1%} xxxxx {%endblock%}
  ```

#### 创建django项目

1. 创建project 

   ``django-admin startproject mysite``

2. 配置

   	-模板路径

   ```python
   TEMPLATES = [
       {
           'BACKEND': 'django.template.backends.django.DjangoTemplates',
           'DIRS': [os.path.join(BASE_DIR, 'templates')],
           'APP_DIRS': True,
           ......
       }
   ```

   	-静态文件路径

   ```python
   STATIC_URL = '/static/'
   STATICFILES_DIRS = [
       os.path.join(BASE_DIR, 'static'),
   ]
   ```

3. 额外配置(注释csrf)

4. url对应关系

   /login/  login

   ```python
   def login(request):
       request.POST  ->请求体
       request.GET   ->请求头
       request.method -
       
       
       return HttpResponse("123123")
   	return redirect("url")
   	return render(request, 'xx.html', {})
   ```

5. 模板渲染

   ```python
   def index(request):
       return render(request, 'xxx.html')
   ```

### CBV/FBV

CBV（反射）使用dispatch方法调用getattr实现

![image-20210129212729869](image/image-20210129212729869.png)

![image-20210217154413556](image/image-20210217154413556.png)

### 学员管理系统:

表:

- 班级
- 学生
- 老师

单表操作:增删改查

一对多操作(学生管理)

多对多操作

![image-20201224133740698](https://gitee.com/WAYNEYHN/TyporaPict/raw/master/img/%E8%A1%A81.png)

22:10:51	ALTER TABLE `my_site`.`teacher`  CHANGE COLUMN `tid` `tid` INT(11) NULL AUTO_INCREMENT	Error Code: 1075. Incorrect table definition; there can be only one auto column and it must be defined as a key	0.000 sec

### 模态对话框

1. form表单提交页面会刷新
2. Ajax只会接收服务器返回的字符串，不会自动跳转，如果在服务端重定向只会返回到异步方法里
3. js实现跳转：location.href = '要跳转的地址' 
4. 模态对话框（Ajax）：少量的输入框或数据少
   新URL方式：数据量大，操作多



### Ajax的使用：

``Ajax 不是新的编程语言，而是一种使用现有标准的新方法`

``Ajax 是于服务器交换数据并更新部分网页的技术，在不重新加载整个页面的情况下``

- jQuery 

- ```js
  $.ajax(){
  	url: '要提交的地址',
  	type: 'POST',//GET或POST，提交的方法
      data: {}, //提交的数据
      success: function(data){
          //当前服务端处理完毕后，自动执行回调函数
          //data时服务端返回的数据
      }
  }
  ```

  标签和样式：

  - Bootstrap
  - fontawesome

可以用js阻止默认事件的发生``<a href='xxx.html' onclick="return func()">``
jQuery也可以用来阻止默认时间的发生，只要在jQuery中绑定事件并返回False即可

### js修改后，代码不生效：

```
问题产生原因

　　如果在用户之前已经访问过系统，那么浏览器中会缓存该系统的CSS、JS，这些CSS、JS缓存未过期之前，浏览器只会从缓存中读取CSS和JS，如果在服务器上修改了css和js，那么这些修改在用户的浏览器中是不会有变化的。

解决方案

解决方式一：

用户按Ctrl + F5强制刷新页面或者手动清空了浏览器的缓存。此时浏览器会重新向服务器获取CSS和JS文件,新的文件便会生效。

解决方式二：

       但是用户量过大的时候总不能让每个用户一一清理缓存吧，于是便从代码的角度着手解决这个问题。在js后面添加版本号，让浏览器把这个JS文件当做新的文件重新向服务器获取资源。
```

js可以通过标签取值，也可以通过django的模板语言取值

mysql获取最后插入的id的方法：``select last_insert_id();``





### day3:

1. 学员管理多对对
2. 插件优化页面
3. 用户登录

![image-20210121214635948](image/image-20210121214635948.png)

### bootstrap ：

- 响应式，@media关键字
  - 栅格：将整个页面分为12份
  - 表格
  - 导航条
  - 路径导航
- 样式 

fontawesome

position: absolute

当鼠标移动到xxx样式的标签上时，其子标签.g应用以下属性``.xxx:hover .g{}``



  

### Cookie：

- 保存在浏览器端的键值对
- 服务端可以向用户浏览器写cookie
- 客户端每次请求时，会携带cookie

set_cookie:

```
key,
value,
max_age=None,
expires=None,
path='/'
domain=None
secure=False
httponly=False
```

写cookie

```
@xxxxx
def index(request):
	obj = HttpResponse("sss")
	obj.set_cookie(....)
	request.COOKIES.get(...)
	
	obj.set_signed_cookie(....)
	request.get_signed_cookie(...)
```

cookie签名（可以自定义）：``obj.set_signed_cookie('ticket','123123', salt='dsfsdf')``

装饰器装饰views中的函数

``position：absolute relative fixed``

### Session[https://blog.csdn.net/u012887259/article/details/102425848]

``Djang``o默认支持Session，并且默认是将Session数据存储在数据库中，即：``django_session ``表中。

同时还提供5种类型的session可供使用：数据库、缓存、文件、缓存+数据库、加密cookie

- 保存在服务端的数据（本质是键值对）``{"随机字符串"：{"key1":value1, "key2":value2}}``

- 应用：依赖cookie

- 作用：保持会话（web网站）

- 好处：敏感信息不会给用户

  ```python
  1、设置session（request.session["xxx"]="xxx"）
  	生成随机字符串
  	通过cookie发送给客户端
  	在服务端进行保存
  2、获取session并进行验证(request.session.get("xxx"))
  	获取客户端cookie中的随机字符串
  	在session中进行查找是否存在相应的key
  ```

1. 配置session  ``settings.py``

```python
配置缓存
    SESSION_ENGINE = 'django.contrib.sessions.backends.cache'  # 引擎
    SESSION_CACHE_ALIAS = 'default'                            # 使用的缓存别名（默认内存缓存，也可以是memcache），此处别名依赖缓存的设置
 
 
    SESSION_COOKIE_NAME ＝ "sessionid"                        # Session的cookie保存在浏览器上时的key，即：sessionid＝随机字符串
    SESSION_COOKIE_PATH ＝ "/"                                # Session的cookie保存的路径
    SESSION_COOKIE_DOMAIN = None                              # Session的cookie保存的域名
    SESSION_COOKIE_SECURE = False                             # 是否Https传输cookie
    SESSION_COOKIE_HTTPONLY = True                            # 是否Session的cookie只支持http传输
    SESSION_COOKIE_AGE = 1209600                              # Session的cookie失效日期（2周）
    SESSION_EXPIRE_AT_BROWSER_CLOSE = False                   # 是否关闭浏览器使得Session过期
    SESSION_SAVE_EVERY_REQUEST = False                        # 是否每次请求都保存Session，默认修改之后才保存
```

缓存



### css:

1. ``position:absolute``
2. ``.c1:hover .c2{}``

![image-20210127224625987](image/image-20210127224625987.png)

### xss跨站脚本攻击

不要随意使用以下两种，如果用记得过滤关键字

- ``{{item | safe}}`` 

- ``from django.utils.safestring import mark_safe``
  ``temp = mark_safe(str)``

### CSRF跨站请求伪造

网站在浏览器访问时发送一个算法生成的随机字符串，浏览器再次请求时如果没有这个字符串，会访问错误

- 基本应用

  在form表单中加``{%csrf_token%}``便会在表单中加一个input框，如果使用``{{csrf_token}}``则表示一个字符串

- 全局禁用

  ``'django.middleware.csrf.CsrfViewMiddleware',``

- 局部禁用(csrf_exempt)/局部使用(csrf_protect)

  ```python
  from django.views.decorators.csrf import csrf_exempt， csrf_protect
  @csrf_exempt
  def csrf1(request):
      if request.method == "GET":
          return render(request, 'csrf1.html')
      else:
          return HttpResponse("ok")
  ```

- 在CBV中，可以直接在dispatch函数上面加@csrf_exempt，并且在使用普通装饰器的时候也可以直接在CBV的方法函数上面使用

- Ajax向服务器发送csrf_token

  ```python
  1、通过隐藏的表单获取
  $('input[name="csrfmiddlewaretoken"]')
  2、通过token获取,加在请求头中
  <script src="/static/jquery-cookies.js"></script>
  $.cookie('csrftoken')
  ```

  

  

### 中间件(middleware)

``django ``中的中间件（``middleware``），在``django``中，中间件其实就是一个类，在请求到来和结束后，``django``会根据自己的规则在合适的时机执行中间件中相应的方法。

在中间件中可以定义以下5个方法：

- process_request(self,request)
- process_view(self, request, callback, callback_args, callback_kwargs)
- process_template_response(self,request,response)
- process_exception(self, request, exception)
- process_response(self, request, response)

<img src="image/image-20210215161118494.png" alt="image-20210215161118494" style="zoom:50%; margin-right=0" />

#### 自定义中间件

当你需要对部分或者全部请求进行某个操作时可以使用自定义的中间件

1. 创建中间件类

   ```python
   class Middle1(MiddlewareMixin):
       #process_request
       def process_request(self, request):	
           pass
       #response函数必须有返回值，将response返回
       def process_response(self, request, response):
           return response
   ```

2. 注册中间件

### Form组件：

#### 作用：

1、对数据进行验证

1. 定义规则

   ```python
   from django.forms import Form, fields
   class UserCheck(Form):
       username = fields.CharField(required=True, max_length=16, min_length=4, error_messages={...})
   ```

   

2. 使用

   ```python
   1、创建UserCheck对象
   obj = UserCheck(request.POST)
   2、进行校验（html标签名=Form字段名）
   if obj.is_valid():
   3、信息获取
   错误信息： obj.errors['password'][0]
   正确信息： obj.cleaned_data
   ```

3. 内部原理

   ```python
   实例化对象时
   self.fields = {
       "username" : 正则表达式1
       "password" : 正则表达式2
   }
   在执行obj.is_valid时
   循环的使用正则去匹配self.field中的字段
   flag = True
   errors, cleaned_data
   for k, v in self.fields.items():
       '''
        k： username， password
      	 v: 正则表达式
       '''	
       inputvalue = request.POST.get(k)
       if 正则失败: flag = False
   return flag
       
   ```

2、保留上次输入

​	生成HTML标签

```
1、在用户获取页面的时候：传递一个空的Form对象使用{{ obj.字段名}}生成html标签
2、用户将数据通过表单提交后: 通过用户的数据生成要给Form对象通过渲染生成一个保留上次输入的html标签
```





