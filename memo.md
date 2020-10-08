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

## デバックモード」の外し方・WhiteNoiseの導入
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
static内のファイルを全てstaticfilesへコピー
- command: python manage.py collectstatic

- whitenoise
    - 静的ファイル導入のため
## サーバー構成
本番設定の構成例
- Django x 3 + whitenoise
- APサーバー(gunicom)
- Webサーバー(リバースプロキシ・Nginx)

- AmazonS3(静的ファイルサーバー)

## STATICの設定
- STATIC_URL
    - AmazonS3の場合
        - STATIC_URL = 'http://amazon~'
    - WhiteNoiseの場合
        - STATIC_URL = '/assets/' # 自由に変更可能

- STATIC_ROOT
    - python manage.py collectstaticを実行した際の保存先

# Djangoを本番サーバーに公開する
- Heroku
    - Githubと連携し、デプロイできるサービス
    - 必要なもの
        - requirements.txt 
            - ライブラリ一覧のこと
            - プロジェクト直下に作成
            - 下記入力
                ```
                django
                whitenoise
                gunicorn
                ```
        - Procfile >> gunicornの起動コマンド
            - 下記を記述(django_website\wsgi.pyを起動)
                ```
                web: gunicorn django_website.wsgi
                ```
    - 手順
        1. Herokuでプロジェクト立ち上げ
        2. Githubにプロジェクトをアップロード
        3. 1.,2.を連携

## DB Serverとの接続
- sqlite
    - 実体はテキストファイル
    - リスクあるので本番サーバーで使用することはない
    - 少人数など
    - 使用方法
        - 何も指定せず、以下のコマンドを実行
        ```python
        python manage.py migrate
        python manage.py createsuperuser
        ```
- MySQL
    - 以下をインストール
    ```python
    pip install dj-database-url
    pip install python-dotenv
    ```
    - MySQL用ライブラリインストール
    ```
    pip install mysqlclient
    mysql -u root
    mysql>ALTER USER 'root'@'localhost' IDENTIFIED BY 'password';
    mysql>CREATE DATABASE dj_db;
    mysql>exit
    mysql -u root -p # 2回目以降ログイン
    ```
    - .envファイル作成
    ```
    DATABASE_URL=mysql://root:password@localhost/pj_db
    ```
    - コマンド実行
    ```
    python manage.py migrate
    python manage.py createsuperuser
    ```
- PostgreSQL
    - ライブラリ・インストール
    ```
    pip install psycopg2-binary
    ```
    - =>うまくいかない。以下、試したこと
        ```terminal
        brew install postgresql
        pip install psycopg2
            => エラー出た。ググって解決

        # postgreSQLサーバーの起動
        pg_ctl start -D /usr/local/var/postgres
        # DB接続
        psql -d postgres -U apple -h localhost
        # 参考URL
        https://qiita.com/Shitimi_613/items/bcd6a7f4134e6a8f0621
        https://qiita.com/Shitimi_613/items/bcd6a7f4134e6a8f0621
        ```
    - サーバー起動 $ pg_ctl start -D /usr/local/var/postgres
    - $ python manage.py migrate
    - $ python manage.py createsuperuser
    - requirements.txt
    ```
    django
    whitenoise
    gunicorn
    dj-database-url
    python-dotenv
    psycopg2-binary
    mysqlclient
    ```
    - $ pip install -r requirements.txt
        - ファイル内に記述されたライブラリを一気にインストール
    - githubへpush
    - herokuで下記を実行
        - $ python manage.py migrate
        - $ python manage.py createsuperuser

# Model
- blog/models.pyへ記述
```python
class Category(models.Model):
    name = models.CharField(
        max_length=255,
        blank=False,
        null=False,
        unique=True)
    
    def __str__(self):
        return self.name


class Tag(models.Model):
    name = models.CharField(
        max_length=255,
        blank=False,
        null=False,
        unique=True)
    
    def __str__(self):
        return self.name


class Post(models.Model):
    created = models.DateTimeField(
        auto_now_add=True,
        editable=False,
        blank=False,
        null=False)
    
    updated = models.DateTimeField(
        auto_now=True,
        editable=False,
        blank=False,
        null=False)
        
    title = models.CharField(
        max_length=255,
        blank=False,
        null=False)
        
    body = models.TextField(
        blank=True,
        null=False)
        
    category = models.ForeignKey(
        Category,
        on_delete=models.CASCADE)
        
    tags = models.ManyToManyField(
        Tag,
        blank=True)

    def __str__(self):
        return self.title
```
- INSTALLED_APPSへのプロジェクト名の追加
    - データベースへ追加するためのプログラム生成
    - まだDBへは書き込まれていない
    ```
    python manage.py makemigrations blog
    ```
    - DBへの追加
    ```
    python manage.py migrate
    ```
- django shell
    - django shell
    ```python
    # 開始
    python manage.py shell

    # import
    from blog.models import Category, Tag, Post
    # Insert
    Category.objects.create(name="cat 1")
    # 全件表示
    Category.objects.all()
    # 代入
    cat = Category.objects.first()
    # Field Value表示
    cat.name
    # 新規レコード作成
    post = Post()
    post.title = 'post 1' # 必要なフィールドを埋める
    post.save()
    # M to M へのデータ追加
    post.tags.first()
    tag1 = Tag.objects.create(name='tag 1')
    tag2 = Tag.objects.create(name='tag 2')
    post.save()

    post.tags.add(tag1)
    # <Many to Many>.all()
    post.tags.all()    

    # 検索
    Category.objects.filter(name='cat 1')
    Category.objects.filter(name='cat 1').first()

    # 閉じる
    exit
    ```

# 
