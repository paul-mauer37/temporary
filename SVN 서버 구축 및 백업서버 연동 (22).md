# SVN 서버 구축 및 백업서버 연동 (2/2)





앞서, SVN 서버를 로컬 PC에 설치하는 방법을 살펴봤다. 여기까지만으로도 SVN을 버전 관리 툴로서 Git처럼 활용할 수 있다. 

이번 장에서는, SVN의 정형화된 사용법과 백업을 위한 작업에 대해 살펴보겠다.



### SVN 구성

svn 저장소를 생성하고 서버를 시작하는 부분까지 완료했으므로, 이제는 사용을 위해 **SVN저장소에 디렉토리를 구성**하고 사용하는 방법을 알아보겠다.



#### 1. trunk, branches, tags 디렉토리 생성

svn 저장소의 기본 디렉토리 구성은 `trunk`, `branches`, `tags` 이다. 

```bash
# trunk : 중심이 되는 디렉토리, 수정된 소스가 merge되는 저장소 디렉토리
$ svn mkdir --parents svn://localhost/저장소/trunk
# branches : trunk에서 수정이 필요한 부분을 뽑아낸 것, branches는 여럿이 존재할 수 있다
$ svn mkdir --parents svn://localhost/저장소/branches
# tags : 배포 버전 저장용 디렉토리 (필자의 경우 백업일자별 저장 디렉토리로 사용하고자 함)
$ svn mkdir --parents svn://localhost/저장소/tags

# 저장소 디렉토리 확인
$ svn list svn://localhost/저장소
branches/
tags/
trunk

$ svn info svn://localhost/저장소
경로:
URL: svn://localhost
저장소 루트: svn://localhost
저장소 UUID: 
리비전: 5
노드 종류: 디렉토리
마지막 수정 작업자:
마지막 수정 리비전: 5
마지막 수정 일자: 2021-10-12 09:15:27 +0900 (2021-10-12, 화)
```



#### 2. 저장소 내려받기

svn 서버는 현재 실행이 가능한 상태지만, 실물로 확인할 수는 없다. 가상의 원격 저장소에 저장된 상태로서 **실물로 확인**하고자 한다면 **로컬 PC에 svn 서버를 내려받아**야 한다.

```bash
# 작업 디렉토리에 저장소를 선택하고 내려받는다
$ svn checkout svn://localhost/저장소 작업디렉토리
--- svn checkout 과 svn co는 동일 기능
A	저장소/tags
A	저장소/trunk
A	저장소/branches
체크아웃된 리비전 3.

# 저장소의 특정 디렉토리를 내려받는 것도 가능하다
$ svn checkout svn://localhost/저장소/trunk 작업디렉토리
--- 작업디렉토리에 저장소의 trunk 디렉토리만 내려받기 된다.

# 저장소와 연결된 작업디렉토리에는 숨김 파일로 .svn디렉토리가 생성된다.
```

**Git**이 **로컬에 존재하는 디렉토리를** 원격 저장소와 연결해 사용한다면, **SVN**은 **원격 저장소에 디렉토리를 먼저 구성**하고 이를 로컬에 내려받아 사용한다는 점에서 차이가 있다. SVN 원격 저장소에 저장된 파일 목록을 보고자 한다면 client 프로그램이나 이클립스를 연동해 시각화해야한다. 



#### 3. SVN 사용하기

checkout 된 상태에서 svn 저장소 파일을 수정하고 디렉토리를 만들거나 삭제할 수 있다. 

```bash
# 3-1. 최신버전 내려받기
$ svn update
--- svn checkout이 된 상태에서 사용
Updating '.':
A	test.txt
...
업데이트 된 리비전 6.

$ svn updata -r 리비전번호
--- 특정 리비전으로 돌아가기

# 3-2. 업로드(Commit)
--- 파일을 수정하거나 새로운 파일을 생성 혹은 존재하던 파일을 삭제하는 등 변화가 발생할 경우,
$ svn status				# 저장소 변동사항 확인
$ svn add 변동파일				# 변동된 파일을 add 해준다.
$ svn commit -m '로그 메시지'		# add된 파일을 commit하며 로그 메시지를 남긴다.
--- commit이 이루어지면 새로운 리비전으로 기록되며 svn서버 저장소에 반영된다.

# 3-3. 저장소 내 파일 또는 디렉토리 삭제
$ svn delete svn://localhost/저장소/삭제디렉토리

# 3-4. 리비전 로그 확인
$ svn log
--- 현재 위치에서 svn log 명령 실행
--- svn: E155007: '/home/svn'은 작업 사본이 아닙니다 => svn 저장소 디렉토리를 지정해주면 된다.
$ svn log /home/svn/backup
--- svn 저장소 디렉토리 지정
$ svn log --revision 1:5		# 1~5까지 리비전 확인
$ svn log 파일				# 특정 파일에 관한 리비전 정보 확인
$ svn log -v					# 자세히
```



#### 4. SVN 사용하기2

```bash
4-1. 변경 내용 비교하기
$ svn diff -r1:4 파일
# 해당 파일의 리비전 1과 4를 비교해 변경된 내용 출력

4-2. 수정자 확인
$ svn blame 파일
# 해당 파일의 수정자와 수정내용을 확인한다.

4-3. 잠금 / 해제
$ svn lock/unlock 파일
# lock되면 다른 계정으로 svn명령을 내릴 수 없다

4-4. 이름 변경
$ svn rename 기존파일명 변경할파일명
# 이름 변경 후, commit 필요
```





---





Bac_com과 Dev_com의 SVN서버 설치가 모두 완료되었다. 

이제 마련된 두 저장소를 서로 연결해 **Dev_com의 svn저장소에서는 개발이 진행**되고, **Bac_com의 svn저장소에서는 개발 내용을 일자별로 백업**하게 만들 것이다. 여기서 활용할 내용은 3가지이다.

1. `svnsync`를 통해 **Dev_com과 Bac_com의 각 저장소를 연결**해준다
2. `svnadmin dump`명령을 통해 **백업**이 진행되고 `.dump` 파일로 저장해둬 필요시 `svnadmin load`로 열어서 확인할 수 있다.
3. `crontab`에 등록해 **정해진 시간과 주기별**로 백업이 진행될 수 있도록 **자동화**한다.

여기서 `svnsync`와 `svnadmin dump`를 사용하는 이유는 Dev_com의 svn저장소에서 이루어지는 **개발의 리비전 정보도 포함해 백업**하기 위함이다.





크론탭할때 대상이되는 svn저장소에 접근하기 위해선 비밀번호가 필요한데, 크론탭에선 비밀번호가 없어 실행은 되지만 실제 작동은 되지 않은 듯.







파일.sh : shell script의 약자로 shell이 실행하는 명령어 모음 파일

#!/bin/sh : 가볍다. 기능 적다. 대부분 사용

#!/bin/bash : 무겁다. 기능 많다. 주로 로그인에 사용







#### 5. SVN 백업

```bash
5-1. svn export
# svn 연동없이 순수 소스만 다운로드(버전관리 정보가 없어 백업용, 배포용 다운로드에 유용)
$ svn export svn://localhost/저장소 다운로드받을위치

5-2. svn copy
# svn 연동을 유지한채 디렉토리 복사(동일 저장소 내에서 주로 사용)
$ svn copy 복사할디렉토리 복사할위치

5-3. svnadmin dump
# 저장소의 물리적 디렉토리를 백업한다(.dump 형식으로 백업해 날짜별 백업 관리에 용이)
$ svnadmin dump 백업할디렉토리 > 백업파일명.dump

# dump된 백업본을 복구
$ svnadmin load 복구할위치 < dump파일.dump
```













ref) https://blog.naver.com/antimidal/221078722917

ref) https://blog.enleaf.me/311

ref) https://blog.naver.com/slimcdp/222455611252







