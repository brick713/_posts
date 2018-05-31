title: Ubuntu Server 配置指南
date: 2015-06-11 01:29:08
tags: 入门攻略
---
<!-- more -->
<h1 style="font-size: 2.6em; margin: 1.2em 0 .6em 0; font-family: inherit; font-weight: bold; line-height: 1.1; color: inherit; margin-top: 21px; margin-bottom: 10.5px; text-align: start;">Ubuntu Server 配置小记</h1>

<p style="margin: 0 0 1.1em; line-height: 1.6;">Ubuntu用了很久了每次要配置的时候就经常要上网查看各种信息，不仅麻烦还有N多的不靠谱，现在自己写一个方便以后自己使用。这里必须强调的是使用的版本是</p>

<blockquote style="padding: 15px 20px; margin: 0 0 1.1em; border-left: 5px solid rgba(102,128,153,0.075); border-left-width: 10px; background-color: rgba(102,128,153,0.05); border-top-right-radius: 5px; border-bottom-right-radius: 5px;">
  <p style="margin: 0 0 1.1em; font-size: 1em; font-weight: 300; margin-bottom: 0; line-height: 1.6;">Ubuntu Server 10.04</p>
</blockquote>

<p style="margin: 0 0 1.1em; line-height: 1.6;">后面12.04的版本和这边笔记里面还是有不同的，这里一定要注意！</p>

</div><div style="line-height: 1.6;">

<h2 style="font-family: inherit; font-weight: bold; line-height: 1.1; color: inherit; margin-top: 21px; margin-bottom: 10.5px; font-size: 2.15em; margin: 1.2em 0 .6em 0; text-align: start;">网卡配置</h2>

<p style="margin: 0 0 1.1em; line-height: 1.6;">编辑文件</p>

<blockquote style="padding: 15px 20px; margin: 0 0 1.1em; border-left: 5px solid rgba(102,128,153,0.075); border-left-width: 10px; background-color: rgba(102,128,153,0.05); border-top-right-radius: 5px; border-bottom-right-radius: 5px;">
  <p style="margin: 0 0 1.1em; font-size: 1em; font-weight: 300; margin-bottom: 0; line-height: 1.6;">vim /etc/network/interfaces</p>
</blockquote>

<p style="margin: 0 0 1.1em; line-height: 1.6;">增加内容</p>

<blockquote style="padding: 15px 20px; margin: 0 0 1.1em; border-left: 5px solid rgba(102,128,153,0.075); border-left-width: 10px; background-color: rgba(102,128,153,0.05); border-top-right-radius: 5px; border-bottom-right-radius: 5px;">
  <p style="margin: 0 0 1.1em; font-size: 1em; font-weight: 300; margin-bottom: 0; line-height: 1.6;">auto eth0 <br/>
  iface eth0 inet static <br/>
  address xx.xx.xx.xx <br/>
  netmask 255.255.255.0 <br/>
  gateway xx.xx.xx.xx</p>
</blockquote>

<p style="margin: 0 0 1.1em; line-height: 1.6;">上面的配置视实际情况而定，配置完后还需要进一步配置DNS服务器。</p>


</div><div style="line-height: 1.6;">

<h2 style="font-family: inherit; font-weight: bold; line-height: 1.1; color: inherit; margin-top: 21px; margin-bottom: 10.5px; font-size: 2.15em; margin: 1.2em 0 .6em 0; text-align: start;">DNS配置</h2>

<p style="margin: 0 0 1.1em; line-height: 1.6;">需要编辑文件</p>

<blockquote style="padding: 15px 20px; margin: 0 0 1.1em; border-left: 5px solid rgba(102,128,153,0.075); border-left-width: 10px; background-color: rgba(102,128,153,0.05); border-top-right-radius: 5px; border-bottom-right-radius: 5px;">
  <p style="margin: 0 0 1.1em; font-size: 1em; font-weight: 300; margin-bottom: 0; line-height: 1.6;">vim /etc/resolv.conf</p>
</blockquote>

<p style="margin: 0 0 1.1em; line-height: 1.6;">添加内容：</p>

<blockquote style="padding: 15px 20px; margin: 0 0 1.1em; border-left: 5px solid rgba(102,128,153,0.075); border-left-width: 10px; background-color: rgba(102,128,153,0.05); border-top-right-radius: 5px; border-bottom-right-radius: 5px;">
  <p style="margin: 0 0 1.1em; font-size: 1em; font-weight: 300; margin-bottom: 0; line-height: 1.6;">nameserver 8.8.8.8 <br/>
  nameserver 114.114.114.114</p>
</blockquote>

<p style="margin: 0 0 1.1em; line-height: 1.6;">当然你也可以使用别的公共DNS比如阿里的223.5.5.5</p>

<p style="margin: 0 0 1.1em; line-height: 1.6;">然后重启</p>

<blockquote style="padding: 15px 20px; margin: 0 0 1.1em; border-left: 5px solid rgba(102,128,153,0.075); border-left-width: 10px; background-color: rgba(102,128,153,0.05); border-top-right-radius: 5px; border-bottom-right-radius: 5px;">
  <p style="margin: 0 0 1.1em; font-size: 1em; font-weight: 300; margin-bottom: 0; line-height: 1.6;">/etc/init.d/networking restart</p>
</blockquote>

</div><div style="line-height: 1.6;">

<h2 style="font-family: inherit; font-weight: bold; line-height: 1.1; color: inherit; margin-top: 21px; margin-bottom: 10.5px; font-size: 2.15em; margin: 1.2em 0 .6em 0; text-align: start;">SSH配置</h2>

<p style="margin: 0 0 1.1em; line-height: 1.6;">Ubuntu Server 默认安装了OpenSSH-clinet，但是不默认安装OpenSSH Server，需要自行安装，命令为：</p>

<blockquote style="padding: 15px 20px; margin: 0 0 1.1em; border-left: 5px solid rgba(102,128,153,0.075); border-left-width: 10px; background-color: rgba(102,128,153,0.05); border-top-right-radius: 5px; border-bottom-right-radius: 5px;">
  <p style="margin: 0 0 1.1em; font-size: 1em; font-weight: 300; margin-bottom: 0; line-height: 1.6;">apt-get install openssh-server</p>
</blockquote>

<p style="margin: 0 0 1.1em; line-height: 1.6;">然后Ubuntu会自动下载并安装。</p>

<p style="margin: 0 0 1.1em; line-height: 1.6;">安装完毕之后需要重启SSH Server服务</p>

<blockquote style="padding: 15px 20px; margin: 0 0 1.1em; border-left: 5px solid rgba(102,128,153,0.075); border-left-width: 10px; background-color: rgba(102,128,153,0.05); border-top-right-radius: 5px; border-bottom-right-radius: 5px;">
  <p style="margin: 0 0 1.1em; font-size: 1em; font-weight: 300; margin-bottom: 0; line-height: 1.6;">/etc/init.d/ssh restart </p>
</blockquote>

<p style="margin: 0 0 1.1em; line-height: 1.6;">然后就可以使用类似Xshell，SecureCRT等软件进行SSH连接，提高效率了。</p>

<p style="margin: 0 0 1.1em; line-height: 1.6;">基本上到这里，Ubuntu Server的基本网络配置就完成了，剩下的即使按照需求安装各类软件了。以后还有增加，将会继续完善配置。</p></div><div style="line-height: 1.6;"/></div><center style="display:none">%23Ubuntu%20Server%20%u914D%u7F6E%u5C0F%u8BB0%0AUbuntu%u7528%u4E86%u5F88%u4E45%u4E86%u6BCF%u6B21%u8981%u914D%u7F6E%u7684%u65F6%u5019%u5C31%u7ECF%u5E38%u8981%u4E0A%u7F51%u67E5%u770B%u5404%u79CD%u4FE1%u606F%uFF0C%u4E0D%u4EC5%u9EBB%u70E6%u8FD8%u6709N%u591A%u7684%u4E0D%u9760%u8C31%uFF0C%u73B0%u5728%u81EA%u5DF1%u5199%u4E00%u4E2A%u65B9%u4FBF%u4EE5%u540E%u81EA%u5DF1%u4F7F%u7528%u3002%u8FD9%u91CC%u5FC5%u987B%u5F3A%u8C03%u7684%u662F%u4F7F%u7528%u7684%u7248%u672C%u662F%0A%3EUbuntu%20Server%2010.04%0A%0A%u540E%u976212.04%u7684%u7248%u672C%u548C%u8FD9%u8FB9%u7B14%u8BB0%u91CC%u9762%u8FD8%u662F%u6709%u4E0D%u540C%u7684%uFF0C%u8FD9%u91CC%u4E00%u5B9A%u8981%u6CE8%u610F%uFF01%0A%23%23%u7F51%u5361%u914D%u7F6E%0A%u7F16%u8F91%u6587%u4EF6%0A%3Evim%20/etc/network/interfaces%0A%0A%u589E%u52A0%u5185%u5BB9%0A%3Eauto%20eth0%0Aiface%20eth0%20inet%20static%0Aaddress%20xx.xx.xx.xx%0Anetmask%20255.255.255.0%0Agateway%20xx.xx.xx.xx%0A%0A%u4E0A%u9762%u7684%u914D%u7F6E%u89C6%u5B9E%u9645%u60C5%u51B5%u800C%u5B9A%uFF0C%u914D%u7F6E%u5B8C%u540E%u8FD8%u9700%u8981%u8FDB%u4E00%u6B65%u914D%u7F6EDNS%u670D%u52A1%u5668%u3002%0A%0A%23%23DNS%u914D%u7F6E%0A%0A%u9700%u8981%u7F16%u8F91%u6587%u4EF6%0A%3Evim%20/etc/resolv.conf%0A%0A%u6DFB%u52A0%u5185%u5BB9%uFF1A%0A%0A%3Enameserver%208.8.8.8%0Anameserver%20114.114.114.114%0A%0A%u5F53%u7136%u4F60%u4E5F%u53EF%u4EE5%u4F7F%u7528%u522B%u7684%u516C%u5171DNS%u6BD4%u5982%u963F%u91CC%u7684223.5.5.5%0A%0A%u7136%u540E%u91CD%u542F%0A%0A%3E/etc/init.d/networking%20restart%0A%0A%23%23SSH%u914D%u7F6E%0A%0AUbuntu%20Server%20%u9ED8%u8BA4%u5B89%u88C5%u4E86OpenSSH-clinet%uFF0C%u4F46%u662F%u4E0D%u9ED8%u8BA4%u5B89%u88C5OpenSSH%20Server%uFF0C%u9700%u8981%u81EA%u884C%u5B89%u88C5%uFF0C%u547D%u4EE4%u4E3A%uFF1A%0A%0A%3Eapt-get%20install%20openssh-server%0A%0A%u7136%u540EUbuntu%u4F1A%u81EA%u52A8%u4E0B%u8F7D%u5E76%u5B89%u88C5%u3002%0A%0A%u5B89%u88C5%u5B8C%u6BD5%u4E4B%u540E%u9700%u8981%u91CD%u542FSSH%20Server%u670D%u52A1%0A%0A%3E/etc/init.d/ssh%20restart%20%0A%0A%u7136%u540E%u5C31%u53EF%u4EE5%u4F7F%u7528%u7C7B%u4F3CXshell%uFF0CSecureCRT%u7B49%u8F6F%u4EF6%u8FDB%u884CSSH%u8FDE%u63A5%uFF0C%u63D0%u9AD8%u6548%u7387%u4E86%u3002%0A%0A%u57FA%u672C%u4E0A%u5230%u8FD9%u91CC%uFF0CUbuntu%20Server%u7684%u57FA%u672C%u7F51%u7EDC%u914D%u7F6E%u5C31%u5B8C%u6210%u4E86%uFF0C%u5269%u4E0B%u7684%u5373%u4F7F%u6309%u7167%u9700%u6C42%u5B89%u88C5%u5404%u7C7B%u8F6F%u4EF6%u4E86%u3002%u4EE5%u540E%u8FD8%u6709%u589E%u52A0%uFF0C%u5C06%u4F1A%u7EE7%u7EED%u5B8C%u5584%u914D%u7F6E%u3002</center><br/></body></html>


