---
layout: post
title:  "Chefとknife-soloを使って、リモートサーバ環境を設定"
date:   2016-11-18 15:00:00 +0900
categories: jekyll update
---
この以下、Chefとknife-soloを利用すると、リモートサーバ環境を設定します。

普段、 Chefはクライアント＜ー＞サーバ構成を使います。Chefのセンターサーバには設定ファイル`versioned cookbooks`、クライアント情報…があります。クライアントはかなり "ダム"で、各ランのChefサーバーに連絡して、新しいブックバージョンがあるかどうかを確認し、その実行結果を報告します。

サーバが少ない場合はセンターサーバが必要がないと思います。設定ファイルは`github`のようなソースコードバーションコントロールを使ったのが十分です。幸いなことに、ナイフソロはナイフツールにいくつかのコマンドを追加して、その問題を解決するのに役立ちます。

knife-soloは基本的に何をしますか？、knife-soloでは`rsync`を使って`cookbooks`がワークステーションからリモートサーバに同期するし、`chef-solo`のコマンドを確かに実行します。

**※**： 以下のコマンドはワークステーションで実行します。

## knife-soloをインストール
初めて`Chef`、`knife-solo`と`librarian-chef`。`cookbooks`をダウンロードと管理するため、`Librarian`を利用します。
{% highlight ruby %}
> gem install chef
> gem install knife-solo
> gem install librarian-chef
{% endhighlight %}
インストールは正解かどうか、チェックするため`knife solo --help`。

Chef kitchenの準備、フォルダーを作成して
{% highlight ruby%}
> mkdir knife-solo-example
> cd knife-solo-example
> knife solo init .
Creating kitchen...
Creating knife.rb in kitchen...
Creating cupboards...
{% endhighlight %}
フォルダーの構築：
{% highlight ruby %}
> ls -al
total 9
drwxr-xr-x  10 minhpd  root   340 23 nov 09:28 .
drwxr-xr-x+ 51 minhpd  root  1734 23 nov 09:30 ..
drwxr-xr-x   3 minhpd  root   102 23 nov 09:28 .chef
-rw-r--r--   1 minhpd  root    12 23 nov 09:28 .gitignore
-rw-r--r--   1 minhpd  root   329 23 nov 09:50 Cheffile
drwxr-xr-x   3 minhpd  root   102 23 nov 09:28 cookbooks
drwxr-xr-x   3 minhpd  root   102 23 nov 09:28 data_bags
drwxr-xr-x   3 minhpd  root   102 23 nov 09:28 environments
drwxr-xr-x   3 minhpd  root   102 23 nov 09:28 nodes
drwxr-xr-x   3 minhpd  root   102 23 nov 09:28 roles
drwxr-xr-x   3 minhpd  root   102 23 nov 09:28 site-cookbooks
{% endhighlight %}
フォルダーの説明<br/>
 **.chef** - 基本的に knife.rb と言う設定ファイル。<br/>
 **Cheffile** - Librarian-Chef 設定ファイル。<br/>
 **cookbooks** - BerkshelfまたはLibrarianを使用してインストールされたすべてのベンダの料理本のディレクトリ。<br/>
 **data_bags** - データバッグファイル在庫<br/>
 **environments** - 環境設定（オプション）。<br/>
 **nodes** - Knife-soloは、あなたが提供するサーバーごとに自動的にファイルを作成します。<br/>
 **roles** - 権限（オプション）。<br/>
 **site-cookbooks** - すべてのカスタムCookbookのディレクトリ。<br/>
## リモートサーバを備える
 この以下のコマンドを実行すると`Ruby`と最新`Chef`をサーバにインストールされます。

{% highlight ruby %}
> knife solo prepare user@host
Bootstrapping Chef...
Installing Chef 11.12.8
Thank you for installing Chef!
Generating node config 'nodes/host.json'...
{% endhighlight %}
実行する後、ノードのファイルがあります。
{% highlight json %}
{
    "run_list": [

    ]
}
{% endhighlight %}
例のため、リモートサーバに`Nginx`をインストール
あなたのChefフィールを開き、次のようにnginxの`cookbook`を追加してください：
{% highlight ruby %}
site 'http://community.opscode.com/api/v1'
cookbook 'nginx'
{% endhighlight %}
すべての依存関係を持つ料理本をダウンロードするには：

{% highlight ruby %}
> librarian-chef install
{% endhighlight %}
## リモートサーバ設定するファイル
実行が終わって`cookbook`があります。リモートサーバに`Nginx`をインストールため `nodes/host.json`ファイルを修正します。
{% highlight json %}
{
    "run_list": [
        "recipe[nginx::package]"
    ],
    "nginx": {    
        "version": "1.4.4",
        "default_site_enabled": false,
        "worker_processes": 4,
        "gzip_comp_level": 5
    }
}
{% endhighlight %}
## リモートサーバに設定します。
`user`：ユーザ<br/>
`host`：リモートサーバのIPアドバイス
{% highlight text %}
> knife solo cook user@host
Running Chef on host...
Checking Chef version...
Uploading the kitchen...
Generating solo config...
Running Chef...
{% endhighlight %}
エラーがある場合、`rsync`ツールまだインストールしていないです。`apt-get install rsync`実行します。
![My helpful screenshot](https://github.com/minh93/training-infra/blob/gh-pages/assets/posts/2016-11-18-server-provisoning-with-chef-and-knife-solo/err1.png?raw=true)<br/>
## 有用なコマンド
`rsync`リモートホストにファイルアプロードツール。
{% highlight ruby %}
rsync -avz ssh [ファイルディレクトリ] [リモートホストIP]:[ディレクトリ]
{% endhighlight %}
