

### Docker(도커)

도커는 **컨테이너 기반 오픈소스 가상화 플랫폼** 이다. 다양한 **프로그램, 실행환경**을 **컨테이너로 추상화**하고 동일한 인터페이스를 제공하여 프로그램의 배포 및 관리를 단순하게 해준다. 

* **백엔드 프로그램, 데이터베이스 서버, 메시지 큐 등 어떤 프로그램이든 컨테이너로 추상화**할 수 있고, 
* **조립PC, AWS, Azure, Google cloud 등 어디에서든 실행**할 수 있다.



#### 컨테이너

컨테이너는 **격리된 공간**에서 **프로세스가 동작**하는 기술로 가상화 기술의 하나지만 기존의 방식과는 차이가 있다.



* 기존의 가상화 방식 : **OS 가상화**

VMware, VirtualBox 같은 가상머신은 호스트 OS 위에 **게스트 OS 전체를 가상화**하여 사용한다. 호스트형 가상화 방식으로 무겁고 느리다.



* 발전

CPU의 가상화 기술(HVM)을 이용한 KVM(Kernel-based Virtual Machine)과 **반가상화**(Paravirtualization) 방식의 Xen이 등장했다. 이 방식은 **게스트 OS**가 필요하지만 **전체를 가상화하지 않아** 성능이 향상되었다. 

>  OpenStack, AWS, Rackspace 같은 클라우드 서비스에서 가상 컴퓨팅 기술의 기반이 되었다.



* Docker(도커)

여기서 더 나아가, **프로세스를 격리**하는 방식이 등장한다.

리눅스에서는 이것을 **리눅스 컨테이너**라고 하며 단순히 **프로세스를 격리**시키기 때문에 가볍고 빠르게 동작한다.

1.  하나의 서버에 **여러 개의 컨테이너**를 실행하면 서로 영향을 미치지 않고 **독립적으로 실행**되어 마치 가벼운 VM(virtual Machine)을 사용하는 느낌을 준다. 

2. **실행중인 컨테이너에 접속**하여 명령어를 입력할 수 있고 `apt-get`이나 `yum`으로 패키지를 설치할 수 있으며 

3. **사용자도 추가**하고 **여러 개의 프로세스를 백그라운드로 실행**할 수도 있습니다. 

4. **CPU나 메모리 사용량을 제한할 수 있고** 

5. **호스트의 특정 포트와 연결하거나 호스트의 특정 디렉토리를 내부 디렉토리인 것처럼 사용할 수도** 있습니다.



이러한 컨테이너 개념은,

리눅스 - cgroups(control groups)와 namespace를 이용한 LXC(Linux container)

FreeBSD - Jail

Solaris - Solaris Zones

Google - Lmctfy(Let Me Contain That For You)

도커 - LXC기반으로 시작해, 0.9버전에서는 자체적인 libcontainer 기술을 사용, 추후 runC기술에 합쳐졌다.



#### 이미지

이미지는 컨테이너 실행에 필요한 **파일과 설정값 등을 포함**하고 있는 것으로, 상태값을 갖지 않고 변하지 않는다(immutable).

컨테이너는 이미지를 실행한 상태라고 볼 수 있고, 추가되거나 변하는 값은 컨테이너에 저장된다. 

같은 이미지에서 여러 개의 컨테이너를 생성할 수 있고 컨테이너의 상태가 바뀌거나 삭제되더라도 이미지는 변하지 않고 그대로 남아있다.



예를 들어, **ubuntu이미지**는 **ubuntu를 실행하기 위한 모든 파일**을 갖고 있고, **MySQL이미지**는 debian을 기반으로 **MySQL을 실행하는데 필요한 파일과 실행 명령어, 포트 정보 등**을 갖고 있다. 더 복잡한 예로, **Gitlab이미지**는 centos를 기반으로 **ruby, go, database, redis, gitlab source, nginx 등**을 갖고 있다.

다시 말해, 이미지는 컨테이너를 실행하기 위한 모든 정보를 지녔다. 따라서 더 이상 의존성 파일을 컴파일하고 추가로 이것저것 설치할 필요가 없다. 새로운 서버가 추가되면 미리 만들어 놓은 이미지를 다운받고 컨테이너를 생성하기만 하면 된다. 한 서버에 여러 컨테이너를 실행할 수 있고, 수백, 수천 대의 서버도 문제없다.



도커는 컨테이너, 오버레이 네트워크(overlay network), 유니온 파일 시스템(union file systems)등의 기술을 이용한다. 레이어(layer)개념을 사용해, 여러 개의 레이어를 하나의 파일 시스템으로 사용할 수 있게 한다. 즉, 이미지는 여러 개의 읽기 전용(read only)레이어로 구성되고 파일이 추가되거나 수정되면 새로운 레이어가 생성된다. 예를 들어, ubuntu이미지가 `A`+`B`+`C` 의 집합이라면, ubuntu이미지를 베이스로 만든 nginx이미지는 `A`+`B`+`C`+`nginx`가 된다. webapp이미지를 nginx이미지 기반으로 만들었다면, `A`+`B`+`C`+`nginx`+`source`가 된다. 이 때, webapp 소스를 수정하면 `A`, `B`, `C,` `nginx`레이어를 제외한 `source(v2)`레이어만 다운받으면 된다.

참고로, 컨테이너를 생성할 때도 레이어 방식을 사용하는데, 기존의 이미지 레이어 위에 읽기/쓰기(read-write)레이어를 추가하여 기존 이미지 레이어를 그대로 사용하면서 컨테이너가 실행되는 중에 생성하는 파일이나 변경된 내용은 읽기/쓰기 레이어에 저장되므로 여러 개의 컨테이너를 생성해도 최소한의 용량만 사용하게 된다.

> cf) 단순한 설계로 보일 수 있지만, 가상화의 특성상 이미지 용량이 크고 여러 대의 서버에 배포하는걸 감안하면 굉장히 효율적이며 영리한 설계라는 것을 알 수 있다.







이미지 링크~~~~~~~~~

ref) https://subicura.com/2017/01/19/docker-guide-for-beginners-1.html













WSL2(Windows Subsystem for Linux 2)는 간단히 말해, 리눅스 애플리케이션을 실행하기 위한 프레임워크이다. 과거 Windows OS에서 Docker를 사용하기 위해선 Virtual Machine이 필요했는데 최근엔 Windows 10 Home 또는 Pro에서 WSL2를 지원하므로 따로 가상 머신을 설치할 필요가 없다.











**Docker CLI commands** - https://docs.docker.com/engine/reference/commandline/docker/





 

```dockerfile
###
### 도커에 java + svn 설치하기
###

# 도커에서 원하는 이미지 설치
PS C:\> docker pull asfaltica/centos-java-svn
---
Using default tag: latest
latest: Pulling from asfaltica/centos-java-svn
d5e46245fe40: Pull complete
...
Status: Downloaded newer image for asfaltica/centos-java-svn:latest
docker.io/asfaltica/centos-java-svn:latest

# 도커 이미지 목록
PS C:\> docker images
---
REPOSITORY                    TAG       IMAGE ID          CREATED             SIZE
elleflorio/svn-server       latest    d76927e90a44      3 months ago         49.7MB
asfaltica/centos-java-svn   latest    0b22b9471f3a      3 years ago           418MB

# 도커 컨테이너 생성
PS C:\> docker create -i -t --name centos asfaltica/centos-java-svn

# 도커 프로세스 확인
PS C:\> docker ps -a
---
CONTAINER ID           IMAGE               COMMAND            CREATED               STATUS               PORTS       NAMES
187523463e2b   asfaltica/centos-java-svn    "bash"         5 seconds ago            Created                          centos
3bf100739e24    elleflorio/svn-server       "/init"        21 hours ago     Exited (0) 12 minutes ago              svn-server

# 도커 컨테이너 실행
PS C:\> docker start centos
---
centos

# 도커 컨테이너 들어가기
PS C:\> docker attach centos
---
[root@home /]# find . -name svn*
---
./run/svnserve
./usr/lib/tmpfiles.d/svnserve.conf
...

# SVN 저장소 생성 및 설정
[root@ home]# svnadmin create /home/svn/test_repo
...
svn 설정
...
[root@ home]# svnserve -d -r /home/svn/test_repo --listern-port 3690
---
이 방법은, 컨테이너를 나오면 svn 서버도 닫히게 된다. 그러므로 백그라운드 모드가 필요!!!

# 도커 컨테이너 백그라운드 모드 실행 (detached mode)
1. PS C:\> docker run -dit --name centos -p 3690:3690 asfaltica/centos-java-svn
2. PS C:\> docker run -dit --name centos -v C:/tools.docker/share:/share.host -p 9290:80 -p 3690:3690 asfaltica/centos-java-svn
3. PS C:\> docker run -dit --name centos -v C:/tools.docker/share:/share.host --network="host" -p 3690:3690 asfaltica/centos-java-svn

# docker 들어가기
PS C:\> docker exec -ti centos bash

# svn 서버 기동
svnserve -d -r 디렉로티 --listen-port 3690
```



```dockerfile
###
### 도커에 svn 설치하기
###

# 1. 도커 볼륨 만들기
PS C:\> docker volume create svn-root

# 2. 도커 볼륨에 다운로드 및 실행
PS C:\> docker run -dit --name svn-server -v svn-root:/home/svn -p 7443:80 -p 3690:3690 -w /homw/svn elleflorio/svn-server
# --name 컨테이너이름
# -v 볼륨:디렉토리지정
# -p 외부포트:내부포트 (svn 기본 포트는 3690)
# -w 작업디렉토리
#  이미지 elleflorio/svn-server

# 3. 컨테이너 설정 (docker accrss http protocol)
PS C:\> docker exec -t-svn-server htpasswd -b /etc/subversion/passwd svnadmin [패스워드]

# 4. 저장소 생성 및 설정
PS C:\> docker exec -it svn-server svnadmin create test-repo
...
svn 설정
...

# 5. 접속
PS C:\> docker exec -it svn-server /bin/sh
/home/svn # svn info svn://localhost:3690/test_repo
---
Path: test_repo
URL: svn://localhost/test_repo
...
Revision: 2
...                                               
```







ref) https://blog.naver.com/brainkorea/222065165554

ref) https://blog.naver.com/brainkorea/222060996586

ref) https://blog.naver.com/teja/221558965716

ref) https://gist.github.com/dpmex4527/1d702357697162384d31d033a7d505eb























































































