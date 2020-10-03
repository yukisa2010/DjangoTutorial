# 仮想環境構築
## virtualenv
pip install virtualenv

=> できない。
=> Versionが違うらしい


curl https://bootstrap.pypa.io/get-pip.py -o get-pip.py
- pipをvirtualenvが使えるバージョンに揃える
python get-pip.py pip==20.0.2

- virtualenvの起動
source /usr/local/bin/virtualenvwrapper.sh
    - 上記を[~/.bash_profile]へ記述することで起動時に立ち上がる



# Djangoアプリの作成方法

## 環境
- paizaサーバー

## アプリ立ち上げ
- django-admin startproject django_website
    - Rails でいうrails new .

## serverの起動コマンド
-  python manage.py runserver

## アプリの作成(Django特有の概念)
- python manage.py startapp app1
- 幾つでも作成可能、基本的には一つでOK

## 以降の手順
- django_website/setting.pyにアプリ名を入れる
```python
ALLOWED_HOSTS = ["*"]

INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    'website',
]

```
- view.pyにhtmlを返す「ビューを追加」
```python

from django.views.generic import TemplateView

class IndexView(TemplateView):
    template_name = 'index.html'


```

- templatesフォルダの作成
    - index.htmlの作成
- urls.pyを作成
    - django_websiteフォルダの同名ファイルから中身をコピー

```python


from django.urls import path
from .views import IndexView
from .views import AboutView

urlpatterns = [
    path('', IndexView.as_view()),    
    path('about/', AboutView.as_view()),    
    path('contact/', IndexView.as_view()),    
]

```
- includeの追加箇所
    - django_website/urls.py
    - from, pathへ追加
```python
from django.urls import path, include

urlpatterns = [
    path('admin/', admin.site.urls),
    path('', include("website.urls"))
]
```



## 各ファイルの役割について

- ファイルの道順
    1. django.urls 
    1. website.urls > viewとurlを結びつける
        - Railsでいうcontroller?router?
    1. view > ユーザーに返す 
    1. templates/index.html

> ## Linuxコマンド
>- cp A Bでコピー
>- rm -r A でディレクトリごと削除
> ## Github
> - Error対応
>   - git config --global user.name "yukisa2010"
>   - git config --global user.email yukisa2010@gmail.com

# Djangoテンプレートビュー

## 共通の要素を配置することができる

- _navbar.html > 共通で使用するテンプレート
```html
<div class='navbar'>
    <a href='/'>TOP</a>
    <a href='/about'>自己紹介</a>
</div>
```
- index.html > テンプレートの配置先へ記述
```html
{% include '_navbar.html' %}
```

## CSSの配置
- templateフォルダにcssを入れても使えない。<br>
まずはstaticフォルダを作成し、中にcssファイルを作成。

```html
{% load static %} <!--HTML行頭に追加しないと使えない-->

<head>
    <link rel="stylesheet" href="{% static 'main.css' %}">
</head>
```

> エラー対処
> - キャッシュが残っており、CSSが再読み込みできなかった。サーバーを切断し、接続し直すと無事成功。

## block, base
- HTMLの共通部分をテンプレート化することができる。
    - 書き換えたい部分だけ{% block %}で囲めばOK

- base.html
```html
<body>
    {% include "_navbar.html" %}

    {% block main %}
    コンテンツがありません。
    {% endblock %}

    {% include "_footer.html" %}
</body>
```
- index.html
```html
{% extends 'base.html' %}
{% block main %}
<h1>Hello, World</h1>
{% endblock %}
```
## 画像
- CSSと同様にstaticフォルダの中
- {% load static %}を忘れない

## 変数
- {{ username }} - HTMLに記述
- views.py
    - IndexView等のclassの中に記述
    - 書き方は下記の通りで決まっている
    - TemplateViewの継承。Djangoが用意している

```python
class IndexView(TemplateView):
    template_name = 'index.html'

    def get_context_data(self):
        ctxt = super().get_context_data()
        ctxt['username'] = "ガッキー"
        return ctxt

```
## カンマ区切り
- HTMLファイル
```html
{% load humanize %}

<p>これまでに{{ num_services | intcomma }}個のサービスを作りました！</p>
```
- django_website/setting.py
```python
INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    'django.contrib.humanize', # <<追加
    'website',
]
```
## 条件分岐
```html
{% if %}
{% elif %}
{% else %}
{% endif %}
```

## 現在時刻
- {% now "Y年m月d日 H時i分" %}
- Defaultが「UTC」になっているのでdjango_website/setting.pyを修正
    - TIME_ZONE = 'Asia/Tokyo'

## デバックモード」の外し方
- website/staticfilesフォルダの追加
- django_website/setting.pyへの変更
```python
# デバッグモードを外す
DEBUG = False
## ファイル末尾へ追加
STATIC_ROOT = os.path.join(BASE_DIR, 'staticfiles')
STATICFILES_STORAGE = 'whitenoise.storage.CompresserManifestStaticFilesStorage'
# Middlewareの項目に下記を追加
'whitenoise.middleware.WhiteNoiseMiddleware',

```
- command: python manage.py collectstaticefiles







