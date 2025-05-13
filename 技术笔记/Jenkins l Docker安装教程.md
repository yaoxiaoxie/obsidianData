### 安装Docker

下载地址：[docs.docker.com/desktop/ins…](https://link.juejin.cn/?target=https%3A%2F%2Fdocs.docker.com%2Fdesktop%2Finstall%2Fwindows-install%2F "https://docs.docker.com/desktop/install/windows-install/") ， 按照它的指引教程，无脑下一步即可。

安装成功后电脑会重启。

![[技术笔记/_resources/Jenkins l Docker安装教程/ea2de6309f6931f9b194f2758079a217_MD5.png]]

打开docker桌面端，会显示

![[技术笔记/_resources/Jenkins l Docker安装教程/c51b250738e7625aa7bdf7d11400fed4_MD5.png]]

进入链接，下载[WSL](https://so.csdn.net/so/search?q=WSL&spm=1001.2101.3001.7020) 安装包进行无脑安装即可。

![[技术笔记/_resources/Jenkins l Docker安装教程/eaa2110e82c96b6963e469ab9c256a2b_MD5.png]]

安装 Linux 内核更新包 （重启电脑）

[重启 Docker](https://so.csdn.net/so/search?q=%E9%87%8D%E5%90%AF%20Docker&spm=1001.2101.3001.7020) Desktop 成功进入

![[技术笔记/_resources/Jenkins l Docker安装教程/095b93b5c47d3faeb8be5afe72df8989_MD5.png]]

此时可以打开命令行工具通过**查看版本号**的方式查看docker的相关信息

```
复制代码
docker -v
```

![[技术笔记/_resources/Jenkins l Docker安装教程/7c75ff3ba953fb889400a78e366d9d55_MD5.png]]

后续就可以在命令行工具中使用docker的命令来操作docker。

### 配置

首先先切换源，

![[技术笔记/_resources/Jenkins l Docker安装教程/1c091c5fcede4ad9b7280ff8174ab935_MD5.png]]

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

![[技术笔记/_resources/Jenkins l Docker安装教程/7c38366f0352dc37c2718312e01a7141_MD5.png]]

然后重启

![[技术笔记/_resources/Jenkins l Docker安装教程/43f054ea2da2a204833a593b55d89314_MD5.png]]

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

![[技术笔记/_resources/Jenkins l Docker安装教程/61b77c0c0e4ec35bfbfdb3a965020a07_MD5.png]]

使用文档可以看[Jenkins文档](https://link.juejin.cn/?target=https%3A%2F%2Fhub.docker.com%2Fr%2Fjenkinszh%2Fjenkins-zh "https://hub.docker.com/r/jenkinszh/jenkins-zh")

下载完成之后进行镜像配置

![[技术笔记/_resources/Jenkins l Docker安装教程/495cb315d1b1f54f7a3f097872f1a54b_MD5.png]]

然后运行好了就可以在本地打开了：[http://localhost:8080](https://link.juejin.cn/?target=http%3A%2F%2Flocalhost%3A8080 "http://localhost:8080")

然后发现Jenkins一直在启动中没有进入

![[技术笔记/_resources/Jenkins l Docker安装教程/070ebb0ca19563cc0a4def933997671d_MD5.png]]

原因：jenkins里面文件指向国外的官网，因为防火墙的原因连不上

解决方法：将配置文件里面的url换成国内的即可

```
ruby
复制代码
# 将红线部分修改为下面的url
http://mirror.xmission.com/jenkins/updates/update-center.json
```

![[技术笔记/_resources/Jenkins l Docker安装教程/21434bea5eea8de040da54c7aa9daee1_MD5.png]] 修改完成后保存重启，如果重启依然没用，然后配置文件的url已经改了，两个方法解决一下：

- 清一下浏览器缓存
- 手动访问一下刚刚修改的那个url

正常进入后页面张这个样子

![[技术笔记/_resources/Jenkins l Docker安装教程/be9f1c362afd8d853936355eec7b7f6f_MD5.png]]

需要输入密码才可正常进入，密码在命令行中可以看到

![[技术笔记/_resources/Jenkins l Docker安装教程/0d0875fe27dabc9af5bfc05c368a0603_MD5.png]]

又或者可以去：目录`/var/jenkins_home/secrets/initialAdminPassword` 里查看。

然后 看到这页面你就大功告成了，除特殊需求外建议安装推荐的插件即可。 ![[技术笔记/_resources/Jenkins l Docker安装教程/9e09138f7ce56a00f3385700bbca0091_MD5.png]]

插件安装失败也不要紧，先点进入系统然后我们在安装。

![[技术笔记/_resources/Jenkins l Docker安装教程/ea243a8953fb30d99d3709f5fe0239fd_MD5.png]]

创建管理员用户，点击完成并保存，然后一路下一步。

![[技术笔记/_resources/Jenkins l Docker安装教程/e1e22b85a6cc8cd345e2756e22b13fcf_MD5.png]]

![[技术笔记/_resources/Jenkins l Docker安装教程/180b69d32c2c31ee0c6319d76159bcbf_MD5.png]]

![[技术笔记/_resources/Jenkins l Docker安装教程/9347ee8d20363e9da96db3d1439ad275_MD5.png]]

![[技术笔记/_resources/Jenkins l Docker安装教程/b47324c2c7d582bbe1037bb700926764_MD5.png]]

配置完成后自动进入首页，这时点击 `系统管理` -> `插件管理` -> `高级` 最底下更新源替换为国内链接，此方法无需重启jenkins；

![[技术笔记/_resources/Jenkins l Docker安装教程/d6e768fbb8b9aba8198a728c49fc684b_MD5.png]]

把url改成一下几种之一：

```
ruby
复制代码
https://mirror.tuna.tsinghua.edu.cn/jenkins/updates/update-center.json

https://updates.jenkins-zh.cn/update-center.json
```

然后在更新一下插件即可。

![[技术笔记/_resources/Jenkins l Docker安装教程/3ea1b3b3c1b3993cfe976ddb7633e901_MD5.png]]

更新完后重启Jenkins即可。