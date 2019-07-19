#前言
传统的开发、测试、部署方式，是由开发人员本机或打包机进行打包，将war包提交给测试人员部署，测试通过后，再由实施人员负责部署到预发、生产环境中。中间的衔接不连贯，容易出错，而且打包、部署存在重复的工作量。自动化构建部署（CICD）就是解决该问题，将从开发到部署的一系列流程变成自动化，衔接连贯，在构建失败时能够告知开发，构建成功后能够告知测试和实施人员。无论大中小公司，都应该有此流程。

我本人在前公司搭建了基于svn(git)+jenkins+maven的自动化构建部署结构，所出的war包部署在tomcat中。此架构仍然不可避免要安装jdk、tomcat、mysql、nginx等应用，而且需要配置环境变量，使用docker可解决上述问题，将所有服务打包成docker镜像，推送到docker registry中。docker的优点就不在这里赘述了
![这里写图片描述](https://imgconvert.csdnimg.cn/aHR0cHM6Ly91cGxvYWQtaW1hZ2VzLmppYW5zaHUuaW8vdXBsb2FkX2ltYWdlcy82OTQ0NjE5LTA2Yjg0ZTlhNWMzYmViOTgucG5n)
#目标
**最终目标**：在linux系统中，搭建`jenkins`服务，定时（或githook）的方式从`github`上拉取maven工程，构建war包。使用docker构建image，推送到`docker registry`上。
>我使用的是ubuntu系统，使用docker形式的jenkins，拉取github工程，gitlab同理，构建war包，再在tomcat镜像的基础上将war包进去，构成新镜像，推送到阿里的registry中，其他的registry（包括自建registry）同理。

为了更好的写这个教程，我逐步完成最终目标，将目标拆分成3个部分。
- 第一步：github代码服务器，提交maven项目
- 第二部：安装jenkins，拉取github工程，构建war包
- 第三步：构建的war包自动推送到tomcat服务器中
- 第四步：基于Dockfile将war包和tomcat镜像构建出新镜像推送到阿里云

>PS：读者最好有linux使用经验，会编写shell脚本。
##第一步 git代码服务器
这里以github为例，其他如svn、gitlab、码云等VCS也都大同小异。有时间我会补充私有的gitlab搭建方式。

>暂时略过，这里先使用github上的公用工程，假装成自己的。
>本人github可用于本例测试：https://github.com/njzcx/DataPlatform

##第二步 jenkins集成
首先你需要有一台linux系统，我使用VMWare搭建的Ubuntu16的虚拟机（本人低配本，觉得VM比VB更快些，虚拟机磁盘最好使用固态，并多分些cpu和内存）。
>有些命令没有时，要会使用`apt-get install`安装。

安装jenkins的docker版本（ps：docker版方便快捷）
打开终端，先把docker安装上
 ` sudo apt install docker.io`
 使用docker安装jenkins，直接调用run命令，会自动pull镜像并运行
 ```
 sudo docker run -d \
-p 8080:8080 \
-p 50000:50000 \
--name jenkins \
-u root \
-v ~/jenkins:/var/jenkins_home  \
jenkinsci/jenkins:lts
 ```
8080端口是jenkins的端口，5000端口是master和slave通信端口（jenkins集群部署后期我再补充，本次为单机配置）。
>顺便说一句，此镜像为jenkins原生，存在一些插件和配置问题，比如不能使用sudo，可根据原声镜像自行扩展，由于不影响此次目标，就不进行再构建了。

初次启动的时候，可以通过`docker logs -f jenkins`查看控制台的密码，通过这个密码登录系统。（~/jenkins的初始化文件也有密码）

启动后就可以通过`127.0.0.1:8080`访问jenkins了。输入密码，新建用户，安装默认插件。手动需要安装的插件有：
`Maven Integration plugin`：有了它在新建Job时才能有Maven项目可以选择
`Deploy to container Plugin`：将war包部署到tomcatshang
`Publish Over SSH`：通过ssh推送文件，并可以执行shell命令
>插件安装完成后最好重启一下jenkins，有几率jenkins会不生效

还需要指定jenkins的jdk和maven，进入`系统管理`->`全局工具配置` ，jdk在jenkins中的`/usr/lib/jvm/java-8-openjdk-amd64`目录中，maven需要让他自动下载（这种方式不是很好，可以使用docker的volumn去挂载一个maven供jenkins使用）
![这里写图片描述](https://img-blog.csdn.net/2018071011045196?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L25qemN4/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
![这里写图片描述](https://img-blog.csdn.net/20180710110459167?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L25qemN4/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
下面开始新建一个Maven项目，在主页左侧点击`新建`，选择`构建一个Maven项目`，点击确定，主页列表会出现该项目。
![这里写图片描述](https://img-blog.csdn.net/20180710102200979?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L25qemN4/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
进入该项目，左侧树中有`配置`按钮，点击进去出现如下界面。
![这里写图片描述](https://img-blog.csdn.net/2018071010222772?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L25qemN4/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
从上到下的配置是（**构建时也是按照从上到下进行执行的**）：
`描述`：就是项目详情，根据项目情况实际情况随意填写
`源码管理`：`Repositories`里面填写giturl，由于开源没有用户密码和ssh文件，下面的`Credentials`为空即可，如果是gitlab私有库或有权限限制则需要`Add`，`Branches to build`选择你需要构建的分支。
`构建触发器`：我选择了两个常用的触发构建方式，`触发远程构建`让git使用hook的方式访问一个jenkins的url进行触发，本例中触发的url为127.0.0.1:8080/job/DataPlatform/build?token=zhangchx。`轮训SCM`是定时检查代码是否有变化，有变化则触发构建，值为5个`*`，分别表示分钟（0-59），小时（0-23），天（1-31），月份（1-12），周（0-7），其中H表示随机，H/5表示每5分钟检查一次。
![这里写图片描述](https://img-blog.csdn.net/20180710102141982?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L25qemN4/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
`构建环境`：无需配置
`Pre Steps`：构建前的操作，可以增加`执行shell`，配置脚本`echo "Pre Steps脚本启动成功"`，此内容会在构建控制台中打印出来
`Build`：`Root POM`配置pom.xml（要构建的工程必须是maven，有pom文件），`Goals and options`配置`clean package`（也就是mvn的构建命令）
`Post Steps`：构建完成后的操作，可以增加`执行shell`，配置脚本`echo "Post Steps脚本启动成功${WORKSPACE}"`，`${WORKSPACE}`为jenkins的环境变量。上方的3个单选项分别代表构建成功后执行、构建成功或不稳定执行、总是执行
![这里写图片描述](https://img-blog.csdn.net/20180710104553560?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L25qemN4/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
`构建设置`：可以配置构建完成后Email通知，我这里没有配置。（很简单，在设置-全局设置中配置Email的发件人账户，这里再配置收件人即可）
`构建后操作`：这一步先不配置
到此基本的配置都已经完成了，可以使用jenkins将github上的代码拉下来进行构建了。返回项目页面，在左侧点击`立即构建`或修改代码等待5分钟或访问`触发远程构建`的URL。jenkins就会开始构建了。
![这里写图片描述](https://img-blog.csdn.net/20180710112203568?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L25qemN4/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
![这里写图片描述](https://img-blog.csdn.net/20180710112212611?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L25qemN4/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
查看控制台，我们可以看到日志，如果失败需要根据日志判断失败原因，是工程build失败还是和jenkins配置有关。
>第一次构建时由于maven要下载jar包，所以有些慢，实在不行就修改pom.xml，把仓库镜像改成国内地址。
##第三步 推送war包到tomcat服务器
上一步已经可以构建出war包，并在target中。这一步我们将war包推送到远程的一台tomcat服务器上去（tomcat我部署在运行VM的宿主机器上）。
进入jenkins的项目配置，修改`构建后操作`这一项
`构建后操作`：由于前面安装了`Deploy to container Plugin`，`Publish Over SSH`插件，这里就会有两个选项
![这里写图片描述](https://img-blog.csdn.net/2018071010490078?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L25qemN4/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
这一步我们只用到`Deploy to container Plugin`，选择它之后，会出现下面这个配置窗。
![这里写图片描述](https://img-blog.csdn.net/20180710112344521?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L25qemN4/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
`WAR/EAR files`：war包相对workspace的地址
`Context path`：部署到tomcat的上下文名称，例如：127.0.0.1:8080/DataCollect可以访问到该项目
`Containers`：指定部署到的tomcat版本，tomcat服务器的地址以及用户名密码，这里用户需要在tomcat中有manager的权限，你需要修改tomcat目录下conf/tomcat-user.xml，添加类似如下的用户。
```
<role rolename="manager-gui"/>
<role rolename="manager-script"/>
<user username="njzcx" password="njzcx" roles="manager-gui,manager-script"/>
```
先启动你的tomcat，再在jenkins构建你的项目，最后war包会被推送到tomcat中去。看构建日志和tomcat日志如下。
![这里写图片描述](https://img-blog.csdn.net/20180710113918371?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L25qemN4/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
访问tomcat的项目地址，可以访问。
![这里写图片描述](https://img-blog.csdn.net/20180710114051241?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L25qemN4/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
>这里我碰到一个坑，就是`WAR/EAR files`一定要存在，不然每次构建都不会执行`构建后操作`，jenkins也不会报错。我一直找不到原因，后来发现war包名称让我写错了
>还有就是选择的tomcat版本和你tomcat服务器版本要对应，不然有些接口发生变化jenkins会访问不到的。

##第四步 基于Dockerfile构建镜像

这一步也很简单，首先你不考虑jenkins，只写一个Dockerfile，能够基于tomcat的镜像+war包构建一个新的镜像就可以了。jenkins的作用就是远程调用一下Dockerfile的build脚本。

Dockerfile在的github里也已经提供了，这里再粘一份。
```
#基础镜像
FROM tomcat:7.0.86

#作者
LABEL maintainer="zhangchx <njzcx@126.com>"

#运行安装telnet和nc
RUN apt-get install -y telnet nc; exit 0

#
VOLUME ["/home/zhangchx/tomcat"]

#TOMCAT环境变量
ENV CATALINA_BASE:   /usr/local/tomcat \
    CATALINA_HOME:   /usr/local/tomcat \
    CATALINA_TMPDIR: /usr/local/tomcat/temp \
    JRE_HOME:        /usr

#启动入口
ENTRYPOINT ["catalina.sh","run"]

#健康检查
# HEALTHCHECK --interval=10s --timeout=3s \
# 	CMD nc -z localhost 5198 >/dev/null || exit 1

#拷贝war包到tomcat
COPY target/DataCollect.war ${CATALINA_HOME}/webapps/
```
Dockerfile如何编写这个需要各位读者自行学习，我这里使用的是tomcat的标准镜像，并通过COPY命令将target的war包拷贝到webapps中。

此Dockerfile在github中，jenkins在拉取源码时，该文件也会被拉取。我们只需要让jenkins把Dockerfile和war包传给docker打包服务器，再调用打包命令就可以生成新的docker镜像，再推送到阿里的registry。

>这里我使用的docker打包机器是VM虚拟机，也就是jenkins的宿主机

由于之前安装了`Publish Over SSH`这个插件，就可以完成上述传输操作。
首先需要到`系统管理`->`系统设置`配置`Publish over SSH`内容。我这里使用的是使用账户密码方式登录（可以使用ssh文件登录）。配置如下：
![这里写图片描述](https://img-blog.csdn.net/20180710125401641?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L25qemN4/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
`Passphrase`：登录密码
`Name`：服务器名称（自定）
`Hostname`：远程服务器地址
`Username`：登录用户
`Remote Directory`：访问的远程目录

再进入jenkins的项目配置，修改`构建后操作`这一项
`构建后操作`：使用`Publish Over SSH`这个插件，对应的选项是`Send build artifacts over SSH`
对`Send build artifacts over SSH`进行配置如下：
`SSH server Name`：需要SSH连接的Name（刚才配置好的）
`Source files`：要拷贝的文件地址（相对`workspace`）
`Remove prefix`：去掉`Source files`的前缀部分
`Remote directory`：要拷贝到host机器的哪个目录（这个目录是相对`Remote Directory`的目录）
`Exec command`：拷贝完成执行的命令

我这里需要传输两个文件，一个是war包，另一个是Dockerfile。我的配置如下：
![这里写图片描述](https://img-blog.csdn.net/20180710130223726?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L25qemN4/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
第一个`Exec command`调用的test.sh是随便echo点东西
第二个`Exec command`是调用一个shell脚本，里面docker会执行build、push等一系列命令，这里贴出来
```/home/zhangchx/docker/docker-datacollect.sh "`pwd`/docker/DataCollect" $BUILD_NUMBER```
```
echo "当前位置："`pwd`
echo "当前用户："`whoami`

# 环境变量ps:我本地的docker在snap中，如果没有这句话下面docker命令找不到
export PATH=$PATH:/snap/bin

# 定义变量
WORKHOME=$1
BUILD_NUMBER=$2
API_NAME="datacollect"
API_VERSION="1.0"
API_PORT=8081
DOCKER_REGISTRY="registry.cn-hangzhou.aliyuncs.com/zhangchx/test"
IMAGE_NAME="$API_NAME:$BUILD_NUMBER"
CONTAINER_NAME=$API_NAME-$API_VERSION
docker --version

# 进入target 目录复制Dockerfile 文件
#cd $WORKSPACE/target
#cp classes/Dockerfile .
cd $WORKHOME

#构建docker 镜像
docker build -t $IMAGE_NAME .

#推送docker镜像
docker push $DOCKER_REGISTRY$IMAGE_NAME

#删除同名docker容器
cid=$(docker ps -a| grep "$CONTAINER_NAME" | awk '{print $1}')
if [ "$cid" != "" ]; then
   docker rm -f $cid
fi

#启动docker 容器
docker run -d -p $API_PORT:8080 --name $CONTAINER_NAME $IMAGE_NAME

#删除 Dockerfile 文件
#rm -f Dockerfile
```
>这里有坑，由于使用的DooD的形式（docker里的jenkins访问宿主机构建），登录用户必须对docker命令有权限，不能加sudo。同时宿主机的docker是在snap目录下，宿主机可以正常使用docker命令（宿主机环境变量里有配置snap），而jenkins远程过来使用的环境变量是jenkins这台docker虚拟机的，所有无法访问docker命令，必须先对PATH进行扩展才行。
>非root执行docker的命令，用户名jmh添加到docker组内：`sudo gpasswd jmh docker`，修改sock权限：`sudo chmod a+rw /var/run/docker.sock`

执行jenkins的构建，可以从控制台看到日志
![这里写图片描述](https://img-blog.csdn.net/20180710125635776?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L25qemN4/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

进入Ubuntu里面查看docker镜像和容器，可以看到容器在运行，也可以正常访问。
![这里写图片描述](https://img-blog.csdn.net/20180710131606340?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L25qemN4/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
![这里写图片描述](https://img-blog.csdn.net/20180710131645123?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L25qemN4/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

#结尾
至此，github+jenkins+maven+docker自动化构建已经达成。该构造可运行在中小公司完全没问题，如果构建频繁等原因性能跟不上，可在此结构上进行扩展，增加jenkins集群和docker服务器。

参考资料：
[https://www.jianshu.com/p/8b1241a90d7a](https://www.jianshu.com/p/8b1241a90d7a)
[https://www.cnblogs.com/hanmk/p/8541814.html](https://www.cnblogs.com/hanmk/p/8541814.html)