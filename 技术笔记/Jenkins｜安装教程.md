
## 一、简介

官网：[https://www.jenkins.io](https://www.jenkins.io/)

中文文档：[https://www.jenkins.io/zh/](https://www.jenkins.io/zh/)

`Jenkins` 是一个开源的[持续集成]（CI）工具，用于自动化构建、测试和部署软件项目。它提供了一个易于使用和可扩展的平台，帮助团队更高效地开发和交付软件。

`Jenkins` 的主要特点和用途包括：

1. 自动化构建：`Jenkins` 可以从源代码库（如`Git`、`SVN` 等）中获取最新的代码，并自动进行构建。它支持各种构建工具和构建脚本，如`Ant、Maven、Gradle` 等。
    
2. 持续集成：`Jenkins` 可以将多个开发者的代码集成到共享的主线分支中，并定期执行构建和测试。这有助于发现和解决集成问题，确保软件的稳定性和可靠性。
    
3. 测试自动化：`Jenkins` 可以与各种测试框架和工具集成，如 `JUnit、Selenium、JMeter` 等。它可以自动执行各种测试，并生成测试报告和分析结果。
    
4. 部署自动化：`Jenkins` 可以自动化部署应用程序到目标服务器或云平台。它支持各种部署工具和配置管理工具，如 `Docker、Kubernetes、Ansible` 等。
    
5. 插件生态系统：`Jenkins`拥有一个强大的插件生态系统，提供了各种功能和集成选项。用户可以根据需要选择和安装插件，以扩展和定制 `Jenkins` 的功能。
    

总之，`Jenkins` 是一个功能强大、易于使用的持续集成工具，可以帮助团队实现软件开发和交付的自动化。通过自动化构建、测试和部署过程，可以提高团队的效率、减少错误，并加速软件项目的交付。

---

## 二、安装前准备

在安装 `jenkins` 之前要先确保电脑上是否已配置过 `Java` 的环境变量，可调出命令窗口（`win + R` 再输入 `cmd`），通过 `java -version` 来检验

如果没有显示 `Java` 的版本信息，就需要先配置 `Java` 环境变量。

---

## 三、windows 安装与启动

进入 `Jenkins` 的 [官方下载页面](https://www.jenkins.io/download/)

`LTS` 是长期支持的版本，是稳定的版本

在 `Windows` 下 `Jenkins` 的安装有三种方式：

- [方式一：下载 `war` 包通过命令启动]
- [方式二：`war` 结合`tomcat`进行安装]
- [方式三：下载安装程序包 `msi` 文件]

在下载安装包之前要先确定应该下载哪个版本的 `Jenkins`，`Jenkins` 的版本依赖于 `Java` 的版本，可在 [Jenkins-Java Support Policy](https://www.jenkins.io/doc/book/platform-information/support-policy-java/) 中进行查看

如果你下载的 `Jenkins` 版本与本地 `Java` 不支持，那么 `Jenkins` 是无法安装成功的，比如说我电脑上 `JDK` 的版本是 `1.8.0_172`，也就是 `Java 8`，那么我只能安装 `2.346.1` 或者该版本之前的，在 `Past Releases` 上可以查看到历史版本

==这里下载 Jenkins 建议下载最新版本的，可以避免很多插件安装不兼容的问题~~，我使用 2.332.4 版本的 Jenkins 就遇到了很多插件安装不上的问题，故又将 JDK 升级到 11，安装了 Jenkins 2.440.1 版本的==

---

### 1. 方式一

`jenkins` 可以通过 `war` 的形式安装起来，`war` 包可以通过 `java -jar` 的命令或者放到 `Tomcat` 上启动起来

首先需要下载 `Java` 所支持的 `jenkins` 版本，我用的时 `Java-8`，所以我就下载 `2.332.4` 的

下载完成后，在 `war` 包所在目录下进入 `cmd 命令`，通过以下命令

```bash
java -jar jenkins.war --httpPort=8080
```

当看到 `Jenkins is fully up and running` 就表示 `jenkins` 已经启动完成了


---

### 2. 方式二

按照 `方式一` 的步骤下载 `war` 包，再将 `war` 放到 `tomcat` 的 `wabapps` 的目录下

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/7bb71992f5de202a0449603cefd2c4e5.png)

启动 `tomcat`，访问 `localhost:tomcatPort/jenkins`，例如：[http://localhost:8080/jenkins](http://localhost:8080/jenkins)

---

### 3. 方式三

在 `LTS` 下选择 `Windows`


即可下载到一个 `jenkins.msi` 安装程序包

注意：我没有找打 `Java 8` 支持的 `jenkins.msi`，包括一些开源的镜像站，如果用这种方式下载 `Jenkins`，建议先下载一个 `JDK 11`

双击运行

点击 `Next`

选择 `安装路径`，再点击 `Next`

选择 `Run service as LocalSystem (not recommended)`，点击 `Next`

设置 `端口号`，测试端口号是否可行，`可行之后` 才能点击 `Next`

选择 `JDK` 的安装路径，再点击 `Next`

点击 `Next`

点击 `Install` 进行安装

点击 `Finish` 完成安装

---

## 四、创建管理员用户

安装完成之后在游览器上访问 `localhost:port` ，`port` 是安装时设置的端口号，比如：[localhost:8080](http://localhost:8080/)

注意：不同版本的 `jenkins` 页面可能会有点差异

根据提示的路径就能找到存放 `管理员密码` 的文件 `initialAdminPassword`

不过此时暂时不用着急地去粘贴 `管理员密码`，因为 `jenkins` 的服务器在国外，到安装插件步骤时会加载得比较慢

建议先去[设置成国内的镜像]

配置好镜像重启后再访问 `localhost:port`，从本地复制密码并粘贴到指定位置

点击 `继续`

如果刚刚已经换成国内的镜像网址了，所以可以直接选择 `安装推荐的插件`，让它自动下一些常用的插件也很快

如果没有替换成国内镜像，直接 `安装推荐的插件` 会比较慢，也可以点击 `选择插件来安装`，再点击 `无`，不安装任何插件，再点击 `安装`

并且成功率比较高

对应那些安装失败的插件可以 `重试` 再安装，有些插件会因为依赖的关系安装不上的话就 `继续` 也没啥影响

创建 `管理员用户` 之后，点击 `保存并完成`

点击 `保存并完成`

到此为止 `jenkins` 就安装完成了，可以点击 `开始使用 jenkins`

---

## 五、常用设置

### 1. 配置镜像地址

在 `jenkins` 的工作目录 `.jenkins` 中，找到 `hudson.model.UpdateCenter.xml` 文件打开

将 `https://updates.jenkins.io/update-center.json` 替换成国内镜像网址（需要管理员权限修改）

- [国内镜像网址](https://mirrors.tuna.tsinghua.edu.cn/)：`https://mirrors.tuna.tsinghua.edu.cn/jenkins/updates/update-center.json`
- [国外镜像网址](https://mirror.xmission.com/)：`https://mirror.xmission.com/jenkins/updates/update-center.json`

再进入到 `updates` 目录下，编辑 `default.json` 文件，将该文件中国外的地址全部替换成国内的（需要管理员权限修改）

- `https://www.google.com` 替换成 `https://www.baidu.com`
- `https://updates.jenkins.io/download` 替换成 `https://mirrors.tuna.tsinghua.edu.cn/jenkins`

修改完配置之后需要重启 `jenkins`， `Win + R` 运行 `compmgmt.msc`

打开 `计算机管理` 界面

在 `服务和应用程序` - `服务` 下找到 `jenkins` 服务，选中右键，点击 `重新启动`

---

### 2. 更改工作目录

从上面安装过程可知 `Jenkins` 的工作目录默认在 `C` 盘下，而 `C` 盘的资源是比较珍贵的，一般情况下会尽量避免将工作目录放置 `C` 盘中，所以在有些情况下可能就需要更改工作目录

在 `计算机管理` 界面中先停止 `jenkins` 程序

再打开 `jenkins` 的安装目录，找到 `jenkins.xml` 文件，进行编辑

将 `%ProgramData%\Jenkins\.jenkins` 修改为目标目录，比如：`D:\jenkins\windows\jenkins-2.440-work`

重启 `jenkins` 即可

---

### 3. 开启可注册用户

默认情况下是不可以注册用户的，如果想要开启注册用户，以 `2.440.1` 的 `jenkins` 版本为例，在 `Manage Jenkins` - `Security` 中选中 `Security`

进入到以下页面

开启 `允许用户注册`，再点击 `应用` 和 `保存`

回到登录页面就能看到已经可以注册用户了

---

### 4. 全局变量配置

以 `2.440.1` 的 `jenkins` 版本为例，在 `Manage Jenkins` - `System Configuration` 中选中 `Tools`

在这里就可以配置 `JDK`、`Ant`、`Maven` 等配置
