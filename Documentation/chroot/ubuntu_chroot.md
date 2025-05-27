이 글은 [LinuxDroidmaster](https://github.com/LinuxDroidMaster)님의 [Termux-Desktops](https://github.com/LinuxDroidMaster/Termux-Desktops)를 수정 및 번역한 것임을 밝힙니다.
자세한 내용은 [Termux-Desktops](https://github.com/LinuxDroidMaster/Termux-Desktops) 및 [원본 영상](https://www.youtube.com/watch?v=rYJaG0uFtdc)에서 확인해주세요!

> [주의!]
> Chroot 환경을 사용하실 때는 완전히 종료하려면 Termux 애플리케이션을 백그라운드 앱에서도 완전히 닫거나, 필요하다면 강제 종료해야 합니다. 그렇지 않으면, 'rm -rf chrootFolder'와 같은 명령을 실행했을 때 기기가 비정상적으로 작동하여 강제로 재부팅해야 할 수 있습니다.

# 📚 목차

## CHROOT (🟠 UBUNTU)
* 🏁 [시작하기](#first-steps-chroot)
* 💻🟠 [Ubuntu chroot 설정하기](#ubuntu-chroot)

<br>

---  

<br>

> [!NOTE]  
> 전체 과정이 담긴 영상은 아래에서 확인하실 수 있습니다.
> ## [뉴윈연구소 영상](https://www.youtube.com/) | [원본 영상](https://www.youtube.com/watch?v=rYJaG0uFtdc)

## 🏁 시작하기 <a name=first-steps-chroot></a>

1.  먼저 기기가 <u>루팅</u>되어 있어야 합니다.
2.  Magisk를 사용하여 [Busybox](https://github.com/Magisk-Modules-Alt-Repo/BuiltIn-BusyBox/releases)를 플래싱해 주세요.
3.  그런 다음 Termux를 열고 다음 패키지들을 설치해 주세요.

```
pkg update
pkg install x11-repo
pkg install root-repo
pkg install termux-x11-nightly
pkg update
pkg install tsu
pkg install pulseaudio
```

---  
<br>

## 💻🟠 Ubuntu chroot 설정하기 <a name=ubuntu-chroot></a>

이 단계들은 Ivon님의 블로그에서 가져온 것이지만, 일부 라인을 조금 수정했습니다. 참고한 게시물은 다음과 같아요.
*   [Install Ubuntu in chroot on Android without Linux Deploy](https://ivonblog.com/en-us/posts/termux-chroot-ubuntu/)
*   [How to use Termux X11 - The X server on Android phone](https://ivonblog.com/en-us/posts/termux-x11/)

1.  Termux에서 루트 권한으로 Android 셸에 진입하세요.
    ```
    su
    ```

2.  `/data/local/tmp`에 chroot 환경을 위한 디렉터리를 생성해 주세요.
    ```
    mkdir /data/local/tmp/chrootubuntu
    cd /data/local/tmp/chrootubuntu
    ```

3.  [Ubuntu 22.0.4 LTS](https://cdimage.ubuntu.com/ubuntu-base/releases/22.04/release/) rootfs를 다운로드해 주세요.
    ```
    curl https://cdimage.ubuntu.com/ubuntu-base/releases/22.04/release/ubuntu-base-22.04-base-arm64.tar.gz --output ubuntu.tar.gz
    ```

4.  다운로드한 파일의 압축을 해제하고 sdcard를 마운트할 폴더들을 생성해 주세요.
    ```
    tar xpvf ubuntu.tar.gz --numeric-owner

    mkdir sdcard
    mkdir dev/shm
    ```

5.  시작 스크립트를 생성해 주세요.
    ```
    cd ../
    vi start.sh
    ```
    다음 내용을 복사하여 붙여넣어주세요.
    ```
    #!/bin/sh

    # The path of Ubuntu rootfs
    UBUNTUPATH="/data/local/tmp/chrootubuntu"

    # Fix setuid issue
    busybox mount -o remount,dev,suid /data

    busybox mount --bind /dev $UBUNTUPATH/dev
    busybox mount --bind /sys $UBUNTUPATH/sys
    busybox mount --bind /proc $UBUNTUPATH/proc
    busybox mount -t devpts devpts $UBUNTUPATH/dev/pts

    # /dev/shm for Electron apps
    busybox mount -t tmpfs -o size=256M tmpfs $UBUNTUPATH/dev/shm

    # Mount sdcard
    busybox mount --bind /sdcard $UBUNTUPATH/sdcard

    # chroot into Ubuntu
    busybox chroot $UBUNTUPATH /bin/su - root
    ```

6.  스크립트를 실행 가능하게 만들고 실행해 주세요.
    ```
    chmod +x start.sh
    sh start.sh
    ```

7.  프롬프트가 `root@localhost`로 변경됩니다. (Termux로 돌아가려면 `exit`을 입력하세요.)
    ```
    echo "nameserver 8.8.8.8" > /etc/resolv.conf
    echo "127.0.0.1 localhost" > /etc/hosts

    groupadd -g 3003 aid_inet
    groupadd -g 3004 aid_net_raw
    groupadd -g 1003 aid_graphics
    usermod -g 3003 -G 3003,3004 -a _apt
    usermod -G 3003 -a root

    apt update
    apt upgrade

    apt install nano vim net-tools sudo git
    ```

8.  시간대를 설정해 주세요.
    ```
    ln -sf /usr/share/zoneinfo/Asia/Seoul /etc/localtime
    ```

9.  새 사용자 (여기서는 `gyul`) 를 생성해 주세요.
    > 원하는 이름으로 바꿀 수 있어요!
    ```
    groupadd storage
    groupadd wheel
    useradd -m -g users -G wheel,audio,video,storage,aid_inet -s /bin/bash gyul
    passwd gyul
    ```

11. sudoers 파일에 사용자를 추가해 주세요.
    ```
    nano /etc/sudoers
    ```
    (이 줄을 추가해 주세요.)
    ```
    gyul ALL=(ALL:ALL) ALL
    ```

12. 생성된 사용자로 전환해 주세요.
    ```
    sudo apt install locales
    sudo locale-gen en_US.UTF-8
    ```

13. 데스크톱 환경을 설치해 주세요.
    *   XFCE4
        ```
        sudo apt install xubuntu-desktop
        ```
    *   KDE Plasma
        ```
        sudo apt install kubuntu-desktop
        ```

14. Snapd 비활성화 (Termux에서는 사용할 수 없습니다.)
    ```
    apt-get autopurge snapd

    cat <<EOF | sudo tee /etc/apt/preferences.d/nosnap.pref
    # To prevent repository packages from triggering the installation of Snap,
    # this file forbids snapd from being installed by APT.
    # For more information: https://linuxmint-user-guide.readthedocs.io/en/latest/snap.html
    Package: snapd
    Pin: release a=*
    Pin-Priority: -10
    EOF
    ```
15. chroot 환경을 종료(`exit` 입력 또는 앱 강제 종료) 하고 Termux에 다음 명령들을 복사해 주세요.
    
    ```
    XDG_RUNTIME_DIR=${TMPDIR} termux-x11 :0 -ac &
    sudo busybox mount --bind $PREFIX/tmp /data/local/tmp/chrootubuntu/tmp
    ```

    chroot로 접속한 뒤
    ```
    su
    ```
    ```
    sh /data/local/tmp/start.sh
    ```
    다음 명령을 실행해 주세요.
    ```
    sudo chmod -R 777 /tmp
    ```

16. chroot 환경을 종료(`exit` 입력 또는 앱 강제 종료) 하고 `start.sh` 스크립트를 수정해 주세요.
    ```
    sudo nano /data/local/tmp/start.sh
    ```
    `busybox chroot $UBUNTUPATH /bin/su - root` 줄을 주석(`# `) 처리하고 이 줄로 변경해 주세요.
    ```
    busybox chroot $UBUNTUPATH /bin/su - gyul -c "export DISPLAY=:0 PULSE_SERVER=tcp:127.0.0.1:4713 && dbus-launch --exit-with-session startxfce4"
    ```
    > `gyul`부분에는 지정했던 사용자 이름을 넣어 주세요!
    
    > 다른 데스크톱 환경을 설치했다면 `startxfce4` 부분을 변경해야 합니다. 예를 들어 KDE Plasma의 경우 `startplasma-x11`이 됩니다.


 17. 편의를 위해 스크립트를 만들어 주세요.
     ```
     nano ./config
     ```
     (입력 후 저장)
     ```
     # Pulseaudio 서버 시작
     pulseaudio --start --exit-idle-time=-1
     pacmd load-module module-native-protocol-tcp auth-ip-acl=127.0.0.1 auth-anonymous=1

     # GPU 가속 설정
     # MESA_NO_ERROR=1 MESA_GL_VERSION_OVERRIDE=4.3COMPAT MESA_GLES_VERSION_OVERRIDE=3.2 GALLIUM_DRIVER=zink ZINK_DESCRIPTORS=lazy virgl_test_server --use-egl-surfaceless --use-gles &
     # virgl_test_server_android &

     # chroot 구성
     XDG_RUNTIME_DIR=${TMPDIR} termux-x11 :0 -ac &
     sudo busybox mount --bind $PREFIX/tmp /data/local/tmp/chrootubuntu/tmp

     # sudo 진입
     su
     ```
     (저장이 끝나면 명령 실행)
     ```
     chmod +x ./config
     ./config
     ```
     (./config 실행이 끝난 su 환경에서)
     ```
     cp /data/local/tmp/start.sh ./start
     chmod +x ./start
     ```

 18. 설치 완료! (편의를 위해) 앱 강제 종료 - 다시 실행 후 다음 명령으로 실행해 보세요.
     ```
     ./config
     ```
     ```
     ./start
     ```
     실행이 끝나면 termux-x11 앱을 열어 주세요!

  ## 💻⚙️ GPU 가속 설정하기
  GPU 가속 설정 부분은 아직 준비 중에 있습니다. 추후에 다시 설명드릴게요!

</details>
