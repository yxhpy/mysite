# rest framework原理解析

## FBV 和 CBV

* 在我们的django里面我们可以使用基于方法的视图(FBV)，也可以基于类的方法的视图(CBV)，rest framework就是基于类的视图

### CBV中如何实现的方法映射

* 如果使用django中的CBV不难发现，我们只需要在类方法里面写入`get` `post` `put` `delete` `patch` 等方法的时候，它总能根据我们请求的方法来判断要执行的函数

### 实现原理解析

* 由于我们的CBV需要继承View类，所以它的实现一定是View里的，我们进入View类
* 进入路由时，我们首先会执行`.as_view()`因此先看看这个函数执行的流程
  * ![image-20200515230137812](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20200515230137812.png)
  * 可以看到，这里是需要先执行dispatch函数的
* 我们再看看dispath里面做了什么呢？
  * ![image-20200515230322166](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20200515230322166.png)
  * 从这个函数就看出了，首先将请求方法进行小写处理，并且使用`getatter()`完成映射并调用我们自己创建的view类，这就是为什么通过view类可以自己判断请求方法了。

## CSRF_TOKEN原理解析

### 首先我们知道`CSRF_TOKEN`原始中间件,是基于类实现的

* 引入中间件 中间件类中的方法有哪些？多少个
  * 最多5个
  * 一般
    * process_request
    * process_view
    * process_response
  * 很少用
    * process_exception
    * process_render_template

### 那`csrf_toke`的校验时再哪一步进行的？

 * 查看源码快速得到答案

    * ![image-20200515231418179](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20200515231418179.png)

    * 确实在process_request的时候就进行了csrf_token的提取

    * 猜也可以猜到，那一定是在process_view里面实现的，验证一下

      	* ![image-20200515231929204](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20200515231929204.png)

      * ![image-20200515232038244](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20200515232038244.png)

   * 图一 表示如果使用了这个装饰器不会再验证我们的csrf_token了
   * 图二 表示验证csrf_token失败则抛出了异常

```python
from django.views.decorators.csrf import csrf_exempt # 装饰后不会再验证csrf_token
from django.views.decorators.csrf import csrf_protect # 装饰后会验证csrf_token
```

## rest_framework用户验证

### 通过源码可以获得如何实现用户验证

* 由于每一个CBV都会经过dispatch,因此可以通过dispatch入手，并且查看一下rest_framework帮助我们做了一些什么？

  * ![image-20200516204738213](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20200516204738213.png)
  * 很明显这一步帮助我们进一步封装一些功能在request里面
  * ![image-20200516204923419](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20200516204923419.png)
  * 将一些东西封装到了新的request里面
  * ![image-20200516205007386](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20200516205007386.png)
  * 这里就是关键了，关于用户认证的类都会装进这个List里面,这也是用户认证的关键点
  * ![image-20200516205234651](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20200516205234651.png)
  * dispatch运行这个函数就是用于用户验证的
  * ![image-20200516205321218](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20200516205321218.png)

  * 顺着这个函数下去就会发现

  * ![image-20200516205436341](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20200516205436341.png)

  * 现在这个self是新request对象了，因此可以拿到authentications循环列表，并且将列表里的实例对象运行，返回一个元组，参数一返回给self.user也就是新request.user,参数二返回给self.auth也就是新的request.auth

  * 因此我们想要自己实现用户认证就需要关注下面几个点

    1. `authentication_classes`  是APIView的成员
    2. `self.user` 和 `self.auth` 是新request的成员
    3. 实例对象的`authenticate`方法

  * 具体实现

    * ![image-20200516222132838](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20200516222132838.png)

    * ```python
      from rest_framework.views import APIView
      from rest_framework.authentication import BaseAuthentication
      class MyAuthentication(BaseAuthentication):
          def authenticate(self, request):
              # 如果验证失败了就必须主动抛出异常AuthenticationFailed
              # raise AuthenticationFailed('用户未登录', 403)
              return (1, 2) # 应当返回元组一是reqeust.user二是reqeust.auth
      
      # 1进入View前先执行APIView的dispatch函数，进一步封装request
      class UserLoginView(APIView):
          authentication_classes = [MyAuthentication, ]
      	# 2遍历实例化authentication_classes里面的类并运行里面的authenticate方法
          def post(self, request, *args, **kwargs):
              # self.dispatch()
              print("request.user", request.user)
              print("request.auth", request.auth)
              return HttpResponse('OK')
      ```

### 按照同样的方法很快就可以找到，权限的类了

* ```python
  class MyPermission(BasePermission):
      def has_permission(self, request, view):
          return True
  class UserLoginView(APIView):
      permission_classes = [MyPermission, ]
  
      def post(self, request, *args, **kwargs):
          print("request.user", request.user)
          print("request.auth", request.auth)
          return HttpResponse('OK')
  ```

### 手动实现访问频率阈值

```python
import time
import collections
from rest_framework.throttling import BaseThrottle
from rest_framework.views import APIView
ThrottleDict = collections.defaultdict(list)
class MyThrottle(BaseThrottle):
    def allow_request(self, request, view):
        remote_addr = self.get_ident(request)
        ctime = time.time()
        if not ThrottleDict.get(remote_addr) or len(ThrottleDict[remote_addr]) < 3:
            ThrottleDict[remote_addr].insert(0, ctime)
            return True
        if ctime - ThrottleDict[remote_addr][-1] > 60:
            ThrottleDict[remote_addr].pop()
            return True
        else:
            self.delay = 10 - (ctime - ThrottleDict[remote_addr][-1])
            return False

    def wait(self):
        return self.delay

class UserLoginView(APIView):
    authentication_classes = [MyAuthentication, ]
    permission_classes = [MyPermission, ]
    throttle_classes = [MyThrottle, ]

    def post(self, request, *args, **kwargs):
        # self.dispatch()
        print("request.user", request.user)
        print("request.auth", request.auth)
        return HttpResponse('OK')

```

### 实现版本控制

```python
from rest_framework.versioning import QueryParameterVersioning, URLPathVersioning
class UserLoginView(APIView):
    versioning_class = URLPathVersioning
    def post(self, request, *args, **kwargs):
        print(request.version)
        return HttpResponse('OK')
```

### 实现请求数据的控制

```python
from rest_framework.parsers import JSONParser
    parser_classes = [JSONParser,]
    def post(self, request, *args, **kwargs):
        print(request.data)
        return HttpResponse('OK')
```

## 序列化

### 普通

```python
class UserInfoView(APIView):
    def get(self, request, *args, **kwargs):
        users = User.objects.all().values('pk', 'username', 'permission')
        print(list(users))
        return HttpResponse('1')
```

```
[{'pk': 1, 'username': '小红', 'permission': 1}, {'pk': 2, 'username': '小南', 'permission': 3}, {'pk': 3, 'username': '大兴', 'permission': 2}, {'pk': 4, 'username': '刘志', 'permission': 1}, {'pk': 5, 'username': '罗辉', 'permission': 2}, {'pk': 6, 'username': '青叶', 'permission': 3}]
```

### 使用serializers

```python
class UserSerializers(serializers.ModelSerializer):
    permission = serializers.CharField(source='get_permission_display') # 当使用choose时使用get_xxxx_display
    bo= serializers.SerializerMethodField() # 使用方法get_bo自定义返回
    class Meta:
        model = User # 关联User模型
        fields = ['username', 'password', 'bo', 'permission'] # 选择哪些字段返回使用__all__可以选择模型中的全部
    def get_bo(self, obj):
        hobbies = obj.bobby.all()
        return [{'id':hobby.pk, 'title':hobby.title} for hobby in hobbies]

class UserInfoView(APIView):
    def get(self, request, *args, **kwargs):
        users = User.objects.all()
        serializer = UserSerializers(instance=users, many=True)
        return HttpResponse(json.dumps(serializer.data, ensure_ascii=False))
```

### 带参数的反转url

```python
from django.shortcuts import reverse
reverse('api:hobby', kwargs={'pk': hobby.pk, 'version': "v1.0"}
```

### 使用serializers来实现验证

```python
class HobbySerializersPost(serializers.Serializer):
    username = serializers.CharField(error_messages={'blank': '用户名不能为空！'})

    def validate_username(self, value):
        # 如果发生错误
        # from rest_framework.exceptions import ValidationError
        # raise ValidationError('错误的值')
        # 如果正确则返回值
        return value
from rest_framework.response import Response
class HoobyView(APIView):

    def post(self, request, *args, **kwargs):
        post_valid = HobbySerializersPost(data=request.data)
        if post_valid.is_valid():
            print(post_valid.errors.values())
            return HttpResponse('OK')
        else:
            print(post_valid.errors)
            return HttpResponse('Error')
```

## 分页

### 三种分页方式

```python
from rest_framework.pagination import PageNumberPagination, LimitOffsetPagination, CursorPagination

# 推荐使用的一种方式CursorPagination
class MyCursorPagination(CursorPagination):
    page_size = 2 #每一页的个数
    ordering = 'id' # 排序
    cursor_query_param = 'page' # 传入的参数

class MyModelSerializer(serializers.ModelSerializer):
    permission = serializers.CharField(source='get_permission_display')
    class Meta:
        model = User
        fields = '__all__'
        depth = 10


class PagerView(APIView):
    def get(self, request, *args, **kwargs):
        users = User.objects.all()
        pg = MyCursorPagination()
        page_source = pg.paginate_queryset(users, request=request, view=self)
        sr = MyModelSerializer(instance=page_source, many=True)
        print(sr.data)
        return pg.get_paginated_response(sr.data) # 采用内置渲染方式，返回值包括上一页和下一页
```

## 视图

### GenericAPIView

```python
from rest_framework.generics import GenericAPIView
```

* 基本无用,封装成一些方法，供给其他类使用的

### GenericViewSet

```python
from rest_framework.viewsets import GenericViewSet
```

* 用于映射请求方法view里面函数的方法

* 路由

  * ```python
    urlpatterns = [
        re_path(r'^(?P<version>v\d+\.\d+)/user/view/$', views.ViewView.as_view({'get': 'list'}), name='view'),
    ]
    ```

  * as_view()传入字典来映射对应方法

* 视图

  * ```python
    class ViewView(GenericViewSet):
        def list(self, request, *args, **kwargs):
            return Response('OK')
    ```

  * 这样就会映射到list执行

### ModelViewSet

```python
from rest_framework.viewsets import ModelViewSet
```

* 封装了增删改查的功能，更加的简单

## 路由

### 路由自动生成

```python
from django.urls import path, re_path, include
from . import views
from rest_framework import routers
ro = routers.DefaultRouter()
view_view = ro.register(r'aaa', views.ViewView)
app_name = 'api'
urlpatterns = [
    re_path(r'^', include(view_view.urls)) # 自动生成某视图的路由
]
```

## 渲染器

### 渲染器选择

* 默认的渲染器

  * ![image-20200519144958792](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20200519144958792.png)

  