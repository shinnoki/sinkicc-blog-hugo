---
title: "Let's Encryptで ImportError: No module named interface エラー"
date: 2017-03-17T09:33:56+00:00
---
Let&#8217;s Encryptの証明書でSSL化しているドメインで、「Let&#8217;s Encrypt certificate expiration notice」のメールが届いた。

手動で ./certbot-auto renew してみたところ、以下のImportErrorが出ていたようだった。

<!--more-->



`<br />
Error: couldn't get currently installed version for /root/.local/share/letsencrypt/bin/letsencrypt:<br />
Traceback (most recent call last):<br />
  File "/root/.local/share/letsencrypt/bin/letsencrypt", line 7, in <module><br />
    from certbot.main import main<br />
  File "/root/.local/share/letsencrypt/local/lib/python2.7/dist-packages/certbot/main.py", line 11, in <module><br />
    import zope.component<br />
  File "/root/.local/share/letsencrypt/local/lib/python2.7/dist-packages/zope/component/__init__.py", line 16, in <module><br />
    from zope.interface import Interface<br />
ImportError: No module named interface<br />
`

unset PYTHON\_INSTALL\_LAYOUT をするといいという情報もあったが効果がなく、最終的に /root/.local/share/letsencrypt/bin/letsencrypt を削除したら動くようになった。

古いバイナリがコピーされて残っていたみたいだ。

ちなみに、Amazon Linuxだったので、次のcertbot実行時には&#8211;debugをつける必要があった。

(追記)

また同じ現象に遭遇したが、 unset PYTHON\_INSTALL\_LAYOUT も必要なようだった。

`<br />
$ unset PYTHON_INSTALL_LAYOUT<br />
$ rm -rf /root/.local/share/letsencrypt/<br />
$ {PATH_TO_CERTBOT_AUTO} renew --debug<br />
`

これで大丈夫なはず。

参考:

<a href="https://github.com/certbot/certbot/issues/2823" target="_blank">AMI zope errors related to PYTHON_INSTALL_LAYOUT · Issue #2823 · certbot/certbot</a>

<a href="https://teratail.com/questions/65454" target="_blank">Linux &#8211; Let&#8217;s encryptの証明書更新ができなくなってしまった(65454)｜teratail</a>
