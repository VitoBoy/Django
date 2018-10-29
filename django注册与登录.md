#django用户注册和登录
###1.新建项目和app---首先进入创建好的虚拟环境
新建项目:
```
命令:django-admin startproject 项目名
```
新建app:
```
命令:python manage.py startapp 应用名
```

###2.配置app
在项目中的settings中的
```
INSTALLED_APPS = [

    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    'myapp',
]
```

###3.配置url
```

from django.conf.urls import url
from myapp import views
urlpatterns = [

    # 注册，使用diango自带的User模型
    url(r'^register/', views.register, name='register'),
    url(r'^login',views.login, name='login'),
    url(r'^index',views.index, name='index')
]
```

###4.配置templates
在工程目录下新建templates目录用来存放模板
然后再settings中配置


```

TEMPLATES =[

    {
        'BACKEND': 'django.template.backends.django.DjangoTemplates',
        'DIRS': [os.path.join(BASE_DIR,'templates')],
        'APP_DIRS': True,
        'OPTIONS': {
            'context_processors': [
                'django.template.context_processors.debug',
                'django.template.context_processors.request',
                'django.contrib.auth.context_processors.auth',
                'django.contrib.messages.context_processors.messages',
            ],
        },
    },
]


```

###5.配置数据库
```
DATABASES = {

    'default': {
        'ENGINE': 'django.db.backends.mysql',
        'NAME': 'dj8',
        'USER':'root',
        'PASSWORD':'123456',
        'PORT':3306,
        'HOST':'localhost'
    }
}


```

#实现注册功能
###1.定义一个用户登录表单(forms.py)
forms.py没有强制的规定，建议放在和app的views.py同一目录


```
from django import forms
from django.contrib.auth.models import User
```

```
```
class UserRegisterForm(forms.Form):
```
```

    username = forms.CharField(max_length=10, min_length=2, required=True,
                               error_messages={
                                   'required': '必填',
                                   'max_length': '不能超过10字符',
                                   'min_length': '不能少于2字符'
                               })
    password = forms.CharField(max_length=10, required=True,
                               error_messages={
                                   'required': '必填',
                                   'max_length': '不能超过10字符'
                               })
    password2 = forms.CharField(max_length=10, required=True,
                               error_messages={
                                   'required': '必填',
                                   'max_length': '不能超过10字符'
                               })

```

    def clean(self):
        user = User.objects.filter(username=self.cleaned_data.get('username')).first()
        if user:
            raise forms.ValidationError({'username': '账号已注册，请去登陆'})
        pasword = self.cleaned_data.get('password')
        pasword2 = self.cleaned_data.get('password2')
        if pasword != pasword2:
            raise forms.ValidationError({'password': '密码不一致'})
        return self.cleaned_data
```


class UserLoginForm(forms.Form):

    username = forms.CharField(max_length=10, min_length=2, required=True,
                               error_messages={
                                   'required': '必填',
                                   'max_length': '不能超过10字符',
                                   'min_length': '不能少于2字符'
                               })
    password = forms.CharField(max_length=10, required=True,
                               error_messages={
                                   'required': '必填',
                                   'max_length': '不能超过10字符'
                               })

```

    def clean(self):
        user = User.objects.filter(username=self.cleaned_data.get('username')).first()
        if not user:
            raise forms.ValidationError({'username':'该账号没有注册，请去注册'})

        return self.cleaned_data
```

###2.定义注册视图views.py
```

from django.contrib.auth.decorators import login_required


from django.contrib.auth.models import User
from django.contrib import auth
from django.shortcuts import render
from django.http import HttpResponseRedirect
from django.urls import reverse
from user.forms import UserRegisterForm, UserLoginForm

```


```
```


def register(request):

    if request.method == 'GET':

        return render(request, 'register.html')

    if request.method == 'POST':
        data = request.POST
        # 校验form表单传递的参数
        form = UserRegisterForm(data)
        if form.is_valid():
            User.objects.create_user(username=form.cleaned_data.get('username'),
                                     password=form.cleaned_data.get('password'))
            return HttpResponseRedirect(reverse('user:login'))
        else:
            return render(request, 'register.html', {'errors': form.errors})


```

注册页面

```

{% extends 'base.html' %}

{% block title %}
    注册
{% endblock %}

{% block content %}
    <form action="" method="post">
        姓名: <input type="text" name="username">
        密码: <input type="text" name="password">
        确认密码: <input type="text" name="password2">
        <input type="submit" value="提交">
    </form>
    {% if errors.username %}
        姓名错误: {{ errors.username }}
    {% endif %}
    {% if errors.password %}
        密码错误: {{ errors.password }}
    {% endif %}
    {% if errors.password2 %}
        确认密码错误: {{ errors.password2 }}
    {% endif %}
{% endblock %}

```

###3.定义登录视图

```

def login(request):

    if request.method == 'GET':
        return render(request, 'login.html')

    if request.method == 'POST':
        data = request.POST
        form = UserLoginForm(data)
        if form.is_valid():
            # 使用随机标识符也叫做签名token
            user = auth.authenticate(username=form.cleaned_data.get('username'),
                                     password=form.cleaned_data.get('password'))
            if user:
                # 登录, 向request.user属性赋值，赋值为登录系统的用户对象
                # 1. 向页面的cookie中设置sessionid值，(标识符)
                # 2. 向django_session表中设置对应的标识符
                auth.login(request, user)
                return HttpResponseRedirect(reverse('user:index'))
            else:
                return render(request, 'login.html', {'msg': '密码错误'})
        else:
            return render(request, 'login.html', {'errors': form.errors})


```

登录页面

```

{% extends 'base.html' %}

{% block title %}
    登录
{% endblock %}
{% block content %}
    <form action="" method="post">
        姓名: <input type="text" name="username">
        密码: <input type="text" name="password">
        <input type="submit" value="提交">
    </form>
    {% if errors.username %}
        姓名错误: {{ errors.username }}
    {% endif %}
    {% if errors.password %}
        密码错误: {{ errors.password }}
    {% endif %}

{% endblock %}


```