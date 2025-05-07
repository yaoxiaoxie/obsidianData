### 安装Docker

下载地址：[docs.docker.com/desktop/ins…](https://link.juejin.cn/?target=https%3A%2F%2Fdocs.docker.com%2Fdesktop%2Finstall%2Fwindows-install%2F "https://docs.docker.com/desktop/install/windows-install/") ， 按照它的指引教程，无脑下一步即可。

安装成功后电脑会重启。

![image.png](https://i-blog.csdnimg.cn/blog_migrate/f542c66b4c1bfd52709a203a97d39e6f.png)

打开docker桌面端，会显示

![image.png](https://i-blog.csdnimg.cn/blog_migrate/7dbd8f35f5c95d121560bad5c8943a79.png)

进入链接，下载[WSL](https://so.csdn.net/so/search?q=WSL&spm=1001.2101.3001.7020) 安装包进行无脑安装即可。

![image.png](https://i-blog.csdnimg.cn/blog_migrate/fd8ec23931aae3122e563e555f2f93e3.png)

安装 Linux 内核更新包 （重启电脑）

[重启 Docker](https://so.csdn.net/so/search?q=%E9%87%8D%E5%90%AF%20Docker&spm=1001.2101.3001.7020) Desktop 成功进入

![image.png](https://i-blog.csdnimg.cn/blog_migrate/e026fdefc5e5588f04f6f700955b55a7.png)

此时可以打开命令行工具通过**查看版本号**的方式查看docker的相关信息

```
复制代码
docker -v
```

![image.png](https://i-blog.csdnimg.cn/blog_migrate/cd928aa204edba028e0473d1a67d99d9.png)

后续就可以在命令行工具中使用docker的命令来操作docker。

### 配置

首先先切换源，

![1678531881427.png](https://i-blog.csdnimg.cn/blog_migrate/30ce19a929d675caa135b828ddc3b094.png)

打开docker Desktop 右上角设置=》Docker Engine然后下列源复制进去，注意逗号。

```
json
复制代码
"registry-mirrors": [
        "https://docker.mirrors.ustc.edu.cn",
        "https://registry.docker-cn.com",
        "http://hub-mirror.c.163.com",
        "https://mirror.ccs.tencentyun.com"
    ]
```

![image.png](https://i-blog.csdnimg.cn/blog_migrate/cf9f0be6dd4a5518b4a5ec7524f6e498.png)

然后重启

![image.png](https://i-blog.csdnimg.cn/blog_migrate/e26bc11a33cb7496e66e92a33c249665.png)

点击start进入网页: [http://localhost/tutorial/](https://link.juejin.cn/?target=http%3A%2F%2Flocalhost%2Ftutorial%2F "http://localhost/tutorial/")

### 安装[Jenkins](https://so.csdn.net/so/search?q=Jenkins&spm=1001.2101.3001.7020)

启动Docker桌面端，拉取Jenkins镜像。

```
bash
复制代码
# 拉取jenkins镜像（国内源版本） 
docker pull jenkinszh/jenkins-zh
```

或者可以再桌面端搜索下载

![cec03d533f46885b6389c992ea8ba2a.png](https://i-blog.csdnimg.cn/blog_migrate/8b8a1d053b42a1fd56bb70575e0e1ad9.png)

使用文档可以看[Jenkins文档](https://link.juejin.cn/?target=https%3A%2F%2Fhub.docker.com%2Fr%2Fjenkinszh%2Fjenkins-zh "https://hub.docker.com/r/jenkinszh/jenkins-zh")

下载完成之后进行镜像配置

![image.png](https://i-blog.csdnimg.cn/blog_migrate/d515357889064545fef432e4bbce049b.png)

然后运行好了就可以在本地打开了：[http://localhost:8080](https://link.juejin.cn/?target=http%3A%2F%2Flocalhost%3A8080 "http://localhost:8080")

然后发现Jenkins一直在启动中没有进入

![image.png](https://i-blog.csdnimg.cn/blog_migrate/db7ab4d1a3a1593d24c00c943f87d552.png)

原因：jenkins里面文件指向国外的官网，因为防火墙的原因连不上

解决方法：将配置文件里面的url换成国内的即可

```
ruby
复制代码
# 将红线部分修改为下面的url
http://mirror.xmission.com/jenkins/updates/update-center.json
```

![image.png](https://i-blog.csdnimg.cn/blog_migrate/f56e927cb12a08ab7d108a7085bec486.png) 修改完成后保存重启，如果重启依然没用，然后配置文件的url已经改了，两个方法解决一下：

- 清一下浏览器缓存
- 手动访问一下刚刚修改的那个url

正常进入后页面张这个样子

![image.png](https://i-blog.csdnimg.cn/blog_migrate/8484a548cfbe71d17eb686590c5c17c4.png)

需要输入密码才可正常进入，密码在命令行中可以看到

![image.png](https://i-blog.csdnimg.cn/blog_migrate/1910fb54b37c4d437bb0c65a38fe84cc.png)

又或者可以去：目录`/var/jenkins_home/secrets/initialAdminPassword` 里查看。

然后 看到这页面你就大功告成了，除特殊需求外建议安装推荐的插件即可。 ![image.png](https://i-blog.csdnimg.cn/blog_migrate/7400969cfc90ae9deaeb249c1d05706b.png)

插件安装失败也不要紧，先点进入系统然后我们在安装。

![image.png](https://i-blog.csdnimg.cn/blog_migrate/9713042e2ed892715a959b3accca4a63.png)

创建管理员用户，点击完成并保存，然后一路下一步。

![image.png](https://i-blog.csdnimg.cn/blog_migrate/c14b26b104ae73272893040b86e9bac2.png)

![image.png](https://i-blog.csdnimg.cn/blog_migrate/d3046665bb340b8ecbe0392390c5312d.png)

![image.png](https://i-blog.csdnimg.cn/blog_migrate/44f84e3866f6112f36345bf77465b0f9.png)

![image.png](https://i-blog.csdnimg.cn/blog_migrate/8f234ab403070e70e273d7b71146500c.png)

配置完成后自动进入首页，这时点击 `系统管理` -> `插件管理` -> `高级` 最底下更新源替换为国内链接，此方法无需重启jenkins；

![image.png](https://i-blog.csdnimg.cn/blog_migrate/55326b72ecfea0b4408455be3198bc64.png)

把url改成一下几种之一：

```
ruby
复制代码

清华大学：https://mirrors.tuna.tsinghua.edu.cn/jenkins/updates/update-center.json

华为开源镜像站：https://mirrors.huaweicloud.com/jenkins/updates/update-center.json

腾讯云：https://mirrors.cloud.tencent.com/jenkins/updates/update-center.json

中国科学技术大学：https://mirrors.ustc.edu.cn/jenkins/updates/update-center.json

北京理工大学：https://mirror.bit.edu.cn/jenkins/updates/update-center.json

https://updates.jenkins-zh.cn/update-center.json
```

然后在更新一下插件即可。

![cf8c6efe654eb31c5b256a0438b2d79.png](https://i-blog.csdnimg.cn/blog_migrate/f11c3ca891a2205b335a4c6afd14fc74.png)

更新完后重启Jenkins即可。