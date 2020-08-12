---
title: Raspberry Pi OS 이미지를 직접 빌드하기
date: 2020-08-12 00:34:59
tags:
    - Raspberry Pi
    - Linux
    - Debian
toc: true
---

얼마 전 Raspberry Pi OS의 arm64 이미지가 공개되었는데요, 데스크탑 버전만 공개되어서 라이트 버전은 구할 수 없나 알아보던 도중 pi-gen을 이용하면 이미지를 원하는 대로 빌드할 수 있다고 해서 한번 사용해봤습니다.

<!-- more -->

## 시작하기 전에
이 글은 https://github.com/RPi-Distro/pi-gen/blob/master/README.md 에서 많은 부분을 참고했습니다.

## 소스 다운로드
[pi-gen](https://github.com/RPi-Distro/pi-gen.git)을 클론합니다. 64비트 이미지를 빌드할 경우 arm64 브랜치를 클론하면 됩니다.

```bash
git clone https://github.com/RPi-Distro/pi-gen.git # 32비트 이미지를 빌드할 경우
git clone -b arm64 https://github.com/RPi-Distro/pi-gen.git # 64비트 이미지를 빌드할 경우
cd pi-gen
```

## 이미지 커스터마이징
빌드할 이미지를 커스터마이징 하기 위해서는 아래의 변수를 수동으로 환경 변수에 등록하거나 `pi-gen` 디렉토리 안에 `config` 라는 파일을 새로 만들어 작성하시면 됩니다.

- `IMG_NAME`
빌드할 이미지의 이름입니다. 이름에 공백이 포함된 경우 빌드가 제대로 되지 않았습니다.

- `RELEASE`
Debian의 어떤 릴리즈를 사용할지 정하는 값이며, 기본값은 `buster` 입니다. `jessie` 이후의 릴리즈만 사용 가능하며 자세한 정보는 [이곳](https://ko.wikipedia.org/wiki/%EB%8D%B0%EB%B9%84%EC%95%88#%EB%B0%B0%ED%8F%AC_%EC%97%AD%EC%82%AC)을 참고하세요.

- `APT_PROXY`
apt 사용시 어떤 프록시를 사용할지 정하는 값이며, 빌드할 때만 사용되고 이미지에 기록되지 않습니다.

- `BASE_DIR`
pi-gen의 최상위 디렉토리를 지정하는 값으로, 빌드 스크립트가 자동으로 이 디렉토리를 찾으므로 굳이 지정할 필요가 없습니다. 잘못된 디렉토리를 지정할 경우 정상적으로 빌드되지 않습니다.

- `WORK_DIR`
빌드하면서 생성되는 임시 파일을 저장할 디렉토리입니다. 기본값은 `$BASE_DIR/work`이며, 용량이 부족하거나 더 빠른 저장장치를 사용해서 빌드 시간을 단축시키고 싶으실 때 디렉토리를 지정하는 것을 추천합니다. NTFS로 포맷된 드라이브에서는 정상적으로 작동되지 않습니다.

- `DEPLOY_DIR`
빌드된 이미지를 저장할 디렉토리입니다. 기본값은 `$BASE_DIR/deploy`입니다.

- `DEPLOY_ZIP`
이미지를 ZIP으로 생성할지 결정하는 값입니다. 0일 경우 .img 파일이, 1일 경우 .zip 파일이 생성됩니다.

- `USE_QEMU`
빌드할 이미지를 QEMU에 최적화할지 결정하는 값입니다. 1일 경우 QEMU에서 사용되는 것을 가정하고 이미지를 빌드합니다.

- `LOCALE_DEFAULT`
시스템 로케일을 지정하며, 기본값은 `en_GB.UTF-8`입니다. `ko-KR.UTF-8`로도 지정이 가능하나 글자라 깨져서 표시될 수 있으니 기본값이나 `en_US.UTF-8`을 사용하는 것을 추천합니다.

- `TARGET_HOSTNAME`
호스트 네임을 지정하며, 기본값은 `raspberrypi` 입니다.

- `KEYBOARD_KEYMAP`
키보드 배열을 지정하며, 기본값은 `gb` 입니다. `gb` 는 영국식 키보드 배열인데, 우리가 흔히 사용하는 배열이랑 꽤 많이 달라 `us` 로 지정하는 것을 추천합니다.

![UK Keyboard layout (출처: Wikipedia)](/images/keymap_gb.svg)

- `TIMEZONE_DEFAULT`
타임존을 지정하며, `Europe/London` 이 기본값입니다. `Asia/Seoul` 로 지정하는 것을 추천합니다.

- `FIRST_USER_NAME`
처음 로그인 할 계정의 이름이며, 기본값은 `pi` 입니다. 영문 소문자, 숫자, 언더스코어(_)로만 이루어져야 하며 숫자가 맨 앞에 나오면 안되고 최대 31글자까지 입력할 수 있습니다.

- `FIRST_USER_PASS`
처음 로그인 할 계정의 패스워드이며, 기본값은 `raspberry` 입니다.

- `WPA_ESSID`
사용할 와이파이의 SSID입니다.

- `WPA_PASSWORD`
사용할 와이파이의 패스워드입니다.

- `WPA_COUNTRY`
사용할 와이파이가 있는 국가를 지정하며, 한국으로 설정할 경우 와이파이가 작동하지 않는 문제가 있는 것으로 알고 있습니다.

- `ENABLE_SSH`
SSH를 사용할지 정하며, 기본값은 0(비활성화)입니다. 1(활성화)로 설정하면 부팅 즉시 SSH 접속이 가능하지만 보안에 취약하며 이 설정은 raspi-config로 변경 가능합니다.

- `STAGE_LIST`
이미지를 만들 때 어떤 단계를 거칠지 정하는 값입니다. `stage0 stage1 stage2` 와 같은 형태로 작성하며, 앞에서부터 순서대로 실행됩니다.

    - `stage0`
    기본적인 파일 시스템을 만들고 부트에 필요한 것들을 설치합니다. 하지만 이 단계만 설치해서는 제대로 부팅이 되지 않습니다.

    - `stage1`
    시스템에 필요한 최소한의 것들을 설치합니다. 부트로더와 네트워크를 사용할 수 있도록 설정하고, raspi-config와 같은 필수적인 패키지를 설치하며 실사용이 가능하지만 빠진 기능이 많아 직접 설치하고 설정해야 하는 것들이 많습니다.

    - `stage2`
    라이트 버전의 이미지를 생성합니다. 와이파이와 블루투스와 같은 기본적인 기능이 추가되며 파이썬, Lua, build-essential과 같은 패키지가 포함되는데 이런 패키지가 포함되지 않은 이미지를 만들고 싶다면 pi-gen을 직접 수정하셔야 합니다.

    - `stage3`
    데스크탑을 위한 프로그램을 설치합니다. X11, LXDE, 웹 브라우저, 개발 도구와 같은 프로그램들이 설치됩니다.

    - `stage4`
    일반적인 라즈베리파이 이미지입니다. 4GB짜리 SD카드에 맞게 설계되었으며, 사용자가 라즈베리파이를 친숙하게 사용할 수 있도록 하는 프로그램들이 설치됩니다.

    - `stage5`
    풀버전의 라즈베리파이 이미지입니다. 더 많은 개발 도구, 이메일 클라이언트, 스크래치, sonic-pi, 리브레 오피스 등을 설치합니다.

## 빌드
### 데비안 기반 OS 사용자
Debian Buster과 Ubuntu Xenial에서 정상적으로 빌드되며 구버전의 OS에서는 제대로 빌드되지 않습니다. 그 외의 운영체제의 경우에는 도커를 이용해서 빌드하세요.

```bash
apt-get install coreutils quilt parted qemu-user-static debootstrap zerofree zip dosfstools bsdtar libcap2-bin grep rsync xz-utils file git curl bc
./build.sh
```

### 도커 사용자
```bash
./build-docker.sh
```