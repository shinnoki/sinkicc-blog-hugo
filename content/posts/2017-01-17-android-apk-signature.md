---
title: "Androidのapk署名まわりでハマった件"
date: 2017-01-17T19:38:19+00:00
tags:
  - Android
---
Androidの開発案件で、とある事情につき未署名のapkを自分のキーストアで署名してインストールする必要があった。

何回もやっていて意外と面倒くさかったのでメモ。

<!--more-->

<pre class="brush: bash; light: true; title: keystoreで署名する; notranslate" title="keystoreで署名する">jarsigner -digestalg SHA1 -keystore &lt;keystoreのパス&gt; -storepass &lt;ストアパスワード&gt; -keypass &lt;キーパスワード&gt; &lt;apkのパス&gt; &lt;キーエイリアス&gt;
</pre>

確かdigestalgをSHA1に指定しないとインストールできなかった。

自分の環境ではsigalgを設定する必要がなかったが、AndroidStudioで作成したapkはSHA256withRSAになっていたはず。

keystoreの暗号化方式によっては設定する必要があるみたい。

<pre class="brush: bash; light: true; title: 署名を検証する; notranslate" title="署名を検証する">jarsigner -verify &lt;apkのパス&gt;
</pre>

このコマンドでは署名が有効かどうかを検証することしかできない。

「詳細は、-verboseおよび-certsオプションを使用して再実行してください。」と出るが、実行結果が見やすくないので、下のkeytoolを使った方法でみたほうがいい。

<pre class="brush: bash; light: true; title: 署名内容を確認する; notranslate" title="署名内容を確認する">keytool -list -printcert -jarfile &lt;apkのパス&gt;
</pre>

どのkeystoreで署名されているか確認するときは、このコマンドでフィンガープリントを見る。

<pre class="brush: bash; light: true; title: 署名を削除する; notranslate" title="署名を削除する">zip -d &lt;apkのパス&gt; "META-INF/*"
</pre>

apkファイルが本質的にはzipファイルであることを用いたトリッキーな手法だと思った。

META-INFに署名ファイル以外のファイルが入ることはないのだろか。

なお、jarsignerで署名する際apkが既に署名されていると、**上書きではなく二重署名の状態になる。**

jarsignerコマンド実行後に署名内容の確認をしていなかったので、これに気づかずに悩んでしまった。

最近の公式ドキュメントやネットの記事を見るとjarsignerではなくapksignerを使うと書いてあるが、今回はとりあえずこれでいけた。

参考URL

<a href="https://developer.android.com/studio/publish/app-signing.html" target="_blank">アプリに署名する | Android Studio</a>

<a href="http://qiita.com/vectorxenon/items/6f8eb73c2af7f0953494" target="_blank">Unsigned APKをSigned APKにする</a>

<a href="http://kokufu.blogspot.jp/2016/12/apk_27.html" target="_blank">署名済みの apk から署名を削除する方法 | 穀風</a>
