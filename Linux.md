# Linux



linux에서는 데이터를 읽을 수 있는 자원 혹은 데이터를 쓸 수 있는 대상은 모두 파일로 간주한다. 따라서 디스크에 저장된 파일 뿐만 아니라, 입출력 장치와 같은 것들도 파일처럼 사용할 수 있다.



**파일 종류**

* ordinary file(일반 파일) : 데이터를 갖고 있으며 저장 장치에 저장됨
* directory(디렉토리) : folder(폴더)라고도 하며, 저장 장치에 저장되어 다른 파일들을 조직하고, 사용하는데 필요한 정보를 유지함
* special file(특수 파일) : 물리적인 device(장치)에 대한 내부적 표현 키보드, 모니터, 프린터 등도 파일처럼 사용할 수 있음



디렉토리 자체도 하나의 파일로, 한 디렉토리는 다른 디렉토리들을 포함함으로써 계층 구조를 이루며 부모 디렉토리는 서브 디렉토리를 갖는다.

리눅스 파일 시스템은 root(루트) 디렉토리에서 시작하여 하위 디렉토리들이 형성된다.



**각  서브 디렉토리의 용도**

| 서브 디렉토리 | 용도                                                        | 서브 디렉토리   | 용도                                                         |
| :------------ | ----------------------------------------------------------- | --------------- | ------------------------------------------------------------ |
| `/bin`        | binary의 약자, 실행파일 모음 ex) mv, cat 등 명령어 프로그램 | `/lib`          | 프로그램 라이브러리                                          |
| `/sbin`       | system-binary의 약자, 시스템 관련 명령어 프로그램           | `/var`          | 메일, 시스템 로그, 스풀링, 웹 서비스                         |
| `/etc`        | 환경 설정 파일                                              | `/tmp`          | 임시 저장 용도(시스템 시동 시, 내용 모두 삭제)               |
| `/boot`       | 부팅 관련 파일                                              | `/usr`          | 주로 새로 설치되는 프로그램                                  |
| `/dev`        | device의 약자, 물리적 장치를 가리키는 특수파일              | `/mnt`          | CD-ROM, 네트워크 파일 등 외부장치 마운트                     |
| `/home`       | 사용자 홈 디렉토리                                          | `/cdrom`        | cdrom                                                        |
| `/root`       | root계정 홈 디렉토리                                        | `/lost`+`found` | 훼손된 파일 장소                                             |
| `/include`    | 프로그램에서 사용하는 include 파일                          | `/proc`         | 현재 실행중인 프로세스들이 파일화되어 저장되는 가상 파일 시스템 |
| `/local`      | 시스템 관리자가 소프트웨어 설치 등을 위해 사용              | `/man`          | 온라인 메뉴얼                                                |





#### Linux 명령어

* `su` (switch user) : 현재 계정을 로그아웃 하지 않고 다른 계정으로 전환
  * `su - user` : 환경변수 반영해 user 계정으로 변경
  * `logout` 또는 `exit` : 이전 계정으로 돌아가기
* `sudo` (superuser do) : 현재 계정에서 root 권한으로 명령 실행
  * `sudo -i` : root계정으로 전환 및 /root 디렉토리 시작
  * `sudo -s` : root계정으로 전환 및 현재 디렉토리 유지
  * `/etc/sudoers` 파일에 지정된 사용자만 `sudo` 명령을 사용할 수 있다



`ls -al` 명령으로 출력되는 정보는 차례로 파일종류, 권한, 링크수, 소유자, 그룹, 파일크기, 수정시간, 파일이름 이다.

여기서 권한을 나타내는 `drwxr-xr-x` 부분은 총 4부분으로 나눌 수 있다.

1. `d` : 파일(-) / 디렉토리(d) 구분
2. `rwx` : 사용자(소유자) 권한
3. `r-x` : 그룹 권한
4. `r-x` : 다른 사용자 권한

여기서, rwx는 각각 `r` 읽기(**r**ead), `w` 쓰기(**w**rite), `x` 실행(e**x**ecute) 권한을 나타낸다.

```bash
# u:사용자(user), g:그룹(group), o:다른사용자(other), a:전부(all)
$ chmod g+w 파일
--- g+w는 그룹(g)에게 write 권한 추가
$ chmod o-r 파일
--- o-r은 다른사용자(o)에게 read 권한 제거

# 숫자를 이용한 권한 위임
# 4:read, 2:write, 1:execute. 숫자 합으로 권한 선택
$ chmod 000 파일
--- 사용자, 그룹, 다른사용자의 모든 권한 제거
$ chmod 777 파일
--- 사용자, 그룹, 다른사용자의 모든 권한 추가
$ chmod 700 파일
--- 사용자에게만 모든 권한 추가
```



파일의 소유권을 변경할 때는, `chown`명령어를 사용한다. 대개 `sudo`를 사용한다.

```bash
# 사용자 변경
$ sudo chown 사용자 파일

# 그룹 변경
$ sudo chgrp 그룹 파일

# 사용자와 그룹 동시 변경
$ sudo chown 사용자.그룹 파일

# 디렉토리 사용자 변경
$ sudo chown 사용자 디렉토리/

# 디렉토리와 하위 파일 모두 사용자 변경
$ sudo chown -R 사용자 디렉토리/
```



* 사용자 계정 관련 명령어
  * `sudo passwd` : 루트 비밀번호 변경
  * `passwd 사용자` : 사용자 비밀번호 변경
  * `cat /etc/passwd `: 사용자 리스트 확인



* 반복 예약 작업 : `crontab` (로그: `/var/log/cron`)

```bash
# crontab 리스트 확인
$ crontab -l
# 사용자의 crontab 리스트 확인 
$ crontab -l -u 사용자

# crontab 등록
$ crontab -e
# 등록형식 : * * * * * 수행명령어
# (분:0-59) (시:0-23) (일:1-31) (월:1-12) (요일:0-6)
* * * * * /root/every_1min.sh
--- 매 1분마다 반복수행
0 2 * * * /root/every_2clock.sh
--- 매일 02시에 반복수행
30 */6 * * * /root/every_6hours.sh
--- 매 6시간마다 반복수행 (00:30 시작)
0 8 * * 1-5 /root/weekday.sh
--- 평일(월-금) 08:00시에 반복수행
0 8 * * 0,6 /root/weekend.sh
--- 주말(일,토) 08:00시에 반복수행
0 6 1 1,4,7,10 * /root/quarter.sh
--- 매분기 익월 1일 06시에 반복수행

# crontab 백업(txt로 저장)
$ crontab -l > /home/backup/crontab_bak.txt
# 자동화
$ crontab -e
50 23 * * * crontab -l > /home/backup/crontab_bak.txt
```



* 기타

```bash
# 파일 및 디렉토리 복사 : cp [source] [target]
$ cp /root/monday /root/monday_save
# omitting directory 에러시
$ cp -r /root/monday /root/monday_save

# 파일 및 디렉토리 이동 또는 이름 변경 : mv [source] [target]
$ mv /root/sunday /home/	# /root/sunday를 /home/ 디렉토리로 이동
$ mv /root/sunday /root/$(date+"%Y-%m-%d")		# /root/sunday를 오늘날짜(%Y-%m-%d)로 이름 변경
# $ date 입력시 설정된 타임존의 오늘 시간 출력. $(date +"%Y%m%d")로 현재 날짜 사용 가능.

# 빈파일 생성
$ touch 파일명
```













ref) https://withcoding.com/106

ref) https://blog.naver.com/sumni10/222145175979

ref) https://zetawiki.com/wiki/%EB%A6%AC%EB%88%85%EC%8A%A4_%EB%B0%98%EB%B3%B5_%EC%98%88%EC%95%BD%EC%9E%91%EC%97%85_cron,_crond,_crontab

















































































