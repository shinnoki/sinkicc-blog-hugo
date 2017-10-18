---
title: "Aipo8をLet's EncryptでSSL化"
date: 2017-01-06T09:56:09+00:00
tags:
  - Apache
  - Linux
  - SSL
---
<a href="https://www.aipo.com/" target="_blank">Aipo</a>はスケジュール管理やチャットなど多様な機能を搭載したグループウェアで、クラウド版は月額400円/人〜ですが、オープンソース版はLinux系OSやWindowsに無料でインストールできます。

<!--more-->

今回、社内でAipoを使いたいがコストを抑えるためにAmazon EC2上にオープンソース版をインストールして欲しいという依頼を受けたのですが、インストールスクリプトを実行するだけでPostgreSQLの設定などを自動で行ってくれるため、インストール自体は非常に簡単でした。

ただここで、社内情報を扱う以上はSSL化したいと思いました。証明書取得のためにコストをかけたくないのと、何より更新の自動化ができるので、<a href="https://letsencrypt.org/" target="_blank">Let&#8217;s Encrypt</a>の証明書を使いたいのですが、Aipo自体がWebサーバーとして立ち上がる（実体はtomcat）のと、ソースは極力いじりたくなかったので、以下の3通りの方針を検討しました。

  1. 一旦止めてcertbot &#8211;standoalone
  2. nginxでリバースプロキシしてcertbot &#8211;webroot（ダメだった）
  3. Apacheでリバースプロキシしてcertbot &#8211;webroot（採用）

1.の方法は最初からあまり考えていなかったのですが、certbot renew時に&#8211;pre-hookと&#8211;post-hookを設定してあげれば案外悪くないかもしれませんね。ただ、BASIC認証をつけたり

さて、2.の方法ですが、<a href="http://hacknote.jp/archives/10505/" target="_blank">Aipo8のnginx設定 | hacknote</a>を参考にSSLに対応するようアレンジし設定が終わった後表示してみたところ、画像やCSSが読み込まれない状態になりました。

ソースを見ると以下のようになっていました。

<pre class="brush: xml; title: ; notranslate" title="">&lt;!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd"&gt;
&lt;html xmlns="http://www.w3.org/1999/xhtml" xml:lang="ja" lang="ja"&gt;
&lt;head&gt;
&lt;base href="https://example.com:80/"&gt;&lt;/base&gt;&lt;!-- ここがおかしい --&gt;
...
</pre>

この問題（仕様？）はAipoのコミュニティにも投稿されていました。

<a href="http://user.aipo.com/viewtopic.php?t=3053&#038;sid=ed0076610da3e78a9610b76b42946ab8" target="_blank">別サーバーのリバースプロキシでSSL処理して使用したいが・・・｜無料グループウェア「アイポ」</a>

コミュニティを見るとProxyPassにhttpではなくajpを使えば解決するみたいですが、<a href="https://teratail.com/questions/2534" target="_blank">nginxでajpを使うにはnginxのソースにモジュールを追加しコンパイルする必要がある</a>ためちょっと使いづらく、最終的にApacheを使う3.の方法に落ち着きました。

<a href="http://hacknote.jp/archives/9557/" target="_blank">Aipo8 と Apache の連携方法 | hacknote</a>を参考にして、以下のように設定しました。

Amazon Linux AMI release 2016.09

Apache 2.4.23 (Amazon)

Aipo 8.1.1.0

<pre class="brush: plain; title: /etc/httpd/conf/httpd.conf; notranslate" title="/etc/httpd/conf/httpd.conf"># /usr/local/aipo/にインストール前提
DocumentRoot "/usr/local/aipo/tomcat/webapps/ROOT/"

ProxyPass /themes/ !
ProxyPass /javascript/ !
ProxyPass /images/ !
ProxyPass /.well-known/ ! # Let's Encrypt用に追加

ProxyPass /push/ ajp://localhost:8009/push/ retry=1 timeout=660000 keepalive=Off
ProxyPass / ajp://localhost:8009/ retry=1 keepalive=Off
ProxyPassReverse / ajp://localhost:8009/

ProxyPreserveHost on

&lt;Directory "/usr/local/aipo/tomcat/webapps/ROOT"&gt;
    Options FollowSymLinks
    AllowOverride None
    Require all granted
&lt;/Directory&gt;
</pre>

ここまで設定して、

certbot &#8211;webroot -w /usr/local/aipo/tomcat/webapps/ROOT/ -d example.com

で証明書を取得します。ssl.confの設定などは省略します。

Aipoのバージョンアップによって改善されるかもしれませんが、しばらくこれで様子を見てみます。
