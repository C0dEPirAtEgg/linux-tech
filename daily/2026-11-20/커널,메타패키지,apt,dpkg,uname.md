2026-11-19 · #linux

> 한 줄 요약:

## 정리
- uname : (커널과 시스템 정보 확인) 지금 실행중인 커널의 정보를 보여주는 명령어.
- dpkg : (데비안/우분투의 패키지 관리도구) 패키지 파일을 실제로 설치하고 제거하는 가장 기본적인 도구.       .deb라는 압축 파일 묶음을 풀어주는데 사용 내 서버에 있는 .deb 파일을 풀어서 설치하고 기록
- apt : (우분투의 패키지 관리도구) 저장소에서 패키지찾아 받고 의존성을 계산해서 dpkg에게 설치를 시키는 관리도구 .deb을 다운로드 저장소 서명(CPG)로 자동 위조 검증
- apt,dpkg의 관계 : apt install nginx라는 명령어를 입력한다고 했을때 저장소에서 nginx와 필요한 패키지(의존성)을 찾음, .deb 파일들을 /var/cache/apt/archives/로 다운, dpkg를 호출해서 실제로 설치.
- dpkg -l 하였을때는 설치되지 않은 패키지는 보이지 않음 dpkg -l 'docker*' 처럼 필터링을 걸면 설치되지 않은 패키지도 보임 -l을 사용했을때 name 앞에 보인는 ii는 설치됨, un은 설치안됨, rc 제거됐지만 설치 파일 남음
- dpkg 설치는 두단계이다. 파일풀기, 설정 -> ii(완료) 풀기만 되면 iU, 설정하다 실패하면 iF
- 우분투 커널 패치 흐름 ① 패치 대상 파악 → ② 계획·승인 → ③ 백업 → ④ 테스트 서버 적용 → ⑤ 운영 서버 설치 → ⑥ 서비스 분리 후 재부팅 → ⑦ 검증 → ⑧ 정리 (문제 시 롤백) 패치하는 이유는 커널취약점, 정기 패치, 보안점검 등
- 커널은 문제가 생기면 이전 버전으로 부팅해야 한다. 그래서 여러 버전을 동시에 설치해둬야함. 
- 메타패키지란 프로그램 파일은(거의)없고, 이것들을 같이 설치해라라는 의존성 목록만 가진 패키지. 여러 패키지를 하나의 이름으로 묶어서 한번에 설치하고 관리하려고 쓴다. 식당의 세트메뉴와 같다.
- 메타패키지 linux-image-aws 의 동작 방법 실제 커널 파일이 들어 있지 않은 빈 껍데기 패키지 하는일은 딱 하나 최신 AWS커널 패키지가 필요하다는 의존성을 가지고 있다 linux-image-aws의 새버전 (1018)이 저장소에 올라오면 linux-image-aws는 linux-image-7.0.0-1018-aws가 필요하다는 의존성을 가진다. 메타패키지를 업그레이드하면 apt가 의존성을 따라 새커널 패키지를 설치함 기존 1017은 지우지 않고 그대로 남음
- linux-aws -> linux-image-aws(커널) + linux-headers-aws(커널 헤더)등을 묶음 linux-aws하나만 업그레이드하면 아래 전부 따라 올라온다.
- 메타패키지를 쓰는 이유 1.한번에 설치, 2.버전이 바뀌어도 이름은 그대로, 3.계속 최신을 따라감(중요)
- 인터넷 저장소 -> apt가 다운로드 -> .deb 파일 -> dpkg가 설치 -> 서버
- apt download는 이파일 하나 가져와 서버에 무엇이 설치되어있는지 상관없어 apt-get install --download-only 의존성을 계산하고 이서버에 없는것만 받는다
## 명령어
```bash
uname -s, uname -a, uname -o, uname -m, uname -n, uname -r #uname 명령어의 옵션

dpkg -l, dpkg -s, dpkg -L, dpkg -r, dpkg -P, dpkg -i #dpkg 명령어의 옵션

apt update, apt remove, apt search, apt install #apt 명령어
```

## 확인
- `uname -s`  커널의 이름을 확인 Linux 출력(기본값)
- `uname -m`  커널의 하드웨어 아키텍처 x86_64, aarch64 출력
- `uname -o` 운영체제 GNU/Linux 출력
- `cat /etc/os-release` 운영체제 배포판 이름 과 버전
- `ls /boot/vmlinuz-*` 부팅 가능한 커널 파일(설치된 커널)
- `cat /var/run/reboot-required.pkgs` 재부팅이 필요한 원인 패키지
- `dpkg -l` 설치된 패키지 목록
- `dpkg -l 'docker*'` docker로 시작하는 패키지 목록 보기
- `dpkg -s 'docker-ce` docker-ce 패키지 상태와 상세정보(버전, 의존성)
- `dpkg -i docker`
- `dpkg -r` 제거(설정 파일은 남김)
- `dpkg -p` 완전 제거(설정파일, purge)
- `apt update` 저장소에서 최신 패키지 목록만 받아온다.
- `apt full-upgrade -s` 서버의 모든 패키지를 한번에 업그레이드. -s로 미리보기
- `apt only-upgrade linux-aws` linux-aws 커널만 업그레이드
- `apt-cache depends linux-aws` linux-aws가 무엇을 묶고 의존을 가지는지
- `apt install` 설치후 기본적으로 .deb 삭제
- `apt install ./nginx.deb` 부족한 의존성을 저장소에서 받아서 함께 설치
- `apt install --reinstall` 재설치
- `apt remove` 제거 (설정 파일 남김)
- `apt purge` 설정 파일까지 완전 제거
- `apt search` 저장소에서 검색
- `apt-get install` 설치후 기본적으로 .deb 남겨둠
- `apt download` 저장소에서 .deb 파일만 받아서 현재 폴더에 저장하는 명령어
- `apt-cache depends nginx` nginx 패키지의 의존성을 보여주는 명령어
- `sudo apt-get install --download-only nginx` nginx 패키지의 의존성을 받음(서버에 없는것만)
- `/etc/apt/sources.list.d/ubuntu.sources` apt가 사용하는 저장소 설정 위치
- `/var/lib/dpkg/status` dpkg 기록
- `/var/cache/apt/archives/` apt는 .deb을 여기에 받아서 설치한다. 위와같이 apt-get을 사용하면 .deb 을 남기고 그냥 apt를 사용해서 설치하면 .deb을 삭제함
- `/var/log/apt/history.log` 누가 언제 어떤 명령으로 무엇을 설치,제거했는지
- `apt list --upgradable` 업데이트가 가능한 패키지 목록
- `apt upgrade -s` 업그레이드 시뮬레이션(미리보기)

## 문제 해결
- 문제: dpkg was interrupted, you must manually run 'sudo dpkg --configure -a' to correct the problem. 설치도중 ssh가 끊기거나 강제 종료 되었을때 뜨는 에러
- 해결: sudo dpkg --configure -a(설정 단계가 덜 끝난 패키지를 마무리), sudo apt -f install(부족한 의존성을 저장소에서 받아 채우거나 해결이 안되면 문제가 되는 패키지 제거)

