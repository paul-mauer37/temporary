# SELinux





SELinux 이해를 위해 필수적인 접근 통제에 대해 알아보자.



**OS에서 접근통제(Access Control)**는 **디렉토리나 파일, 네트워크 소켓과 같은 시스템 자원**을 적절한 권한을 가진 **사용자나 그룹**이 접근하고 사용할 수 있도록 통제하는 것을 의미한다. 

* 시스템 자원 - 객체 (Object)

* 자원에 접근하는 사용자 혹은 프로세스 - 주체 (Subject)

```bash
# example)
/etc/passwd 파일 자체는 객체(Object)
암호변경 명령어 passwd는 주체(Subject)
```



이러한 접근 통제의 방법 중 대부분의 유닉스나 윈도우 등 운영체제에서 사용하는 모델이 임의 접근 통제(DAC; Discretionary Access Control)이다.

사용자나 그룹이 객체의 소유자라면 다른 주체에 대해 해당 객체에 대한 접근 권한을 설정할 수 있다. 임의적이라는 말은 소유자의 판단에 의해 권한을 줄 수 있다는 뜻이다. 사용이 간편하고 구현이 용이한 장점이 있지만, 만약 모든 권한을 가진 root 계정이 탈취당한다면 시스템이 장악될 수 있다는 위험도 존재한다.



유닉스 계열 OS에서는 실행 파일의 속성에 setuid(set user ID upon execution)또는 setgid(set group ID upon execution) 비트라는 것을 설정할 수 있다. 이 비트가 설정되어 있을 경우 해당 실행 파일을 실행할 때, 설정된 사용자(setuid) 또는 그룹(setgid) 권한으로 동작한다.

즉, passwd명령어와 ping명령어의 경우, 실행시 root 권한이 필요한데, 소유자를 root로 하고 setuid 비트를 설정하면 실행 시점에 root 사용자로 전환되게 된다. setuid, setgid 비트를 통해 root만 가능한 동작을 수행할 수 있게 된다.

```bash
$ ls -l /bin/ping /usr/bin/passwd
-rwxr-xr-x. 1 root root 40760  2월  20  2020 /bin/ping
-rwsr-xr-x. 1 root root 30768  2월  20  2020 /usr/bin/passwd
--- 사용자 퍼미션 부분의 's'표시는 setuid 비트
--- 그룹 퍼미션 부분의 's'표시는 setgid 비트

# setuid 비트가 설정된 프로그램 찾기
$ find /bin /usr/bin /sbin -perm -4000 -exec ls -ldb {} \;

# 비트가 설정된 프로그램을 이용하면 공격자는 손쉽게 root 권한을 획득할 수 있다.
```



---



well-known port(잘 알려진 포트)는 특정한 쓰임새를 위해 IANA에서 할당한 TCP 및 UDP 포트 번호의 일부로, 1024 미만의 포트 번호를 갖게 된다. 예를 들어, 웹에 사용되는 http(80), 메일 전송에 사용되는 smtp(25), 파일 전송에 사용되는 ftp(20, 21) 등이 well-known port입니다.

well-known port는 root만이 사용할 수 있으므로 서비스 데몬 역시 root 권한으로 기동된다. 여기서 서비스 데몬에 보안 취약점이 있거나 잘못된 설정이 있을 경우, 서비스 데몬을 통해 공격자는 root 권한을 획득하게 된다. 



---



강제 접근 통제(MAC; Mandatory Access Control)는 미리 정해진 정책과 보안 등급에 의거하여 주체에게 허용된 접근 권한과 객체에게 부여된 허용 등급을 비교하여 접근 통제하는 모델이다. 높은 보안을 요구하는 정보는 낮은 보안 수준의 주체가 접근할 수 없으며 소유자라고 할지라도 정책에 어긋나면 객체에 접근할 수 없으므로 강력한 보안을 제공한다.

MAC정책에서는 root로 구동한 http 서버라도 접근 가능한 파일과 포트가 제한된다. 다시 말해, 취약점을 이용해 httpd의 권한을 획득했어도 /var/www/html, /etc/httpd 등의 사전에 허용한 폴더에만 접근 가능하며 80, 443, 8080 등의 포트만 접근이 허용되므로, ssh로 다른 서버로 접근을 시도하는 등의 2차 피해가 최소화된다. 다만, 구현이 복잡하고 모든 주체와 객체에 대해 보안 등급과 허용 등급을 부여해야 하므로 설정이 복잡하고 시스템 관리자가 통제 모델에 대해 잘 이해하고 있어야 한다.



SELinux

SELinux는 기존 접근 통제 규칙보다 먼저 동작하므로 SELinux의 보안 정책에 맞지 않을 경우 차단해 버린다.

* 사전 정의된 접근 통제 정책 탑재

  사용자, 역할, 타입, 레벨 등의 다양한 정보를 조합하여 어떤 프로세스가 어떤 파일, 디렉토리, 포트 등에 접근 가능한지에 대해 사전에 잘 정의된 접근 통제 정책을 제공하기 때문에 MAC(강제 접근 통제) 적용을 위해 시스템 관리자가 할 일이 대폭 줄고 애플리케이션의 변경없이 setuid와 1024 이하 포트를 사용하는 데몬을 안전하게 사용할 수 있다.

* "Deny All, Permit Some"정책으로 잘못된 설정 최소화

  "모든 걸 차단하고 필요한 것만 허용"하는 정책으로 보안 정책이 사전에 설정되어 있으므로 잘못된 설정이 기본으로 포함돼있을 여지가 적다. 

* 권한 상승 공격에 의한 취약점 감소

  setuid비트 설정이나 root로 실행되는 프로세스처럼 위험한 프로그램들은 샌드박스 안에서 별도의 도메인으로 격리되어 실행되므로, root 권한을 탈취당해도 해당 도메인에만 영향을 미치고 전체 시스템에 미치는 영향은 최소화된다. 예를 들어, 아파치 httpd 서버의 보안 취약점을 통해 권한을 획득했어도 아파치같은 서버 데몬은 낮은 등급의 권한을 부여받으므로 공격자는 일반 사용자의 홈 디렉토리를 읽을 수 없고, /tmp 임시 디렉토리에 파일을 쓸 수 없다.

* 잘못된 설정과 버그로부터 시스템 보호

  잘못된 설정이나 신뢰할 수 없는 입력을 악용한 공격에서 프로세스를 보호할 수 있다. 예를 들어, 버퍼의 입력 길이 등을 제대로 체크하지 않아서 발생하는 buffer overflow attack(버퍼 오버 플로 공격)의 경우, SELinux가 애플리케이션 메모리에 있는 코드를 실행할 수 없게 통제하므로 데몬 프로그램에 buffer overflow 버그가 있어도 쉘을 얻을 수 없다.



---



SELinux는 커널의 기본 기능으로 동작하며, 모든 시스템 콜이 보안 정책을 확인해 접근 허용 여부를 판단하며 빠르게 처리하기 위해서 보안 정책은 AVC(Access Vector Cache)라는 이름으로 커널 내부에서 캐싱한다.



SELinux는 enforce, permissive, disable 3가지 모드가 있으며, 기본 설정은 enforce mode로 보안 정책에 위배되는 모든 액션이 차단됩니다. permissive mode는 경고 메시지를 내고 차단하지는 않는다. 

```
SELinux 사용 여부 확인
# sestatus
---
SELinux status:                 enabled
SELinuxfs mount:                /sys/fs/selinux
SELinux root directory:         /etc/selinux
Loaded policy name:             targeted
Current mode:                   enforcing
Mode from config file:          enforcing
Policy MLS status:              enabled
Policy deny_unknown status:     allowed
Max kernel policy version:      28

SELinux 모드 전환
# setenforce 0
--- enforce mode
# setenforce 1
--- permissive mode

SELinux를 끄는 것은 권장하지 않지만, /etc/selinux/config의 SELinux 항목을,
SELINUX=disabled
와 같이 설정하면 된다.
```

> SELinux를 비활성화했다가 다시 활성화할 때는 재부팅이 필요하다. 모든 리소스에 대해 보안 레이블을 추가해야 하므로 부팅 시간이 오래 걸릴 수 있다.



* Security Context : SELinux는 모든 프로세스와 객체마다 보안 컨텍스트(Security Context)라고 부르는 정보를 부여하여 관리하고 있다. Security Context는 접권 권한을 확인하는데 사용되며 다음 4가지 구성 요소로 이루어져 있다.
  * 사용자 : 시스템 사용자와는 별도의 SELinux 사용자이며, 역할이나 레벨과 연계하여 접근 권한을 관리하는데 사용.
  * 역할(Role) : 하나 이상의 타입과 연결되어 SELinux 사용자의 접근을 허용할 지 결정하는데 사용.
  * **타입(Type)** : **Type Enforcement**의 속성 중 하나로 프로세스의 도메인이나 파일의 타입을 지정하고 이를 기반으로 접근 통제를 수행.
  * 레벨(Level) : 레벨은 MLS(Multi Level System)에 필요하며 강제 접근 통제보다 더 강력한 보안이 필요할때 사용하는 기능.

4가지 구성 요소 중 핵심 부분은 타입이며 꼭 알아야 한다. 

SELinux를 활성화하면 파일이나 디렉토리 등의 객체마다 보안 컨텍스트를 부여하는데, SELinux가 되입되면서 `ls`, `ps`, `cp`, `mv` 등의 유틸리티에는 `-Z`, `--context` 옵션이 추가되었다. 

```bash
ls -ldZ /var/www
---
drwxr-xr-x. centos centos unconfined_u:object_r:httpd_sys_content_t:s0 /var/www
--- unconfined_u : 사용자 / object_r : 역할 / httpd_sys_content_t : 타입 / s0 : 레벨

ps -Z | grep httpd
---
system_u:system_r:httpd_t:s0    15638 ?        00:00:38 php-fpm
system_u:system_r:httpd_t:s0    15640 ?        00:00:05 php-fpm
--- httpd_t : 보안 컨텍스트(Security Context)
```



* Type Enforcement

TE(Type Enforcement)는 SELinux의 기본적인 접근 통제를 처리하는 매커니즘으로 주체가 객체에 접근하려고 할 때 주체에 부여된 보안 컨텍스트가 객체에 접근할 권한이 있는지 판단하는 역할을 수행한다.

예를 들어, 아파치 웹 서버(httpd)가 /var/www/html/ 에 접근하려고 할 때, 아파치 웹 서버(httpd)는 주체(subject)가 되며 아파치 웹 서버에 부여된 보안 컨텍스트는 httpd_t가 된다. 객체에 부여된 보안 컨텍스트는 httpd_sys_content_t 이며 httpd_t 는 httpd_sys_content_t 보안 컨텍스트가 부여된 객체에 접근이 허용되므로 아파치 웹 서버는 /var/www/html/ 디렉토리에 있는 콘텐츠를 읽을 수 있다. 



** __Security Context__ 와 __Type Enforcement__에 따르면,

사전에 탑재된 httpd 에 대한 정책에 의해 httpd_sys_content_t가 붙은 컨텐츠만 읽을 수 있고, /var/www 아래에 파일을 생성하면 자동으로 httpd_sys_content_t 가 붙게 된다. 그러므로 /data/myweb-app 폴더를 만들고 이 안에 웹 서비스할 파일을 넣고 httpd에 DocumentRoot를 설정해도 SELinux는 미리 허용된 경로가 아니므로 차단시켜서 permission denied error가 발생한다.

마찬가지로 httpd가 접근할 수 있게 사전에 허용된 context는 http_port_t이며 semanage 명령어로 해당 포트 목록을 조회할 수 있다.

```bash
semanage port -l | grep http_port_t

# 기본적으로 허용된 포트는 아래처럼 80, 443, 9000, 8000 등이다.
# http_port_t			tcp		9004, 8000, 8080, 10080, 8001, 80, 81, 443, 488, 8008, 8009, 8443, 9000
```

Type Enforcement로 인해 공격자가 "제로 데이 취약점"을 사용하여 httpd의 권한을 획득해도 다른 서버로 ssh 연결을 할 수 없다. 22번 포트는 허용되지 않았기 때문이며 덕분에 2차 피해를 최소화할 수 있다.

 

또 다른 예로, 파일 공유 서비스와 mysql DBMS가 같이 구동되는 서버에서 파일 공유 서비스를 공격하여 권한을 탈취했다고 가정했을때, 공격자는 mysql DB 파일을 가져가려고 시도할 수 있다. 

```bash
ls -lZ /var/lib/mysql/

-rw-rw----. mysql mysql system_u:object_r:mysqld_db_t:s0 ib_logfile0
-rw-rw----. mysql mysql system_u:object_r:mysqld_db_t:s0 ib_logfile1
-rw-rw----. mysql mysql system_u:object_r:mysqld_db_t:s0 ibdata1
```

SELinux 하에서는 파일 공유 서비스와 mysql은 별도의 도메인으로 격리되어 동작하며 mysql 데이터는 mysqld_db_t 보안 컨텍스트가 설정되어 있어 해당 객체에 접근할 수 없다. 이때, 웹 서버와 php-fpm을 연동하는데 포트를 기본 fpm 포트(9000)가 아닌 9001을 사용했거나 톰캣과 연동하는데 8009, 8000이 아닌 포트를 사용했다면 SELinux가 이를 차단해서 서비스가 동작하지 않는다. 



---



### SELinux 원활하게 사용하기

#### 1. context 정보 얻기

SELinux 문제를 해결하기 위해서는 context 정보를 확인하고 이를 맞춰주는게 중요하다. 이를 위해 context 정보를 확인할 수 있는 유틸리티 사용법을 알아보자.

```bash
$ yum install setools-console

# seinfo는 policy 조회하고 출력하는 명령어
$ seinfo

# seindo -a 옵션으로 조회할 속성 지정 가능
$ seinfo -adomain -x		
--- 전체 도메인 출력

# sesearch 지정한 룰 조회
$ sesearch --role_allow -t httpd_sys_content_t		
--- httpd_sys_content_t 객체에 접근 가능한 롤 그룹 표시

# sesearch --allow 옵션으로 특정 context에 허용된 액션을 알 수 있다.
$ sesearch --allow -s httpd_t
---
allow httpd_t httpd_sys_content_t : file { ioctl read getattr lock open } ; 
allow httpd_t zoneminder_log_t : file { ioctl getattr lock append open } ; 
allow httpd_t httpd_sys_content_t : dir { ioctl read getattr lock search open } ; 
allow httpd_t zoneminder_log_t : dir { getattr search open } ;
---
# httpd_t는 httpd_sys_content_t가 설정된 파일에 대해 [ioctl, read, getattr, lock, open] system call이 가능합니다. 즉, httpd_t는 httpd_sys_content_t 컨텐츠를 읽을 수 있다.
```



#### 2. 수정

만약 mysql을 3307 포트로 구동했다면 허용된 포트가 아니기 때문에 web 서버가 mysql에 연결할 수 없으며 허용된 포트를 사용하거나 정책을 수정하는 semanage명령어로 변경된 정보를 SELinux에게 알려주면 된다.

semanage는 SELinux의 보안 정책을 조회하고 [추가/변경/삭제] 할 수 있는 명령행 기반 유틸리티이다.

앞서, 보안 컨텍스트에는 파일, 네트워크 포트, 네트워크 인터페이스 등이 있으며, 서비스 데몬이 SELinux 에서 문제없이 동작하려면 필수적으로 알아두어야 할 것이 바로 **파일**과 **네트워크 포트** 컨텍스트이다.

```bash
$ yum install -y policycoreutils-python

# httpd가 연결할 수 있는 포트 정보 확인
$ semanage port -l | grep http_port_t
---
http_port_t			tcp		8081, 8080, 8090, 80, 81, 443, 488, 8008, 8009, 8443, 9000
--- semanage 명령어의 첫번째는 컨텍스트 이름이 와야하며(여기선 포트), Object 리스트를 보는 -l 옵션을 추가한다.

### 위 상황에서 WAS가 9876 포트를 사용한다면 등록되지 않은 포트이므로, SELinux는 웹 서버가 9876 포트에 연결하지 못하도록 차단할 것이다. 이때, semanage 명령어로 포트를 추가해 주면 웹 서버가 9876 포트에 연결할 수 있다.
$ semanage port -a -p tcp -t http_port_t 9876

# 포트 사용중 에러 발생시 : /usr/sbin/semanage: tcp/9876에 대한 포트가 이미 지정되었습니다
$ semanage port -m -p tcp -t http_port_t 9876
--- [추가/변경/삭제] 옵션은 [-a/-m/-d]
```



파일 컨텍스트는 fcontext 아규먼트를 이용할 수 있다.

```bash
# semanage fcontext -l | grep httpd_sys_content_t
---
/var/lib/htdig(/.*)?		all files		system_u:object_r:httpd_sys_content_t:s0 
/var/lib/trac(/.*)?			all files		system_u:object_r:httpd_sys_content_t:s0 
/var/www(/.*)?				all files		system_u:object_r:httpd_sys_content_t:s0 
/var/www/icons(/.*)?		all files		system_u:object_r:httpd_sys_content_t:s0 
/var/www/svn/conf(/.*)?		all files		system_u:object_r:httpd_sys_content_t:s0
--- /var/www(/.*)?은 /var/www 하위에 생성되는 모든 파일과 디렉토리는 httpd_sys_content_t*를 붙이라는 의미

# 웹 서버 콘텐츠가 /opt/mycontent 에 있다면 해당 디렉토리에 존재하는 모든 파일과 디렉토리에 httpd_sys_content_t를 붙여야 정상적으로 서비스가 가능하므로 semanage fcontext를 지정한다.
$ semanage fcontext -a -t httpd_sys_content_t "/opt/mycontent(/.*)?"
```







* etc

SELinux 사용시 문제가 발생하는 원인과 로그가 쌓이는 위치를 몰라 메시지 해석이 어렵다.

SELinux 관련 로그는 /var/log/audit/audit.log에 저장되며 root만 볼 수 있다.

```bash
# SELinux의 차단 이유에 대한 audit 로그를 출력
$ audit2why < /var/log/audit/audit.log
---
type=AVC msg=audit(1463625135.602:32366): avc:  denied  { name_connect } for  pid=25862 comm="nginx" dest=9001 scontext=unconfined_u:system_r:httpd_t:s0 tcontext=system_u:object_r:tor_port_t:s0 tclass=tcp_socket

        Was caused by:
        The boolean httpd_can_network_connect was set incorrectly. 
        Description:
        Allow HTTPD scripts and modules to connect to the network using TCP.

        Allow access by executing:
        # setsebool -P httpd_can_network_connect 1
--- 결과는 다음과 같이 차단된 process와 원인, 해결책 등을 제시한다.
```



audit2why 는 모든 로그를 보여주기 때문에 로그가 많이 쌓여있을 경우 보기가 어렵다. 커널 로그를 검색할 수 있는 ausearch 명령어를 사용하면 기간별, 타입별로 로그를 조회할 수 있다.

```bash
$ yum install audit

# 2016년 5월 13일 이후 발생한 이벤트만 표시
$ ausearch -m AVC,USER_AVC -ts 05/13/2016
--- -m : 메시지 타입(콤마를 이용해 여러 개 지정 가능) / -ts 또는 -start : 시작 날짜 지정

# 이번 달 발생 로그
$ ausearch -m AVC,USER_AVC -ts this-month
--- now, recent, today, this-week, this-month, this-year 가능

```



만약 audit2why 와 ausearch 가 출력하는 이벤트 로그가 익숙하지 않다면, 이를 쉽게 번역해주는 유틸리티가 존재한다. sealert를 이용해보자.

```bash
$ yum install setroubleshoot-server

# 현재 이벤트 로그 분석 (-a 옵션)
$ sealert -a /var/log/audit/audit.log
---
SELinux is preventing /usr/sbin/nginx from name_connect access on the tcp_socket .

*****  Plugin catchall_boolean (89.3 confidence) suggests  *******************

If you want to allow HTTPD scripts and modules to connect to the network using TCP.
Then you must tell SELinux about this by enabling the 'httpd_can_network_connect'boolean.
Do
setsebool -P httpd_can_network_connect 1
---
```



ref) https://lesstif.gitbooks.io/web-service-hardening/content/selinux.html#semanage























