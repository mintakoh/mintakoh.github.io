---
layout: post
title:  "서버 세팅을 위한 bash shell script 작성방법"
date:   2019-02-14 11:20:00
author: Mintak OH
categories: Dev
tags: NHN-Ent dev script shell
---

# Q. 서버 세팅을 하려는게 1~2대가 아닌 여러대 일 때?
다양한 방법이 있지만,  스크립트를 작성하여 서버 세팅을 하는 방법을 소개하겠다.


## echo와 sed를 활용한 Bash shell script를 작성하여 한번에 처리하자!

echo와 sed를 사용하면 파일 내용을 추가하고 수정할 수 있다. 

### echo
예시 코드
```bash
echo '{문자열}' >> 파일
```
echo 와 >> 를 사용하면 파일의 제일 끝부분에 문자열을 추가할 수 있다.
```bash
echo 'export JAVA_HOME=/home_path/apps/jdk' >> ~/.bash_profile
echo 'export PATH=${JAVA_HOME}/bin:$PATH' >> ~/.bash_profile
```
echo를 사용하여 다양한 스크립트 작성이 가능하겠지만 우선 위 코드처럼 파일의 제일 끝부분에 문자열을 집어넣는 용도로만 사용했다.


* * *


### sed 
예시 코드
```bash
sed -i 's/{찾는 문자열}/{바꿀 문자열}/옵션' 파일명
```
옵션에는 g를 통해 global 옵션을 추가할 수 있습니다. 파일에서 모든 찾는 문자열을 바꿀 문자열로 바꾸는 옵션이다.

```bash
sed -i 's/JkMount \/\*.jsp tomcat/JkMount \/\* {tomcat 폴더 이름}/' ~/apps/apache/conf/httpd.conf
```
-i 옵션을 이용하여 문자열 패턴을 찾아 다른 문자열로 바꾸는 코드를 작성했다.

```bash
sed -i 's/JkMount \/\*.jsp tomcat//' ~/apps/apache/conf/httpd.conf
```
문자열을 지우고 싶을 때 위와 같은 방법으로 빈 문자열로 바꾸는 것도 가능하다.

```bash
sed -i '{줄번호}{i옵션} {문자열}' 파일이름
```
위와 같은 방식으로 특정 행에 문자열을 추가할 수도 있다.
```bash
sed -i '13i export JAVA_OPTS="$JAVA_OPTS -Dspring.profiles.active=db"' ~/apps/{tomcat 폴더 이름}/bin/setenv.sh
```
특정 행을 지정하여 문자열을 추가하는 경우 문자열을 추가하려는 파일의 내용이 바뀌거나 했을 경우에는 위험할 수 있지만 우선 이런식으로 진행했다.



* * *

<!--stackedit_data:
eyJoaXN0b3J5IjpbNTUwMDQ3ODk1XX0=
-->