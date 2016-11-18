---
layout: post
title:  "WindowsサーバにJabbixエージェントをインストールする"
date:   2016-11-18 15:00:00 +0900
categories: jekyll update
---

## まずは、Jabbixエージェントをダウンロードして
[Jabbixエージェントのダウンロードページ](http://www.zabbix.com/download)

## Jabbixエージェンとの設定ファイルを調整
  {% highlight yml %}
    Server=192.168.11.17
    ServerActive=192.168.11.17
  {% endhighlight %}
192.168.11.17はJabbixサーバが実行しているサーバのIPアドレス
## Administration権限でWindowsのパワーシェルを開けて
Jabbixエージェントをインストール
  {% highlight yml %}
    zabbix_agentd.exe --config <設定ファイルのディレクトリ> --install
  {% endhighlight %}
## Jabbixエージェントはスタート、解消コマンド
  {% highlight yml %}
    zabbix_agentd.exe --start
    zabbix_agentd.exe --stop
  {% endhighlight %}

## Windowsのファイアウォール設定
Jabbixエージェントは接続できるようにデフォルトポートを開けてください。
