#ǰ��
��ͳ�Ŀ��������ԡ�����ʽ�����ɿ�����Ա�������������д������war���ύ��������Ա���𣬲���ͨ��������ʵʩ��Ա������Ԥ�������������С��м���νӲ����ᣬ���׳������Ҵ������������ظ��Ĺ��������Զ�����������CICD�����ǽ�������⣬���ӿ����������һϵ�����̱���Զ������ν����ᣬ�ڹ���ʧ��ʱ�ܹ���֪�����������ɹ����ܹ���֪���Ժ�ʵʩ��Ա�����۴���С��˾����Ӧ���д����̡�

�ұ�����ǰ��˾��˻���svn(git)+jenkins+maven���Զ�����������ṹ��������war��������tomcat�С��˼ܹ���Ȼ���ɱ���Ҫ��װjdk��tomcat��mysql��nginx��Ӧ�ã�������Ҫ���û���������ʹ��docker�ɽ���������⣬�����з�������docker�������͵�docker registry�С�docker���ŵ�Ͳ�������׸����
![����дͼƬ����](https://imgconvert.csdnimg.cn/aHR0cHM6Ly91cGxvYWQtaW1hZ2VzLmppYW5zaHUuaW8vdXBsb2FkX2ltYWdlcy82OTQ0NjE5LTA2Yjg0ZTlhNWMzYmViOTgucG5n)
#Ŀ��
**����Ŀ��**����linuxϵͳ�У��`jenkins`���񣬶�ʱ����githook���ķ�ʽ��`github`����ȡmaven���̣�����war����ʹ��docker����image�����͵�`docker registry`�ϡ�
>��ʹ�õ���ubuntuϵͳ��ʹ��docker��ʽ��jenkins����ȡgithub���̣�gitlabͬ������war��������tomcat����Ļ����Ͻ�war����ȥ�������¾������͵������registry�У�������registry�������Խ�registry��ͬ��

Ϊ�˸��õ�д����̳̣������������Ŀ�꣬��Ŀ���ֳ�3�����֡�
- ��һ����github������������ύmaven��Ŀ
- �ڶ�������װjenkins����ȡgithub���̣�����war��
- ��������������war���Զ����͵�tomcat��������
- ���Ĳ�������Dockfile��war����tomcat���񹹽����¾������͵�������

>PS�����������linuxʹ�þ��飬���дshell�ű���
##��һ�� git���������
������githubΪ����������svn��gitlab�����Ƶ�VCSҲ����ͬС�졣��ʱ���һᲹ��˽�е�gitlab���ʽ��

>��ʱ�Թ���������ʹ��github�ϵĹ��ù��̣���װ���Լ��ġ�
>����github�����ڱ������ԣ�https://github.com/njzcx/DataPlatform

##�ڶ��� jenkins����
��������Ҫ��һ̨linuxϵͳ����ʹ��VMWare���Ubuntu16������������˵��䱾������VM��VB����Щ��������������ʹ�ù�̬�������Щcpu���ڴ棩��
>��Щ����û��ʱ��Ҫ��ʹ��`apt-get install`��װ��

��װjenkins��docker�汾��ps��docker�淽���ݣ�
���նˣ��Ȱ�docker��װ��
 ` sudo apt install docker.io`
 ʹ��docker��װjenkins��ֱ�ӵ���run������Զ�pull��������
 ```
 sudo docker run -d \
-p 8080:8080 \
-p 50000:50000 \
--name jenkins \
-u root \
-v ~/jenkins:/var/jenkins_home  \
jenkinsci/jenkins:lts
 ```
8080�˿���jenkins�Ķ˿ڣ�5000�˿���master��slaveͨ�Ŷ˿ڣ�jenkins��Ⱥ����������ٲ��䣬����Ϊ�������ã���
>˳��˵һ�䣬�˾���Ϊjenkinsԭ��������һЩ������������⣬���粻��ʹ��sudo���ɸ���ԭ������������չ�����ڲ�Ӱ��˴�Ŀ�꣬�Ͳ������ٹ����ˡ�

����������ʱ�򣬿���ͨ��`docker logs -f jenkins`�鿴����̨�����룬ͨ����������¼ϵͳ����~/jenkins�ĳ�ʼ���ļ�Ҳ�����룩

������Ϳ���ͨ��`127.0.0.1:8080`����jenkins�ˡ��������룬�½��û�����װĬ�ϲ�����ֶ���Ҫ��װ�Ĳ���У�
`Maven Integration plugin`�����������½�Jobʱ������Maven��Ŀ����ѡ��
`Deploy to container Plugin`����war������tomcatshang
`Publish Over SSH`��ͨ��ssh�����ļ���������ִ��shell����
>�����װ��ɺ��������һ��jenkins���м���jenkins�᲻��Ч

����Ҫָ��jenkins��jdk��maven������`ϵͳ����`->`ȫ�ֹ�������` ��jdk��jenkins�е�`/usr/lib/jvm/java-8-openjdk-amd64`Ŀ¼�У�maven��Ҫ�����Զ����أ����ַ�ʽ���Ǻܺã�����ʹ��docker��volumnȥ����һ��maven��jenkinsʹ�ã�
![����дͼƬ����](https://img-blog.csdn.net/2018071011045196?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L25qemN4/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
![����дͼƬ����](https://img-blog.csdn.net/20180710110459167?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L25qemN4/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
���濪ʼ�½�һ��Maven��Ŀ������ҳ�����`�½�`��ѡ��`����һ��Maven��Ŀ`�����ȷ������ҳ�б����ָ���Ŀ��
![����дͼƬ����](https://img-blog.csdn.net/20180710102200979?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L25qemN4/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
�������Ŀ�����������`����`��ť�������ȥ�������½��档
![����дͼƬ����](https://img-blog.csdn.net/2018071010222772?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L25qemN4/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
���ϵ��µ������ǣ�**����ʱҲ�ǰ��մ��ϵ��½���ִ�е�**����
`����`��������Ŀ���飬������Ŀ���ʵ�����������д
`Դ�����`��`Repositories`������дgiturl�����ڿ�Դû���û������ssh�ļ��������`Credentials`Ϊ�ռ��ɣ������gitlab˽�п����Ȩ����������Ҫ`Add`��`Branches to build`ѡ������Ҫ�����ķ�֧��
`����������`����ѡ�����������õĴ���������ʽ��`����Զ�̹���`��gitʹ��hook�ķ�ʽ����һ��jenkins��url���д����������д�����urlΪ127.0.0.1:8080/job/DataPlatform/build?token=zhangchx��`��ѵSCM`�Ƕ�ʱ�������Ƿ��б仯���б仯�򴥷�������ֵΪ5��`*`���ֱ��ʾ���ӣ�0-59����Сʱ��0-23�����죨1-31�����·ݣ�1-12�����ܣ�0-7��������H��ʾ�����H/5��ʾÿ5���Ӽ��һ�Ρ�
![����дͼƬ����](https://img-blog.csdn.net/20180710102141982?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L25qemN4/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
`��������`����������
`Pre Steps`������ǰ�Ĳ�������������`ִ��shell`�����ýű�`echo "Pre Steps�ű������ɹ�"`�������ݻ��ڹ�������̨�д�ӡ����
`Build`��`Root POM`����pom.xml��Ҫ�����Ĺ��̱�����maven����pom�ļ�����`Goals and options`����`clean package`��Ҳ����mvn�Ĺ������
`Post Steps`��������ɺ�Ĳ�������������`ִ��shell`�����ýű�`echo "Post Steps�ű������ɹ�${WORKSPACE}"`��`${WORKSPACE}`Ϊjenkins�Ļ����������Ϸ���3����ѡ��ֱ�������ɹ���ִ�С������ɹ����ȶ�ִ�С�����ִ��
![����дͼƬ����](https://img-blog.csdn.net/20180710104553560?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L25qemN4/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
`��������`���������ù�����ɺ�Email֪ͨ��������û�����á����ܼ򵥣�������-ȫ������������Email�ķ������˻��������������ռ��˼��ɣ�
`���������`����һ���Ȳ�����
���˻��������ö��Ѿ�����ˣ�����ʹ��jenkins��github�ϵĴ������������й����ˡ�������Ŀҳ�棬�������`��������`���޸Ĵ���ȴ�5���ӻ����`����Զ�̹���`��URL��jenkins�ͻῪʼ�����ˡ�
![����дͼƬ����](https://img-blog.csdn.net/20180710112203568?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L25qemN4/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
![����дͼƬ����](https://img-blog.csdn.net/20180710112212611?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L25qemN4/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
�鿴����̨�����ǿ��Կ�����־�����ʧ����Ҫ������־�ж�ʧ��ԭ���ǹ���buildʧ�ܻ��Ǻ�jenkins�����йء�
>��һ�ι���ʱ����mavenҪ����jar����������Щ����ʵ�ڲ��о��޸�pom.xml���Ѳֿ⾵��ĳɹ��ڵ�ַ��
##������ ����war����tomcat������
��һ���Ѿ����Թ�����war��������target�С���һ�����ǽ�war�����͵�Զ�̵�һ̨tomcat��������ȥ��tomcat�Ҳ���������VM�����������ϣ���
����jenkins����Ŀ���ã��޸�`���������`��һ��
`���������`������ǰ�氲װ��`Deploy to container Plugin`��`Publish Over SSH`���������ͻ�������ѡ��
![����дͼƬ����](https://img-blog.csdn.net/2018071010490078?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L25qemN4/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
��һ������ֻ�õ�`Deploy to container Plugin`��ѡ����֮�󣬻��������������ô���
![����дͼƬ����](https://img-blog.csdn.net/20180710112344521?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L25qemN4/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
`WAR/EAR files`��war�����workspace�ĵ�ַ
`Context path`������tomcat�����������ƣ����磺127.0.0.1:8080/DataCollect���Է��ʵ�����Ŀ
`Containers`��ָ�����𵽵�tomcat�汾��tomcat�������ĵ�ַ�Լ��û������룬�����û���Ҫ��tomcat����manager��Ȩ�ޣ�����Ҫ�޸�tomcatĿ¼��conf/tomcat-user.xml������������µ��û���
```
<role rolename="manager-gui"/>
<role rolename="manager-script"/>
<user username="njzcx" password="njzcx" roles="manager-gui,manager-script"/>
```
���������tomcat������jenkins���������Ŀ�����war���ᱻ���͵�tomcat��ȥ����������־��tomcat��־���¡�
![����дͼƬ����](https://img-blog.csdn.net/20180710113918371?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L25qemN4/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
����tomcat����Ŀ��ַ�����Է��ʡ�
![����дͼƬ����](https://img-blog.csdn.net/20180710114051241?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L25qemN4/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
>����������һ���ӣ�����`WAR/EAR files`һ��Ҫ���ڣ���Ȼÿ�ι���������ִ��`���������`��jenkinsҲ���ᱨ����һֱ�Ҳ���ԭ�򣬺�������war����������д����
>���о���ѡ���tomcat�汾����tomcat�������汾Ҫ��Ӧ����Ȼ��Щ�ӿڷ����仯jenkins����ʲ����ġ�

##���Ĳ� ����Dockerfile��������

��һ��Ҳ�ܼ򵥣������㲻����jenkins��ֻдһ��Dockerfile���ܹ�����tomcat�ľ���+war������һ���µľ���Ϳ����ˡ�jenkins�����þ���Զ�̵���һ��Dockerfile��build�ű���

Dockerfile�ڵ�github��Ҳ�Ѿ��ṩ�ˣ�������ճһ�ݡ�
```
#��������
FROM tomcat:7.0.86

#����
LABEL maintainer="zhangchx <njzcx@126.com>"

#���а�װtelnet��nc
RUN apt-get install -y telnet nc; exit 0

#
VOLUME ["/home/zhangchx/tomcat"]

#TOMCAT��������
ENV CATALINA_BASE:   /usr/local/tomcat \
    CATALINA_HOME:   /usr/local/tomcat \
    CATALINA_TMPDIR: /usr/local/tomcat/temp \
    JRE_HOME:        /usr

#�������
ENTRYPOINT ["catalina.sh","run"]

#�������
# HEALTHCHECK --interval=10s --timeout=3s \
# 	CMD nc -z localhost 5198 >/dev/null || exit 1

#����war����tomcat
COPY target/DataCollect.war ${CATALINA_HOME}/webapps/
```
Dockerfile��α�д�����Ҫ��λ��������ѧϰ��������ʹ�õ���tomcat�ı�׼���񣬲�ͨ��COPY���target��war��������webapps�С�

��Dockerfile��github�У�jenkins����ȡԴ��ʱ�����ļ�Ҳ�ᱻ��ȡ������ֻ��Ҫ��jenkins��Dockerfile��war������docker������������ٵ��ô������Ϳ��������µ�docker���������͵������registry��

>������ʹ�õ�docker���������VM�������Ҳ����jenkins��������

����֮ǰ��װ��`Publish Over SSH`���������Ϳ�������������������
������Ҫ��`ϵͳ����`->`ϵͳ����`����`Publish over SSH`���ݡ�������ʹ�õ���ʹ���˻����뷽ʽ��¼������ʹ��ssh�ļ���¼�����������£�
![����дͼƬ����](https://img-blog.csdn.net/20180710125401641?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L25qemN4/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
`Passphrase`����¼����
`Name`�����������ƣ��Զ���
`Hostname`��Զ�̷�������ַ
`Username`����¼�û�
`Remote Directory`�����ʵ�Զ��Ŀ¼

�ٽ���jenkins����Ŀ���ã��޸�`���������`��һ��
`���������`��ʹ��`Publish Over SSH`����������Ӧ��ѡ����`Send build artifacts over SSH`
��`Send build artifacts over SSH`�����������£�
`SSH server Name`����ҪSSH���ӵ�Name���ղ����úõģ�
`Source files`��Ҫ�������ļ���ַ�����`workspace`��
`Remove prefix`��ȥ��`Source files`��ǰ׺����
`Remote directory`��Ҫ������host�������ĸ�Ŀ¼�����Ŀ¼�����`Remote Directory`��Ŀ¼��
`Exec command`���������ִ�е�����

��������Ҫ���������ļ���һ����war������һ����Dockerfile���ҵ��������£�
![����дͼƬ����](https://img-blog.csdn.net/20180710130223726?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L25qemN4/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
��һ��`Exec command`���õ�test.sh�����echo�㶫��
�ڶ���`Exec command`�ǵ���һ��shell�ű�������docker��ִ��build��push��һϵ���������������
```/home/zhangchx/docker/docker-datacollect.sh "`pwd`/docker/DataCollect" $BUILD_NUMBER```
```
echo "��ǰλ�ã�"`pwd`
echo "��ǰ�û���"`whoami`

# ��������ps:�ұ��ص�docker��snap�У����û����仰����docker�����Ҳ���
export PATH=$PATH:/snap/bin

# �������
WORKHOME=$1
BUILD_NUMBER=$2
API_NAME="datacollect"
API_VERSION="1.0"
API_PORT=8081
DOCKER_REGISTRY="registry.cn-hangzhou.aliyuncs.com/zhangchx/test"
IMAGE_NAME="$API_NAME:$BUILD_NUMBER"
CONTAINER_NAME=$API_NAME-$API_VERSION
docker --version

# ����target Ŀ¼����Dockerfile �ļ�
#cd $WORKSPACE/target
#cp classes/Dockerfile .
cd $WORKHOME

#����docker ����
docker build -t $IMAGE_NAME .

#����docker����
docker push $DOCKER_REGISTRY$IMAGE_NAME

#ɾ��ͬ��docker����
cid=$(docker ps -a| grep "$CONTAINER_NAME" | awk '{print $1}')
if [ "$cid" != "" ]; then
   docker rm -f $cid
fi

#����docker ����
docker run -d -p $API_PORT:8080 --name $CONTAINER_NAME $IMAGE_NAME

#ɾ�� Dockerfile �ļ�
#rm -f Dockerfile
```
>�����пӣ�����ʹ�õ�DooD����ʽ��docker���jenkins��������������������¼�û������docker������Ȩ�ޣ����ܼ�sudo��ͬʱ��������docker����snapĿ¼�£���������������ʹ��docker�������������������������snap������jenkinsԶ�̹���ʹ�õĻ���������jenkins��̨docker������ģ������޷�����docker��������ȶ�PATH������չ���С�
>��rootִ��docker������û���jmh��ӵ�docker���ڣ�`sudo gpasswd jmh docker`���޸�sockȨ�ޣ�`sudo chmod a+rw /var/run/docker.sock`

ִ��jenkins�Ĺ��������Դӿ���̨������־
![����дͼƬ����](https://img-blog.csdn.net/20180710125635776?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L25qemN4/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

����Ubuntu����鿴docker��������������Կ������������У�Ҳ�����������ʡ�
![����дͼƬ����](https://img-blog.csdn.net/20180710131606340?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L25qemN4/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
![����дͼƬ����](https://img-blog.csdn.net/20180710131645123?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L25qemN4/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

#��β
���ˣ�github+jenkins+maven+docker�Զ��������Ѿ���ɡ��ù������������С��˾��ȫû���⣬�������Ƶ����ԭ�����ܸ����ϣ����ڴ˽ṹ�Ͻ�����չ������jenkins��Ⱥ��docker��������

�ο����ϣ�
[https://www.jianshu.com/p/8b1241a90d7a](https://www.jianshu.com/p/8b1241a90d7a)
[https://www.cnblogs.com/hanmk/p/8541814.html](https://www.cnblogs.com/hanmk/p/8541814.html)