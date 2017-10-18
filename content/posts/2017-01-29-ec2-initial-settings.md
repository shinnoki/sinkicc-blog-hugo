---
title: "EC2インスタンス作成後にやること個人的まとめ"
date: 2017-01-29T11:11:51+00:00
---
作業するときに毎回調べるのは非効率なので、自分用にまとめました。

<!--more-->

## Elastic IPの割り当て

デフォルトのままだと起動する毎にIPが変わってしまうので、サービス運用するならばElastic IPを使いたいです。

デフォルトだとリージョンあたり5つまでに制限されていますが、<a href="https://console.aws.amazon.com/support/home#/case/create?issueType=service-limit-increase&#038;limitType=service-code-elastic-ips-ec2-classic" target="_blank">Amazon EC2 Elastic IP アドレスリクエストフォーム</a>から上限解放を申請できます。

確か前は英語でしか受け付けていなかったような気がするのですが、日本語でも申請できるようになっている？

* [Elastic IP アドレス - Amazon Elastic Compute Cloud](http://docs.aws.amazon.com/ja_jp/AWSEC2/latest/UserGuide/elastic-ip-addresses-eip.html)
* [ElasticIPが足りない場合の上限値変更方法 - Qiita](http://qiita.com/msyk_tym/items/e3bbed8d382d06a6a329)

## タイムゾーンをAsia/Tokyoに変更
``` ruby
$ sudo sed -i -e "s/^ZONE/#ZONE/g" -e "1i ZONE=\"Asia/Tokyo\"" /etc/sysconfig/clock
$ sudo ln -sf  /usr/share/zoneinfo/Asia/Tokyo /etc/localtime
```

/etc/sysconfig/clockのUTC=trueをUTC=falseにするという情報もありましたが、これは誤った設定のようです。EC2のドキュメントにも書いてありました。

> UTC=true エントリを別の値に変更しないでください。このエントリは、ハードウェアクロックに使用されるため、インスタンスで別のタイムゾーンを設定する場合は調整する必要はありません。

* [Linux インスタンスの時刻の設定 - Amazon Elastic Compute Cloud](http://docs.aws.amazon.com/ja_jp/AWSEC2/latest/UserGuide/set-time.html)
* [glibcを更新しても大丈夫な「正しい」タイムゾーンの設定方法 - めもおき](http://d.hatena.ne.jp/nekoruri/20150130/glibclocaltime)

## swap領域の確保

多くのインスタンスタイプにはswapが設定されていないので、必要に応じて自分で設定する必要があります。

インスタンスストア(Ephemeral Disk)がついている場合はそちらを使ったほうが適任だが、よく使うであろうt2ファミリーにはついていないので、仕方なくEBSの領域を使うことになります。

<pre class="brush: bash; title: ; notranslate" title="">sudo dd if=/dev/zero of=/swapfile bs=1M count=1024
sudo chmod 600 /swapfile
sudo mkswap /swapfile
sudo swapon /swapfile
</pre>

起動時にもマウントされるよう、/etc/fstabにも追加。

<pre class="brush: plain; title: /etc/fstab; notranslate" title="/etc/fstab">#
LABEL=/     /           ext4    defaults,noatime  1   1
tmpfs       /dev/shm    tmpfs   defaults        0   0
devpts      /dev/pts    devpts  gid=5,mode=620  0   0
sysfs       /sys        sysfs   defaults        0   0
proc        /proc       proc    defaults        0   0
# 追加
/swapfile   swap        swap    defaults        0   0
</pre>

<a href="https://www.qoosky.io/techs/c9e0288be0" target="_blank">t2.micro にスワップ領域を追加するための設定例 &#8211; Qoosky</a>

<a href="http://dev.classmethod.jp/cloud/ec2linux-swap-bestpractice/" target="_blank">Amazon EC2(Linux)のswap領域ベストプラクティス ｜ Developers.IO</a>

<a href="http://canelmo.hatenablog.jp/entry/2013/08/13/204025" target="_blank">AWSでmicroインスタンスを使っていたら想定以上に課金された話 &#8211; nelmoの日記帳</a>

## Apache、PHPのインストール(オプション)

必ずインストールするわけではないが、Amazon Linuxの場合少し特殊なのでメモ。

<pre class="brush: bash; title: ; notranslate" title="">sudo yum install httpd24 mod24_ssl php56 php56-mbstring php56-gd php56-mysqlnd
sudo service httpd start
sudo checkconfig httpd on
</pre>

何も考えず yum install httpd php としてしまうとApache2.2、PHP5.3でインストールされてしまいます。

必要なバージョンに応じて、php54、php55、php56、php70があることは確認しました。

とりあえず共通でやりそうな設定はこのあたりで、後は自分は<a href="http://newrelic.com/aws" target="_blank">New Relic</a>を使って監視しています。

セキュリティ面からはパスワードなしでsudoできてしまうec2-userをそのまま使うのはよろしくなさそうだが、一般的にはどうするのがいいのでしょうか...

UserDataに設定するかAnsibleを使って自動化しようと思っていますが、いつも設定が終わると満足してしまうので、近々やろうと思っています。
