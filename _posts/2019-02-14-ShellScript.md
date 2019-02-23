# Q. 서버 세팅을 하려는게 1~2대가 아닌 여러대 일 때?
다양한 방법이 있지만, 저희 행복TF에서는 스크립트를 작성하여 서버 세팅을 하는 것으로 이야기 되었습니다. 


## echo와 sed를 활용한 Bash shell script를 작성하여 한번에 처리하자!

echo와 sed를 사용하면 파일 내용을 추가하고 수정할 수 있습니다. 

### echo
예시 코드
```bash
echo '{문자열}' >> 파일
```
echo 와 >> 를 사용하면 파일의 제일 끝부분에 문자열을 추가할 수 있습니다.
```bash
echo 'export JAVA_HOME=/home_path/apps/jdk' >> ~/.bash_profile
echo 'export PATH=${JAVA_HOME}/bin:$PATH' >> ~/.bash_profile
```
echo를 사용하여 다양한 스크립트 작성이 가능하겠지만 우선 위 코드처럼 파일의 제일 끝부분에 문자열을 집어넣는 용도로만 사용했습니다.


* * *


### sed 
예시 코드
```bash
sed -i 's/{찾는 문자열}/{바꿀 문자열}/옵션' 파일명
```
옵션에는 g를 통해 global 옵션을 추가할 수 있습니다. 파일에서 모든 찾는 문자열을 바꿀 문자열로 바꾸는 옵션입니다.

```bash
sed -i 's/JkMount \/\*.jsp tomcat/JkMount \/\* tomcat_mail/' ~/apps/apache/conf/httpd.conf
```
-i 옵션을 이용하여 문자열 패턴을 찾아 다른 문자열로 바꾸는 코드를 작성했습니다.

```bash
sed -i 's/JkMount \/\*.nhn tomcat//' ~/apps/apache/conf/httpd.conf
```
문자열을 지우고 싶을 때 위와 같은 방법으로 빈 문자열로 바꾸는 것도 가능합니다.

```bash
sed -i '{줄번호}{i옵션} {문자열}' 파일이름
```
위와 같은 방식으로 특정 행에 문자열을 추가할 수도 있습니다.
```bash
sed -i '13i export JAVA_OPTS="$JAVA_OPTS -Dspring.profiles.active=db"' ~/apps/tomcat_account/bin/setenv.sh
```
특정 행을 지정하여 문자열을 추가하는 경우 문자열을 추가하려는 파일의 내용이 바뀌거나 했을 경우에는 위험할 수 있지만 우선 이런식으로 진행했습니다.



* * *

### 전체 스크립트
저희 행복TF와 톰캣 구성이 다르면 스크립트를 그대로 활용할 수는 없겠지만 참고하여 스크립트를 수정하면 될 것 같습니다.
스크립트에 대략적인 설명을 작성했습니다.

서버 세팅하는 것과 다들 사용하고 있을 거라 생각하는 deploy와는 별개로 구성해야할 것 같아서 이 스크립트로는 톰캣이 켜지지 않습니다.
세팅을 한 뒤에 deploy 스크립트를 활용하여 실행시키면 될 것 같습니다!
```bash
#!/bin/bash

## install java
./set_webapps.sh install jdk@openjdk8u_hotspot_8u192b12

echo 'export JAVA_HOME=/home_path/apps/jdk' >> ~/.bash_profile
echo 'export PATH=${JAVA_HOME}/bin:$PATH' >> ~/.bash_profile

## install apache
./set_webapps.sh install apache@2.4.37

echo 'export APACHE_HTTP_HOME=/home_path/apps/apache' >> ~/.bash_profile
echo 'export PATH=${APACHE_HTTP_HOME}/bin:$PATH' >> ~/.bash_profile

## install tomcat
./set_webapps.sh install tomcat@8.5.28

echo 'export TOMCAT_HOME=/home_path/apps/tomcat' >> ~/.bash_profile
echo 'export PATH=${TOMCAT_HOME}/bin:$PATH' >> ~/.bash_profile

source ~/.bash_profile


## git clone
mkdir -p ~/build
cd ~/build
git clone https://github.com/{유저명}/{프로젝트 이름}.git

## web 프로젝트
# build
cd ~/build/{프로젝트 이름}/{프로젝트 web 폴더명}
./mvnw clean package -DskipTests
cd target
mv {프로젝트 web 폴더명}*.war {프로젝트 web 폴더명}.war
rm ~/deploy/{프로젝트 web 폴더명}.war
cp {프로젝트 web 폴더명}.war ~/deploy/

## api 프로젝트
# build
cd ~/build/{프로젝트 이름}/{프로젝트 api 폴더명}
./mvnw clean package -DskipTests
cd target/
mv {프로젝트 api 폴더명}*.war {프로젝트 api 폴더명}.war
rm ~/deploy/{프로젝트 api 폴더명}.war
cp {프로젝트 api 폴더명}.war ~/deploy/


## tomcat copy
cd ~/apps
cp -r apache-tomcat-8.5.28 apache-tomcat-8.5.28_2

ln -s apache-tomcat-8.5.28_2 {web 프로젝트 명}
mv tomcat {api 프로젝트 명}

cd ~


## setting apache
sed -i 's/JkMount \/\*.jsp tomcat/JkMount \/\* {web 프로젝트 명}/' ~/apps/apache/conf/httpd.conf
sed -i 's/JkMount \/\*. tomcat//' ~/apps/apache/conf/httpd.conf
sed -i 's/JkMount \/jkmanager\/\* jkstatus//' ~/apps/apache/conf/httpd.conf

## tomcat_mail change war
sed -i 's/{web 프로젝트 명}.war/' ~/apps/{web 프로젝트 명}/conf/server.xml

## tomcat_accout change port
sed -i 's/"7001"/"17001"/' ~/apps/{api 프로젝트 명}/conf/server.xml
sed -i 's/"8001"/"18001"/' ~/apps/{api 프로젝트 명}/conf/server.xml
sed -i 's/"9001"/"19001"/' ~/apps/{api 프로젝트 명}/conf/server.xml

## tomcat_account change war
sed -i 's/{api 프로젝트 명}.war/' ~/apps/{api 프로젝트 명}/conf/server.xml

## tomcat insert export catalina
sed -i "2i export CATALINA_HOME=/home_path/apps/{api 프로젝트 명}" ~/apps/{api 프로젝트 명}/bin/catalina.sh
sed -i "3i export TOMCAT_HOME=/home_path/apps/{api 프로젝트 명}" ~/apps/{api 프로젝트 명}/bin/catalina.sh
sed -i "4i export CATALINA_BASE=/home_path/apps/{api 프로젝트 명}" ~/apps/{api 프로젝트 명}/bin/catalina.sh
sed -i "5i CATALINA_PID=/home_path/apps/{api 프로젝트 명}/bin/tomcat.pid" ~/apps/{api 프로젝트 명}/bin/catalina.sh

sed -i "2i export CATALINA_HOME=/home_path/apps/{web 프로젝트 명}" ~/apps/{web 프로젝트 명}/bin/catalina.sh
sed -i "3i export TOMCAT_HOME=/home_path/apps/{web 프로젝트 명}" ~/apps/{web 프로젝트 명}/bin/catalina.sh
sed -i "4i export CATALINA_BASE=/home_path/apps/{web 프로젝트 명}" ~/apps/{web 프로젝트 명}/bin/catalina.sh
sed -i "5i CATALINA_PID=/home_path/apps/{web 프로젝트 명}/bin/tomcat.pid" ~/apps/{web 프로젝트 명}/bin/catalina.sh


## apache change workers.properties
sed -i 's/tomcat/{web 프로젝트 명}/g' ~/apps/apache/conf/workers.properties
sed -i 's/worker.list={web 프로젝트 명}/worker.list=tomcat_mail ,{api 프로젝트 명}/' ~/apps/apache/conf/workers.properties
sed -i "10i worker.{api 프로젝트 명}.type=ajp13 \nworker.{}.port=18001\n#worker.tomcat_account.connect_timeout=1000\n#worker.tomcat_account.prepost_timeout=1000\nworker.tomcat_account.socket_timeout=10\nworker.tomcat_account.connection_pool_timeout=10\n#worker.tomcat_account.reply_timeout=1000\n" ~/apps/apache/conf/workers.properties


## tomcat profile default setting
sed -i '13i export JAVA_OPTS="$JAVA_OPTS -Dspring.profiles.active=db"' ~/apps/tomcat_account/bin/setenv.sh
sed -i '13i export JAVA_OPTS="$JAVA_OPTS -Dspring.profiles.active=web"' ~/apps/tomcat_mail/bin/setenv.sh
```


* * *
스크립트 작성이 처음이라 부족한 부분이 많습니다. 
java 설치할 때나 apache, tomcat 설치 할 때 \[Y/N]이 나오는 부분이나
git pull을 처음 할 때 아이디 비밀번호를 물어보는 경우 자동으로 입력되는 부분을 찾아보려 했는데 아직 못찾았습니다. 찾게되면 공유하겠습니다.
더 좋은 방법으로 작성할 수 있게 계속 찾아보겠지만 문제점이나 더 나은 방법이 있다면 공유 부탁드립니다.
<!--stackedit_data:
eyJoaXN0b3J5IjpbMjA0MDcxODU5N119
-->