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
- django-admin startproject django-website
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


# Djangoテンプレートビュー


