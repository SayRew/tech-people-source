---
title: React-1부 [개발환경 구축하기]
date: 2019-12-12 14:56:55
thumbnail : /images/react-hooks.png
tags: [react]
category : [IT Tech, 3. React]
---

> 작성자 : 플랫폼 개발실 R&D팀 유주빈

#### 들어가기 앞서

- 일반 사용자 혹은 기업을 상대로 소프트웨어를 개발하고 서비스를 제공하는 과정은 매우 많은 과정이 있습니다. 이를 조금이나마 간편하게 하고자 아래와 같은 실습을 진행할 예정입니다.

- 시스템 구성도

- 위의 그림은 개발자가 개발을 완료하면 해당 소스코드에 대해 git으로 형상관리를 하며 해당 내용을 도커 이미지로 만들고 해당 배포에 대해 젠킨스를 통하여 자동배포가 이루어지고 있는 시스템 구성도입니다.

- 본 글에서는 개념설명이 위주가 아닌 실습을 위주로 하고 있습니다.

#### 예제 어플리케이션 살펴보기

- 깃 주소

- 프로젝트 간략 설명

- 시스템 구성도


#### 

1. 깃허브에 pntbiz-react-docker라는 레퍼지토리를 추가하였습니다. 아래의 링크는 해당 레퍼지토리 주소입니다.

###### [도커실습 깃허브 주소](https://github.com/SayRew/pntbiz-react-docker)

2. 해당 레퍼지토리에는 react + node 로 구성되어 있는 소스코드입니다. 해당 react 소스 코드는 "React-2부 [상태관리]" 글에서 사용한 react 프로젝트에서 01번 예제만을 복사하였습니다. 아래의 링크는 해당 글의 링크입니다.

###### [React-2부 상태관리](https://tech-people.github.io/2019/12/29/react-hooks-chapter02/)

3. 해당 프로젝트 package.json 파일을 잠시 보겠습니다. 

package.json

```

```

"React-2부 상태관리" 예제의 소스코드를 그대로 갖고 왔습니다. 변경사항으로는 node의 express 모듈과 ejs 모듈이 추가되어 react로 작성한 프론트 산출물을 실제 서버를 실행하여 브라우저로 확인할 수 있다는 점입니다.

server.js

```

```

server.js 입니다. 기본적으로 3000번 포트로 서버는 열립니다.


이렇게 설정함으로서 하나의 이미지를 여러개의 컨테이너로 실행하였을 때 브라우저로 구분할 수 있습니다.

Dockerfile

```

```

도커 파일은 도커의 이미지를 생성하는 명세서 역할을 합니다. 해당 파일로 도커의 이미지를 생성합니다.


이번 실습을 위해 AWS EC2 인스턴스 2개를 미리 준비해 놓았습니다. 본 글에서는 EC2 인스턴스를 생성하는 방법에 대해서는 다루지 않습니다. 해당 내용에 대해 아래의 링크를 참고해주세요.

###### [EC2 생성하기](https://github.com/SayRew/pntbiz-react-docker)

참고 : AWS 리전은 서울 리전을 사용하며 EC2의 ssh 키는 1개로 동일한 키를 사용중입니다. 

그럼 바로 실습을 시작해보겠습니다.


## 자동화 배포 서버

지금부터 bash 명령어를 통해 설명을 하도록 하겠습니다.

1. root 비밀번호 설정
```
설명 : ssh 접속을 하여 root 비밀번호를 설정합니다. 본 글에서는 pntbiz09로 설정하였습니다.
[ec2-user@ip-172-31-9-194 ~]$ sudo passwd
Changing password for user root.
New password: 
Retype new password: 
passwd: all authentication tokens updated successfully.
```

2. 자바 버전 8로 다운로드

```
설명 : root로 접속을 합니다.
[ec2-user@ip-172-31-9-194 ~]$ su root

설명 : yum 패키지를 최신으로 업데이트 합니다.
[root@ip-172-31-9-194 ec2-user]#  yum -y update
... 

설명 : 자바 버전을 확인합니다. 젠킨스는 java 8이 필요합니다.
[root@ip-172-31-9-194 ec2-user]# java -version
java version "1.7.0_231"
OpenJDK Runtime Environment (amzn-2.6.19.1.80.amzn1-x86_64 u231-b01)
OpenJDK 64-Bit Server VM (build 24.231-b01, mixed mode)

설명 : 자바 7 버전을 삭제합니다.
[root@ip-172-31-9-194 ec2-user]# yum -y remove java-1.7.0-openjdk

설명 : 자바 버전을 확인합니다. 기존에 존재하던 자바 7버전을 삭제하였으므로 존재하지 않습니다.
[root@ip-172-31-9-194 ec2-user]# java -version
bash: java: command not found

설명 : 자바 8 버전을 다운로드 합니다.
[root@ip-172-31-9-194 ec2-user]# yum -y install java-1.8.0
...

설명 : 자바 8 버전이 정상적으로 다운로드 된 것을 확인할 수 있습니다.
[root@ip-172-31-9-194 ec2-user]# java -version
openjdk version "1.8.0_222"
OpenJDK Runtime Environment (build 1.8.0_222-b10)
OpenJDK 64-Bit Server VM (build 25.222-b10, mixed mode)
```

3. git 설치
```
설명 : 깃을 설치합니다.
[root@ip-172-31-9-194 ec2-user]# yum -y install git
```

4. 젠킨스 설치 및 실행

```
설명 : 젠킨스를 다운로드 하기 위한 레퍼지토리를 추가합니다. -o 옵션은 해당 메세지를 파일로 생성합니다.
젠킨스를 다운로드 하기 위한 레퍼지토리를 추가합니다. -o 옵션은 해당 메세지를 파일로 생성합니다.
[root@ip-172-31-9-194 ec2-user]# wget -O /etc/yum.repos.d/jenkins.repo http://pkg.jenkins-ci.org/redhat/jenkins.repo

설명 : rpm 포맷은 전자서명을 첨부할 수 있으므로 위변조 여부를 확인할 수 있습니다. 해당 검증용 키를 추가합니다.
[root@ip-172-31-9-194 ec2-user]# rpm --import http://pkg.jenkins-ci.org/redhat/jenkins-ci.org.key

설명 : 젠킨스를 설치합니다.
[root@ip-172-31-9-194 ec2-user]# yum -y install jenkins

설명 : 젠킨스를 실행합니다.
[root@ip-172-31-9-194 ec2-user]# service jenkins start
Starting Jenkins                                           [  OK  ]

설명 : 설치를 완료하였으면 해당 경로의 파일을 vi 편집기로 열어줍니다. 해당 파일에 최초 젠킨스 접속시 admin 비밀번호가 있습니다.
[root@ip-172-31-9-194 secrets]# vi /var/lib/jenkins/secrets/initialAdminPassword

```

인바운드 규칙 수정

- 젠킨스를 서버에서 실행하였습니다. 해당 서버로 접속하기 위해서는 AWS의 인바운드 규칙을 수정해야 합니다. 젠킨스 기본 포트가 8080이므로 8080포트를 열어줍니다.

![EC2 인바운드 규칙](/images/react/oneFlow.png)


젠킨스 초기 설정

 - 최초 접속을 하면 admin 비밀번호를 입력합니다. 위에서 vi 편집기로 열었던 initialAdminPassword 파일의 내용을 입력해줍니다.

![admin 비밀번호 입력](/images/react/oneFlow.png)

- 다음으로 젠킨스 최초 플러그인 설치에 대한 안내가 나옵니다. 여기서는 "Install suggested plugins"를 선택합니다.

![젠킨스 플러그인 설치](/images/react/oneFlow.png)


![플러그인 설치 중..](/images/react/oneFlow.png)

젠킨스 깃허브 연동

1. 젠킨스 비밀키/공개키 생성

```
설명 : 현재 젠킨스 서버를 실행하고 있는 user 를 확인합니다. 특별히 설정한 부분이 없기에  기본적으로 jenkins 
      사용자가 젠킨스 서버를 실행하고 있습니다.
[root@ip-172-31-9-194 ec2-user]# ps -aux | grep jenkins

설명 : jenkins 사용자는 일반적으로 서버에 등록된 사용자는 아니기에 su -u jenkins로 전환이 불가능합니다.
[root@ip-172-31-9-194 ec2-user]# sudo -u jenkins /bin/bash 
bash-4.2$ 

설명 : 해당 경로에 .ssh 경로를 만들고 이동합니다.
bash-4.2$ mkdir /var/lib/jenkins/.ssh
bash-4.2$ cd /var/lib/jenkins/.ssh

설명 : ssh 키를 생성합니다. -t는 압호화 방식이고 -f는 파일 이름을 설정합니다.
bash-4.2$ ssh-keygen -t rsa -f /var/lib/jenkins/.ssh/github-jenkins

설명 : ls -al 옵션으로 해당 디렉토리에 비밀키/공개키가 생성이 잘 되었는지 확인합니다.
bash-4.2$ ls -al
total 16
drwxr-xr-x  2 jenkins jenkins 4096 Jan  5 04:57 .
drwxr-xr-x 15 jenkins jenkins 4096 Jan  5 04:55 ..
-rw-------  1 jenkins jenkins 1675 Jan  5 04:57 github-jenkins
-rw-r--r--  1 jenkins jenkins  405 Jan  5 04:57 github-jenkins.pub

```

2. 깃허브에 공개키 등록


- 배포할 레퍼지토리로 이동하여 Settings로 이동합니다.

![배포할 레퍼에 셋팅즈로 이동](/images/react/oneFlow.png)

- 여기서 Deploy Keys로 한번더 이동하여 "Add deploy key"를 선택합니다.

![add키 마우스 오버](/images/react/oneFlow.png)

- 여기서 Deploy Keys로 한번더 이동하여 "Add deploy key"를 선택합니다.

- 다시 배포 서버의 ssh 콘솔로 이동하여 아래의 명령어를 실행합니다.

```
설명 : 공개키 정보를 읽습니다.
bash-4.2$ cat github-jenkins.pub 
.... (키 값이 나옵니다. 메모장에 복사합니다.)
```

- 해당 키의 값을 복사하여 깃허브의 key값 안에 넣어줍니다.

![키 값 넣은 모습](/images/react/oneFlow.png)

- 정상적으로 키가 등록된 것을 확인할 수 있습니다.

![키가 정상적으로 등록된 모습](/images/react/oneFlow.png)

3. 젠킨스의 비밀키 등록

- 젠킨스에서 Credentials > System > Global credentials 로 이동합니다.

![비밀키 등록하는 사진 .. 약간 좀 손봐야함](/images/react/oneFlow.png)

- Add Credentials를 선택합니다. 그리고 다시 배포서버의 ssh 콘솔창에 아래오 같은 명령어를 입력합니다.

```
bash-4.2$ cat github-jenkins
... (키 값이 나옵니다.메모장에 복사합니다.)
```

- Kind는 인증 방식입니다. Username 은 젠킨스 Job에서 보여줄 인증키 이름입니다.

![입력한 모습](/images/react/oneFlow.png)

4. 젠킨스와 깃허브 연동 테스트

- 젠킨스 메인 페이지로 돌아와 새로운 Item을 추가합니다. item은 Freestyle project로 선택하며 프로젝트 이름은 재량으로 설정합니다.

![새롭게 추가하는 모습](/images/react/oneFlow.png)

- 여러개의 설정 중에 "소스 코드 관리"라는 탭으로 이동합니다. 여기서 깃허브 주소를 넣고 위에서 jenkins에 등록한 키를 선택합니다. Branch는 master이므로 */master로 입력합니다. 다 완룍

![젠킨스에서 깃허브 설정한 사진](/images/react/oneFlow.png)

- "Build Now"를 선택하여 빌드를 진행합니다. 정상적으로 완료가 되면 파랑색 불이 들어오며 화살표를 클릭하여 "Console Output"을 선택하여 결과를 확인할 수 있습니다.

![Console Output을 선택하는 사진](/images/react/oneFlow.png)

![Console Output 사진](/images/react/oneFlow.png)

- 다시 배포서버 ssh 콘솔로 이동하여 아래의 명령어를 실행합니다.

```
설명 : 다시 root 계정으로 접속합니다.
bash-4.2$ su root
Password:

설명 : 기본적으로 젠킨스는 /var/lib/jenkins/workspace/ 경로에 깃허브로 부터 파일을 저장합니다.
[root@ip-172-31-9-194 .ssh]# cd /var/lib/jenkins/workspace/
[root@ip-172-31-9-194 workspace]# ll
total 8
drwxr-xr-x 4 jenkins jenkins 4096 Jan  5 05:20 docker-github-jenkins
drwxr-xr-x 2 jenkins jenkins 4096 Jan  5 05:20 docker-github-jenkins@tmp
```

도커 설치 및 컨테이너 실행하기

1. 도커 설치

```
설명 : 도커를 설치합니다.
[root@ip-172-31-9-194 workspace]# yum -y install docker
.. (설치 관련내용 출력..)

설명 : 도커를 실행합니다.
[root@ip-172-31-9-194 workspace]# service docker start
```

2. 젠킨스를 통한 도커 빌드

젠킨스로 이동하여 위에서 생성한 item으로 이동합니다. 이후 item의 구성으로 이동하여 Build의 Execute shell을 선택합니다. 그러면 쉘 스크립트를 실행을 할 수 있으며 실행되는 위치는 해당 깃허브에서 내려받은 디렉토리 경로인 /var/lib/jenkins/workspace/ 이 됩니다.

![Console Output 사진](/images/react/oneFlow.png)

도커를 빌드하는 스크립트를 작성해줍니다. 작성이 완료되면 현재 docker 명령어는 root 권한으로만 실행이 가능하므로 jenkins 유저에 sudo 권한을 부여하기 위해 아래의 명령어를 실행합니다.

```
설명 : sudo 권한을 추가할 파일입니다. vi로 열어줍니다.
[root@ip-172-31-9-194 docker-github-jenkins]# vi /etc/sudoers

... 내용을 보다보면 아래와 같은 내용에 jenkins 유저도 추가해줍니다. 
## Allow root to run any commands anywhere
root    ALL=(ALL)       ALL
jenkins ALL=(ALL) NOPASSWD: ALL   <== 추가한 내용
```



 


docker rm -f 'docker ps -a' 도커 컨테이너 전부 삭제
docker ps -a : 실행중인 모든 컨테이너 확인
docker images : 이미지 확인
docker rfi -f 'docker images'

docker stop $(docker ps -a -q) : 도커 컨테이너 전부 중지
docker rm $(docker ps -a -q) : 도커 컨테이너 전부 삭제
docker rmi -f $(docker images -q) : 도커 이미지 전부 삭제

docker build -t pntbiz .

docker run -d -p 3000:3000 pntbiz-docker
 
docker exec -it  66bca6a55d53 /bin/bash
 
sudo usermod -aG docker $USER
> 작성자 : 플랫폼 개발실

docker pull prorew94/pntbiz-docker

docker push prorew94/pntbiz-docker

docker tag pntbiz prorew94/pntbiz-docker

##### 출처
  - 출처