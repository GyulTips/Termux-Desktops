# 안드로이드에서 리눅스 구동하기

Termux X11에서 데스크톱 환경을 실행하기 위한 스크립트들을 모아두었습니다. 원하시는 리눅스 배포판을 쉽게 설치해 보세요!

### 이 글은 [LinuxDroidmaster](https://github.com/LinuxDroidMaster)님의 [Termux-Desktops](https://github.com/LinuxDroidMaster/Termux-Desktops)를 수정 및 번역한 것임을 밝힙니다.
자세한 내용은 [Termux-Desktops](https://github.com/LinuxDroidMaster/Termux-Desktops) 및 [원본 채널](https://youtube.com/@linuxdroidmaster)에서 확인해주세요!

# 📚 목차
* 🏁 [첫 단계](#first-steps)
* ⚔️ [Termux 네이티브 VS Proot-distro VS Chroot](#choose-linux)
* 🐧 [Proot 배포판 설치 방법: Alpine, Ubuntu, Debian, Arch, Kali Nethunter, Parrot OS, PostMarket OS](#proot-distributions)
* 💀 [Chroot 배포판 설치 방법: Ubuntu, Debian, Box64Droid](#chroot-distributions)
* 💻 [Termux 네이티브 데스크톱 설치 방법](#termux-native)
* 🔥 [Termux에서 하드웨어 가속](https://github.com/GyulTips/Termux-Desktops/blob/main/Documentation/HardwareAcceleration.md)

<br>
<br>  

---  
<br>

## 리눅스 환경 미리 보기
XFCE4 데스크톱 기준입니다. 설치 시 다른 데스크톱 환경으로도 변경할 수 있어요!

| Proot 배포판 (Debian) | 네이티브 | Chroot (Debian) |
|---------------------------------------------|---------------------------------------------|---------------------------------------------|
| <img src="/Documentation/images/preview_proot.jpg"/> | <img src="/Documentation/images/preview_native.jpg"/>| <img src="/Documentation/images/preview_chroot.jpg"/>|

---  
<br>
<br>

# 🏁 첫 단계 <a name=first-steps></a>
안드로이드 기기에서 완전한 리눅스 데스크톱 환경을 즐기기 위해 Termux와 Termux X11을 사용해 볼 거예요.

* [[영상] Termux 설치 방법](https://www.youtube.com/watch?v=OMJAyq5NHp0)
* [[영상] Termux X11 설치 및 사용 방법](https://www.youtube.com/watch?v=mXkXzFqSeYE)

Termux에 설치해야 할 기본 패키지들입니다.

```
pkg update
pkg upgrade
pkg install x11-repo
pkg install termux-x11-nightly
pkg install tur-repo
pkg install pulseaudio
pkg install proot-distro
pkg install wget
pkg install git
```

---  
<br>

# ⚔️ Termux 네이티브 VS Proot-distro VS Chroot <a name=choose-linux></a>

안드로이드 기기에 리눅스를 설정하실 때, 여러 가지 옵션 중에서 선택하실 수 있습니다. 각 옵션의 차이점을 이해하시면 본인에게 가장 적합한 환경을 결정하는 데 도움이 되실 거예요.

### [1. Termux 네이티브](#termux-native)

- 메인 영상: [Termux 네이티브 데스크톱](https://www.youtube.com/watch?v=rq85dxMb7e4)

Termux 네이티브는 Termux 앱 내에서 리눅스 명령어를 추가적인 가상화나 컨테이너화 없이 직접 실행합니다. 리눅스 유틸리티를 가볍고 직관적인 방법으로 사용할 수 있으나, 호환성과 성능에 있어 부족할 수 있습니다.

### [2. Proot-Distro](#proot-distro)

- 메인 영상: [Debian proot 및 Termux X11 기본 설치](https://www.youtube.com/watch?v=mXkXzFqSeYE)

Proot-Distro는 가상 환경 내에서 완전한 리눅스 배포판을 실행하는 방식입니다. 루트 권한 없이도 다양한 리눅스 배포판을 설치하고 사용할 수 있게 해주지만, 네이티브 설치에 비해 몇 가지 제한 사항이 있을 수 있습니다.

### [3. Chroot](#chroot)

- 메인 영상: [Debian Chroot](https://www.youtube.com/watch?v=EDjKBme0DRI)

Chroot는 PRoot와 비슷하지만 루팅을 통해 시스템 단에 배포판을 설치한다는 차이가 있습니다. PRoot에 비해 더 쾌적한 환경을 제공하지만, 더 고급 설정과 추가적인 도구가 필요하며 시스템이 손상될 수 있습니다.

#### 요약

-   **Termux 네이티브:** 간단하고 가볍지만, 완전한 리눅스 배포판에 비해 기능이 제한적일 수 있습니다.
-   **Proot-Distro:** 루트 권한 없이 완전한 리눅스 배포판을 실행할 수 있게 해주지만, 몇 가지 제한 사항이 있을 수 있습니다.
-   **Chroot:** 가장 쾌적한 환경을 제공하지만, 더 복잡한 설정과 추가 도구가 필요합니다.

안드로이드 기기에서 사용할 리눅스 환경을 선택하실 때는 사용자님의 요구 사항과 선호도를 고려해 주세요.

---  
<br>

## 안드로이드 리눅스 환경 비교

| 특징             | Proot          | 네이티브         | Chroot         |
|---------------------|----------------|----------------|----------------|
| 루트 필요?         | ❌ (아니요)        | ❌ (아니요)        | ✅ (예)       |
| 다양한 리눅스 앱?    | ✅ (예)   | ❌ (제한적)       | ✅ (예)       |
| 성능         | 보통 💼    | 좋음 🚀        | 좋음 🚀   |

-   **루트 필요?:** 환경을 설정하는 데 루트 권한이 필요한지 여부를 나타냅니다.
-   **다양한 리눅스 앱?:** 다양한 리눅스 애플리케이션과의 호환성 수준을 보여줍니다.

---  
<br>

## 🐧 Proot 배포판 설치 방법: Ubuntu, Debian, Arch, Kali Nethunter, Parrot OS, PostMarket OS <a name=proot-distributions></a>

원하는 배포판을 설치하는 방법을 보시려면 각 아이콘을 클릭해 주세요. 모든 배포판에 대한 설치 과정을 설명하는 영상이 준비되어 있습니다.

| Alpine | Ubuntu | Debian | Arch | Kali NetHunter | Parrot OS | PostMarket | Void |
|--------|--------|------|----------------|----------------|----------------|----------------|----------------|
| <a href="/Documentation/proot/alpine_proot.md"><img src="https://upload.wikimedia.org/wikipedia/commons/thumb/6/60/New_Logo_Alpine_Linux.svg/1200px-New_Logo_Alpine_Linux.svg.png" alt="Alpine 로고" width="100"></a> | <a href="/Documentation/proot/ubuntu_proot.md"><img src="https://upload.wikimedia.org/wikipedia/commons/thumb/a/ab/Logo-ubuntu_cof-orange-hex.svg/1200px-Logo-ubuntu_cof-orange-hex.svg.png" alt="Ubuntu 로고" width="100"></a> | <a href="/Documentation/proot/debian_proot.md"><img src="https://www.shareicon.net/data/2015/09/16/101872_debian_512x512.png" alt="Debian 로고" width="100"></a> | <a href="/Documentation/proot/arch_proot.md"><img src="https://cdn0.iconfinder.com/data/icons/flat-round-system/512/archlinux-512.png" alt="Arch 로고" width="100"></a> | <a href="/Documentation/proot/kalinethunter_proot.md"><img src="https://static-00.iconduck.com/assets.00/distributor-logo-kali-linux-icon-2048x2005-dki611fk.png" alt="Kali 로고" width="100"></a> | <a href="/Documentation/proot/parrotos_proot.md"><img src="https://gdm-catalog-fmapi-prod.imgix.net/ProductLogo/b91dba39-aef6-4808-be11-8eda81f81f56.png" alt="Parrot OS 로고" width="100"></a> | <a href="/Documentation/proot/postmarket.md"><img src="https://upload.wikimedia.org/wikipedia/commons/thumb/a/a6/PostmarketOS_logo.svg/1024px-PostmarketOS_logo.svg.png" alt="PostMarket 로고" width="100"></a> | <a href="/Documentation/proot/voidlinux.md"><img src="https://upload.wikimedia.org/wikipedia/commons/thumb/0/02/Void_Linux_logo.svg/2485px-Void_Linux_logo.svg.png" alt="Void 로고" width="100"></a> |

---  
<br>

## 💀 Chroot 배포판 설치 방법: Ubuntu, Debian, Box64Droid <a name=chroot-distributions></a>

원하는 배포판을 설치하는 방법을 보시려면 각 아이콘을 클릭해 주세요. 모든 배포판에 대한 설치 과정을 설명하는 영상이 준비되어 있습니다.

| Ubuntu | Debian | Box64Droid (Ubuntu) | Arch | Fedora |
|--------|--------|--------|--------|--------|
| <a href="/Documentation/chroot/ubuntu_chroot.md"><img src="https://upload.wikimedia.org/wikipedia/commons/thumb/a/ab/Logo-ubuntu_cof-orange-hex.svg/1200px-Logo-ubuntu_cof-orange-hex.svg.png" alt="Ubuntu 로고" width="100"></a> | <a href="/Documentation/chroot/debian_chroot.md"><img src="https://www.shareicon.net/data/2015/09/16/101872_debian_512x512.png" alt="Debian 로고" width="100"></a> | <a href="/Documentation/chroot/box64droid_chroot.md"><img src="https://box64droid.com/wp-content/uploads/2023/10/Box64droid-logo.png" alt="Box64droid 로고" width="100"></a> | <a href="/Documentation/chroot/arch_chroot.md"><img src="https://cdn0.iconfinder.com/data/icons/flat-round-system/512/archlinux-512.png" alt="Arch 로고" width="100"></a> | <a href="/Documentation/chroot/fedora_chroot.md"><img src="https://upload.wikimedia.org/wikipedia/commons/4/41/Fedora_icon_%282021%29.svg" alt="Fedora 로고" width="100"></a> |

---  
<br>

## 💻 Termux 네이티브 데스크톱 설치 방법 <a name=termux-native></a>
네이티브 Termux 데스크톱을 설치하는 데 필요한 모든 정보와 사용 가능한 앱들은 [여기](/Documentation/native/termux_native.md)에서 확인하실 수 있습니다.
