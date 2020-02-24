---
typora-root-url: ./
---

## 上线工具 客户端

> 上线工具客户端 是[<http://h5.companyclub.cn/>](流刀上线工具)用于在您团队内网进行项目的打包上线发布流程的代理工具。流刀上线工具并不会直接与该客户端工具所在的服务器进行交互，但要求您团队的成员能够有权访问到上线工具部署的服务器。

##### 体验账号

账号：test@qq.com  密码：Test123456  测试微服务对应的git地址为：<https://gitee.com/onlinetool/test>

![线上部署](/doc/演示.gif)

> 由于作者本人忙于工（ban）作（zhuan），对该上线工具中出现的各类bug的维护缺乏诚意。固将项目的全部代码公开，如果您的企业也有同样或类似的需求，可以在贵司的企业内部部署，遇到bug帮忙完善。。。我想该项目适合创业公司使用，笔者所在的公司则是使用瓦力上线工具进行上线。但瓦力有三个缺点，让我无法忍受，固忙里偷闲的开发了此项目：
>
> 1. 对微服务支持力太差。笔者一个app的版本迭代上线通常涉及六七个微服务，使用瓦力需要创建六七个上线单。
>
> 2. 没有考虑互联网公司开发、测试、预发布、上线流程的规范化的问题。一般都是自己提交上线单自己上线。
>
> 3. 只支持代码级别的部署上线，对数据库、定时任务、后台脚本的上线根本没有支持。
>
> 4. 上线过程中有时会遇到莫名其妙的bug，包括软连接指向错误、执行脚本命令卡死。。。
>
>    如果您在使用本产品的过程中，不仅帮忙改善了本软件的一些bug，还增加了以下这些功能，并将它作为产品开源出来，将是功德无量啊。
>
>    1. 自动在预发布环境跑数据功能。开发人员将导数据的需求写在某个微服务的某个分支上，上线人员在预发布环境切到该分支，执行开发人员指定的cli命令。完了之后，将预发布环境指定目录下的某个csv文件下载下来，给程序员或相关运营部门的人员。
>    2. 提交上线单时指定的定时任务，能够在上线单上线之后自动部署到服务器。主要目的无非是，将固定重复且包含一定危险性的登录服务器的操作，尽量交给程序来完成。

上线工具服务端：<https://gitee.com/onlinetool/onlinetool_server> 使用springboot框架开发。编译部署不知道的别来问我，代码架构就是springboot的标准代码结构

上线工具前端：<https://gitee.com/onlinetool/static> 使用react框架开发，redux要求的标准代码结构。代码编译打包命令：npm run build.

希望在研究学习代码的过程中不忘造福他人的使命，一个不想写业务逻辑的架构师不是一个好码农！！！

### 主要功能：

- 项目/微服务管理

  每个项目由一个或多个微服务构成。

  微服务管理 可以配置微服务名称、git文件夹目录、主分支、线上服务器、线上数据库、部署目录、打包前执行的名称、需要打包的目录、打包排除文件、部署后执行命令等。

  ![微服务配置](/doc/微服务配置.gif)

- 提交上线单

  勾选该项目下需要上线的微服务，填写微服务需要上线的分支，如需在线上建表，可填写sql语句。另外也支持配置job服务器执行job指令等。

- 完整的上线流程管控

  提交上线单->部署到测试环境->测试环境测试通过->部署到预发布环境->预发布环境测试通过->代码上线。

  全流程自动化。各流程通过企业微信/钉钉进行通知相关人员。

  ![流程管控](/doc/流程管控.gif)

  

  #### 问答：

  1. 上线工具上线的原理是什么？

     在测试、预发布环境部署是通过连接到部署的服务器，执行分支切换脚本、源代码编译脚本以及job相关脚本进行部署的。在线上服务器部署，我们在客户端服务器上进行分支合并、代码编译最后将部署内容打包成tar包，分发到需要部署的每一台服务器上。最后进行压缩包的解压部署流程。

  2. 如何进行迭代部署？如何进行重新部署？

     在测试、预发布、线上每一次部署之前，我们都会在客户端机器下记录当前git的commitID,当前部署已经成功执行掉的sql语句，已经成功创建的job。无论是迭代部署还是重新部署，已经执行过的sql语句都不会重复执行，已经创建的job也不会重复创建（但会重新reload相关job）。迭代部署相对于重新部署，会检测对应git的commitID是否发生变化，只部署在上次部署之后，发生过代码变化的分支，也只执行之前没有执行过的sql语句和job脚本。   

  3. 可能存在哪些问题？

     该部署工具在原理上参照  [瓦力上线工具](http://walle-web.io/) ，是通过在目标服务器上创建文件的方式进行部署，可能存在上线过程中出现线上服务有报错的情况。因此，应尽量避免业务访问高峰时上线代码。同时，如果做好负载均衡，通过切换部署服务器的方式也可以避免这样的问题。

     

  ### 从源码编译：

  需要jdk 1.8

  git 克隆项目源码，执行

  ```
  ./gradlew bootJar
  ```

  运行

  ```
  nohup java -jar build/lib/onlinetool_client.jar &
  ```

  使用nginx代理配置

  ```nginx
  server {
      listen       80;
      server_name  client.companyclub.cn;
  
      location / {
          proxy_pass http://127.0.0.1:9535;
          proxy_set_header    Host     $host;
          proxy_http_version 1.1;
          proxy_set_header Upgrade $http_upgrade;
          proxy_set_header Connection "upgrade";
      }
  
      access_log  /var/log/nginx/client.companyclub.cn.log main;
  }
  ```

  

  ### 联系我

  使用中有任何问题可以直接与我沟通，微信如下：

  ![加我微信](/doc/加我微信.png)

  