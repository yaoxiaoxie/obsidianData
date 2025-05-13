  

## 一、准备工作：

1、安装jenkins  
2、安装jenkins插件(Email Extension Plugin)  
3、注册163邮箱，并开始POP3/SMTP/IMAP，设置客户端授权码（授权码记录在文档里，后续jenkins配置需要用到）  
![[技术笔记/_resources/Jenkins l Jenkins邮箱设置/d4111da300266dfe07618230bf63439c_MD5.png]]
![[技术笔记/_resources/Jenkins l Jenkins邮箱设置/cfd25d5a947823fa6a1f13201ecc80bf_MD5.png]]

可以点击新增授权码：  
![[技术笔记/_resources/Jenkins l Jenkins邮箱设置/5b3ed9c45ac9e87bef87a94de860c61c_MD5.png]]
## 二、Jenkins自动发送邮件配置：

### 1.打开系统管理->系统配置

在系统设置中找到Jenkins Locaction项填入Jenkins URL和系统管理员邮件地址，系统管理员邮件地址一定要配置，否则发不了邮件通知。因为邮件通知都是由系统管理员的邮箱发出来的。  
![[技术笔记/_resources/Jenkins l Jenkins邮箱设置/3dc6ade02a6e6874c7cfc069feefae75_MD5.png]]

### 2.设置发件人等信息

PS：这里的发件人邮箱地址切记要和系统管理员邮件地址保持一致（当然，也可以设置专门的发件人邮箱，不过不影响使用，根据具体情况设置即可）  
![[技术笔记/_resources/Jenkins l Jenkins邮箱设置/00bdd0a84b3b4f95e1e3fc60afd65dca_MD5.png]]

### 3.设置报告的格式和默认邮箱

![[技术笔记/_resources/Jenkins l Jenkins邮箱设置/7ed2851af08b24ee5b2a11058fba4674_MD5.png]]

### 4.设置邮件模板内容

![[技术笔记/_resources/Jenkins l Jenkins邮箱设置/9a10698bf0a488fb758f7bce0b72d3b4_MD5.png]]

```scss
【构建通知】：$PROJECT_NAME - Build # $BUILD_NUMBER - $BUILD_STATUS!
```

配置邮件内容模版

```xml
<!DOCTYPE html>    
<html>    
<head>    
<meta charset="UTF-8">    
<title>${ENV, var="JOB_NAME"}-第${BUILD_NUMBER}次构建日志</title>    
</head>    
    
<body leftmargin="8" marginwidth="0" topmargin="8" marginheight="4"    
    offset="0">    
    <table width="95%" cellpadding="0" cellspacing="0"  style="font-size: 11pt; font-family: Tahoma, Arial, Helvetica, sans-serif">    
        <tr>    
            本邮件由系统自动发出，无需回复！<br/>            
            各位同事，大家好，以下为${PROJECT_NAME }项目构建信息</br> 
            <td><font color="#CC0000">构建结果 - ${BUILD_STATUS}</font></td>   
        </tr>    
        <tr>    
            <td><br />    
            <b><font color="#0B610B">构建信息</font></b>    
            <hr size="2" width="100%" align="center" /></td>    
        </tr>    
        <tr>    
            <td>    
                <ul>    
                    <li>项目名称 ： ${PROJECT_NAME}</li>    
                    <li>构建编号 ： 第${BUILD_NUMBER}次构建</li>    
                    <li>触发原因： ${CAUSE}</li>    
                    <li>构建状态： ${BUILD_STATUS}</li>    
                    <li>构建日志： <a href=" ">${BUILD_URL}console</a ></li>    
                    <li>构建  Url ： <a href="${BUILD_URL}">${BUILD_URL}</a ></li>    
                    <li>工作目录 ： <a href="${PROJECT_URL}ws">${PROJECT_URL}ws</a ></li>    
                    <li>项目  Url ： <a href="${PROJECT_URL}">${PROJECT_URL}</a ></li>
                    <li>测试报告： <a href="${PROJECT_URL}HTML_20Report">${PROJECT_URL}HTML_20Report</a ></li>        
                </ul>    

<h4><font color="#0B610B">失败用例</font></h4>
<hr size="2" width="100%" />
$FAILED_TESTS<br/>

<h4><font color="#0B610B">最近提交(#$SVN_REVISION)</font></h4>
<hr size="2" width="100%" />
<ul>
${CHANGES_SINCE_LAST_SUCCESS, reverse=true, format="%c", changesFormat="<li>%d [%a] %m</li>"}
</ul>
详细提交: <a href="${PROJECT_URL}changes">${PROJECT_URL}changes</a ><br/>

            </td>    
        </tr>    
    </table>    
</body>    
</html>
```

### 5.配置Jenkins自带的邮件功能，配置内容如下，和Email Extension Plugin插件同样的配置，可以通过勾选通过发送测试邮件测试配置按钮来测试配置是否成功发送邮件

![[技术笔记/_resources/Jenkins l Jenkins邮箱设置/dc694ccfe6918004496755d15d8d297e_MD5.png]]

可测试邮件是否发送成功  
![[技术笔记/_resources/Jenkins l Jenkins邮箱设置/d55447d90420abce639ed387d896d09d_MD5.png]]

配置完成之后点击 应用 保存

## 三、项目配置

在完成系统设置后，还需要给需要构建的项目进行邮件配置  
进入项目->配置->构建后操作：  
![[技术笔记/_resources/Jenkins l Jenkins邮箱设置/901785e0f8fe9f782fdb070d0ec1f672_MD5.png]]

![[技术笔记/_resources/Jenkins l Jenkins邮箱设置/0d794768e2fa2e553ffef994dd9adc0c_MD5.png]]

![[技术笔记/_resources/Jenkins l Jenkins邮箱设置/4a8e6caae3d9fc363fa85d8ce28d45a4_MD5.png]]

![[技术笔记/_resources/Jenkins l Jenkins邮箱设置/14e36a77af1a90e1fe31e41c45aa9011_MD5.png]]

配置内容默认即可，邮件内容类型可以根据自己的配置选择，收件人列表可以从前面的系统设置中默认收件人选项配置。  
![[技术笔记/_resources/Jenkins l Jenkins邮箱设置/accdbd35de45d988736f4a55556e8797_MD5.png]]

## 四、构建触发邮件测试

如下图，为我收到的测试邮件，邮件内容可以通过系统设置里面进行个性化的配置，可参考我上面的模板，或者自定义即可。

![[技术笔记/_resources/Jenkins l Jenkins邮箱设置/91e289ce466f87a7431ba563cc1ea438_MD5.png]]
