*   먼저 [용어집](./terminology.md)을 읽어보시길 강력히 권해드립니다.

# 📚 목차
*   💻 [Termux (Proot & Chroot)에서 하드웨어 가속](#hardware-acceleration-prootandchroot)
*   💻 [Termux 네이티브에서 하드웨어 가속](#hardware-acceleration-termux-native)
*   🪲 [문제 해결 및 수정](#troubleshoot)

<br>
<br>  

---  
<br>

# Termux (Proot & Chroot)에서 하드웨어 가속 <a name=hardware-acceleration-prootandchroot></a>
> [!NOTE]  
> 하드웨어 가속 분야는 워낙 방대해서, 제가 아직 연구 중인 모든 정보들을 이곳에 담아두려고 합니다. 혹시 내용 중에 오류나 오해의 소지가 있는 부분을 발견하시면, YouTube, Telegram 또는 이 GitHub에서 이슈를 열어 댓글로 알려주시면 정말 감사하겠습니다.

🔥[[영상] 하드웨어 가속 파트 1 - 무엇이며 어떻게 사용하나요 (VIRGL 및 ZINK)](https://www.youtube.com/watch?v=fgGOizUDQpY)  
🔥[[영상] 하드웨어 가속 파트 2 - (VIRGL, ZINK, TURNIP) - 어떻게 사용할 수 있나요](https://www.youtube.com/watch?v=07kq4RHbXrE)  
🔥[[영상] 하드웨어 가속 파트 3 - 하드웨어 가속을 통해 완전한 데스크톱을 실행하는 방법](https://youtu.be/OiLXkvFoUJQ?feature=shared)

## 1. 패키지 설치
Termux에서 다음 패키지들을 설치해 주세요:
```
pkg install -y tur-repo
pkg install mesa-zink virglrenderer-mesa-zink vulkan-loader-android virglrenderer-android
```

## 2. Termux에서 그래픽 서버 초기화:
proot에 로그인하여 하드웨어 가속을 사용하시려면, 먼저 그래픽 서버를 시작해 주셔야 합니다:

*   Vulkan (ZINK):
    ```
    MESA_NO_ERROR=1 MESA_GL_VERSION_OVERRIDE=4.3COMPAT MESA_GLES_VERSION_OVERRIDE=3.2 GALLIUM_DRIVER=zink ZINK_DESCRIPTORS=lazy virgl_test_server --use-egl-surfaceless --use-gles &
    ```
*   OpenGL (VIRGL):
    ```
    virgl_test_server_android &
    ```
*   [Turnip (Adreno GPU 6XX/7XX 호환만 해당)](https://www.reddit.com/r/termux/comments/19dpqas/proot_linux_only_dri3_patch_mesa_turnip_driver/)
    별도로 그래픽 서버를 초기화할 필요가 없습니다. Reddit 게시물에 설명된 단계를 따라주세요. 간략하게 요약해 드리자면:

    1.  Turnip 드라이버 다운로드: [mesa-vulkan-kgsl_24.1.0-devel-20240120_arm64.deb](https://drive.google.com/file/d/1f4pLvjDFcBPhViXGIFoRE3Xc8HWoiG-/view?usp=drive_link)
    2.  proot-distro에 설치해 주세요 (예: Debian에서는 다음 명령을 사용합니다)
        ```
        sudo dpkg -i mesa-vulkan-kgsl_24.1.0-devel-20240120_arm64.deb
        ```
        드라이버를 제거하고 싶으시다면:
        ```
        sudo dpkg -r mesa-vulkan-drivers:arm64
        ```

## 3. proot 배포판에서
제 스크립트로 데스크톱을 실행해 주세요 (만약 제 스크립트 대신 수동으로 진행하신다면, 원활한 작동을 위해 `tmp` 디렉터리를 공유해야 한다는 점을 꼭 기억해 주세요):
```
./startxfce4_debian.sh
```

데스크톱에 진입한 후, 하드웨어 가속으로 프로그램을 실행하고 싶으실 때는 이 명령을 먼저 사용해 주세요:

*   VIRGL 및 ZINK의 경우 (Termux에서 시작한 그래픽 서버에 따라 ZINK 또는 VIRGL이 사용될 거예요):
    ```
    GALLIUM_DRIVER=virpipe MESA_GL_VERSION_OVERRIDE=4.0 program
    ```
*   TURNIP의 경우:
    ```
    MESA_LOADER_DRIVER_OVERRIDE=zink TU_DEBUG=noconform program
    ```

# 성능 결과

사용된 기기: Lenovo Legion Y700 2022 모델 (Snapdragon 870 - Adreno 650)

### GLMARK2
glmark2를 설치하시려면:
```
# Termux에서: pkg install glmark2
# proot-distro (Debian)에서: sudo apt install glmark2
```
> [!IMPORTANT]  
> 다음 테스트들은 proot distro 환경의 Debian 및 XFCE4 데스크톱에서 진행되었습니다.

<table>
  <thead>
    <tr>
      <th scope="col" colspan="6">DEBIAN PROOT (GLMARK2 점수 - 점수가 높을수록 성능이 좋습니다)</th>
    </tr>
    <tr>
        <th scope="col">실행</th>
        <th scope="col">LLVMPIPE</th>
        <th scope="col">VIRGL</th>
        <th scope="col">VIRGL ZINK</th>
        <th scope="col">TURNIP</th>
        <th scope="col">ZINK</th>
    </tr>
  </thead>

  <tbody>
    <tr>
      <td>1</td>
      <td>93</td>
      <td>70</td>
      <td>66</td>
      <td>198</td>
      <td>오류</td>
    </tr>
    <tr>
      <td>2</td>
      <td>93</td>
      <td>77</td>
      <td>66</td>
      <td>198</td>
      <td>오류</td>
    </tr>
    <tr>
      <td>3</td>
      <td>72</td>
      <td>70</td>
      <td>71</td>
      <td>198</td>
      <td>오류</td>
    </tr>
    <tr>
      <td>4</td>
      <td>94</td>
      <td>76</td>
      <td>66</td>
      <td>197</td>
      <td>오류</td>
    </tr>
    <tr>
      <td>5</td>
      <td>93</td>
      <td>75</td>
      <td>67</td>
      <td>198</td>
      <td>오류</td>
    </tr>
  </tbody>
  <tfoot>
    <tr>
      <th scope="row">서버 초기화</th>
      <td>필요 없음</td>
      <td><code>virgl_test_server_android &</td>
      <td><code>MESA_NO_ERROR=1 MESA_GL_VERSION_OVERRIDE=4.3COMPAT MESA_GLES_VERSION_OVERRIDE=3.2 GALLIUM_DRIVER=zink ZINK_DESCRIPTORS=lazy virgl_test_server --use-egl-surfaceless --use-gles &</code></td>
      <td>필요 없음</td>
      <td><code>MESA_NO_ERROR=1 MESA_GL_VERSION_OVERRIDE=4.3COMPAT MESA_GLES_VERSION_OVERRIDE=3.2 GALLIUM_DRIVER=zink ZINK_DESCRIPTORS=lazy virgl_test_server --use-egl-surfaceless --use-gles &</code></td>
    </tr>
    <tr>
      <th scope="row">사용된 명령어</th>
      <td><code>glmark2</td>
      <td><code>GALLIUM_DRIVER=virpipe MESA_GL_VERSION_OVERRIDE=4.0 glmark2</td>
      <td><code>GALLIUM_DRIVER=virpipe MESA_GL_VERSION_OVERRIDE=4.0 glmark2</td>
      <td><code>MESA_LOADER_DRIVER_OVERRIDE=zink TU_DEBUG=noconform glmark2</td>
      <td><code>GALLIUM_DRIVER=zink MESA_GL_VERSION_OVERRIDE=4.0 glmark2</td>
    </tr>
    <tr>
      <th scope="row">GLMARK GPU 정보</th>
      <td>llvmpipe</td>
      <td>virgl (Adreno)</td>
      <td>virgl (zink Adreno)</td>
      <td>virgl (Turnip Adreno)</td>
      <td>zink (Adreno)</td>
    </tr>
  </tfoot>
</table>

---

> [!IMPORTANT]  
> 다음 테스트들은 Termux (proot-distro가 아님) 및 XFCE4 데스크톱에서 진행되었습니다.

<table>
  <thead>
    <tr>
      <th scope="col" colspan="6">TERMUX (PROOT 아님) (GLMARK2 점수 - 점수가 높을수록 성능이 좋습니다)</th>
    </tr>
    <tr>
        <th scope="col">실행</th>
        <th scope="col">LLVMPIPE</th>
        <th scope="col">VIRGL</th>
        <th scope="col">VIRGL ZINK</th>
        <th scope="col">ZINK</th>
        <th scope="col">TURNIP</th>
    </tr>
  </thead>

  <tbody>
    <tr>
      <td>1</td>
      <td>69</td>
      <td>오류</td>
      <td>92</td>
      <td>121</td>
      <td>해당 없음</td>
    </tr>
    <tr>
      <td>2</td>
      <td>70</td>
      <td>오류</td>
      <td>92</td>
      <td>122</td>
      <td>해당 없음</td>
    </tr>
    <tr>
      <td>3</td>
      <td>69</td>
      <td>오류</td>
      <td>93</td>
      <td>121</td>
      <td>해당 없음</td>
    </tr>
    <tr>
      <td>4</td>
      <td>69</td>
      <td>오류</td>
      <td>93</td>
      <td>124</td>
      <td>해당 없음</td>
    </tr>
    <tr>
      <td>5</td>
      <td>69</td>
      <td>오류</td>
      <td>93</td>
      <td>123</td>
      <td>해당 없음</td>
    </tr>
  </tbody>
  <tfoot>
    <tr>
      <th scope="row">서버 초기화</th>
      <td>필요 없음</td>
      <td><code>virgl_test_server_android &</td>
      <td><code>MESA_NO_ERROR=1 MESA_GL_VERSION_OVERRIDE=4.3COMPAT MESA_GLES_VERSION_OVERRIDE=3.2 GALLIUM_DRIVER=zink ZINK_DESCRIPTORS=lazy virgl_test_server --use-egl-surfaceless --use-gles &</code></td>
      <td><code>MESA_NO_ERROR=1 MESA_GL_VERSION_OVERRIDE=4.3COMPAT MESA_GLES_VERSION_OVERRIDE=3.2 GALLIUM_DRIVER=zink ZINK_DESCRIPTORS=lazy virgl_test_server --use-egl-surfaceless --use-gles &</code></td>
      <td>해당 없음</td>
    </tr>
    <tr>
      <th scope="row">사용된 명령어</th>
      <td><code>glmark2</td>
      <td><code>GALLIUM_DRIVER=virpipe MESA_GL_VERSION_OVERRIDE=4.0 glmark2</td>
      <td><code>GALLIUM_DRIVER=virpipe MESA_GL_VERSION_OVERRIDE=4.0 glmark2</td>
      <td><code>GALLIUM_DRIVER=zink MESA_GL_VERSION_OVERRIDE=4.0 glmark2</td>
      <td>해당 없음</td>
    </tr>
    <tr>
      <th scope="row">GLMARK GPU 정보</th>
      <td>llvmpipe</td>
      <td>virgl (Adreno)</td>
      <td>virgl (zink Adreno)</td>
      <td>zink (Adreno)</td>
      <td>해당 없음</td>
    </tr>
  </tfoot>
</table>


---
### [Firefox Aquarium WebGL 벤치마크](https://webglsamples.org/aquarium/aquarium.html)

> [!NOTE]  
> GPU를 사용하시려면, Firefox에서 [WebGL을 활성화](https://tecnorobot.educa2.madrid.org/tecnologia/-/visor/configurar-webgl)해 주셔야 합니다.

<table>
  <thead>
    <tr>
      <th scope="col" colspan="4">DEBIAN PROOT (FIREFOX-ESR WEBGL 아쿠아리움 FPS - 점수가 높을수록 성능이 좋습니다)</th>
    </tr>
    <tr>
        <th scope="col">LLVMPIPE</th>
        <th scope="col">VIRGL</th>
        <th scope="col">VIRGL ZINK</th>
        <th scope="col">TURNIP</th>
    </tr>
  </thead>

  <tbody>
    <tr>
      <td>4</td>
      <td>20</td>
      <td>17</td>
      <td>웹페이지 충돌</td>
    </tr>
  </tbody>
</table>

<table>
  <thead>
    <tr>
      <th scope="col" colspan="5">TERMUX (PROOT 아님) (FIREFOX-ESR WEBGL 아쿠아리움 FPS - 점수가 높을수록 성능이 좋습니다)</th>
    </tr>
    <tr>
        <th scope="col">LLVMPIPE</th>
        <th scope="col">VIRGL</th>
        <th scope="col">VIRGL ZINK</th>
        <th scope="col">ZINK</th>
        <th scope="col">TURNIP</th>
    </tr>
  </thead>

  <tbody>
    <tr>
      <td>2</td>
      <td>오류</td>
      <td>24</td>
      <td>40</td>
      <td>해당 없음</td>
    </tr>
  </tbody>
</table>

![Firefox의 WEB GL 아쿠아리움](./images/webglaquarium.png)

---
제가 진행한 다른 테스트:

*   SuperTuxKart 30초 테스트 결과
![SUPERTUXKART 비교](./images/supertuxkart_comparison.png)


# Archlinux에서 하드웨어 가속 (Proot & Chroot) <a name=hardware-acceleration-arch-linux></a>
(Turnip은 Adreno 610 이상 GPU에서만 호환되며, 710, 642L 등 일부 예외가 있습니다)

## Turnip
*   1. 패키지 설치
    ```
    sudo pacman -S unzip mesa vulkan-icd-loader vulkan-headers
    ```
*   2. Turnip 드라이버 다운로드
    ```
    wget https://github.com/MatrixhKa/mesa-turnip/releases/download/24.1.0/mesa-turnip-kgsl-24.1.0-devel.zip
    ```
*   3. 설정
    ```
    unzip mesa-* -d turnip/
    cd turnip/
    sudo mkdir -p /usr/share/vulkan/icd.d/
    sudo mv libvulkan_freedreno.so /usr/lib/
    sudo mv freedreno_icd.aarch64.json /usr/share/vulkan/icd.d/
    ```
*   추가.  
    (시작 스크립트에 변수를 추가하실 수 있습니다)
    ```
    busybox chroot $mnt /bin/su - matrixz -c 'export DISPLAY=:0 && export PULSE_SERVER=127.0.0.1 && dbus-launch --exit-with-session && TU_DEBUG=noconform ZINK_DESCRIPTORS=lazy ZINK_DEBUG=compact openbox-session'
    ```

*   XFCE에서 추가 설정.
    컴포지터를 비활성화하셔야 합니다. 그렇지 않으면 검은 화면이 나올 수 있어요.
    `설정 -> 창 관리자 조정 -> 컴포지터 -> 컴포지터 활성화 체크 해제`

    이미 검은 화면이 나오셨다면, chroot 루트로 진입해 주세요.
    ```
    pacman -R vulkan-icd-loader
    ```

    Termux 데스크톱을 다시 시작하신 후, 컴포지터를 비활성화해 주세요. 그 다음 vulkan-icd-loader를 다시 설치하시면 됩니다.
    ```
    pacman -S vulkan-icd-loader
    ```
    
## 즐겁게 사용해 보세요😉.
(모든 과정이 잘 진행되었다면, 다음과 같은 화면을 보실 수 있을 거예요)

![Screenshot_20250305-195359_Termux_X11](https://github.com/user-attachments/assets/bacd609c-ef15-4588-9cd8-ebccae602db6)
![Screenshot_20250305-195524_Termux_X11](https://github.com/user-attachments/assets/30de9ef7-4f80-45c4-b659-17ec3621eeff)

# Termux 네이티브에서 하드웨어 가속 <a name=hardware-acceleration-termux-native></a>
(Freedreno 및 Turnip은 Adreno 610 이상 GPU에서만 호환되며, 710, 642L 등 일부 예외가 있습니다)

## Turnip
*   설치
    ```
    apt install mesa-vulkan-icd-freedreno-dri3
    ```
*   Turnip으로 프로그램을 실행하시려면:
    ```
    MESA_LOADER_DRIVER_OVERRIDE=zink PROGRAM
    ```

## Freedreno(실험적)
> [!WARNING]
> 이 기능은 아직 테스트 중이며, 설치 시 XFCE4 데스크톱 환경이 손상될 수 있습니다. 설치는 본인의 책임 하에 진행해 주세요!
    #### freedreno 다운로드 및 빌드
    ```
    apt update
    apt install python3 bison
    pip install PyYAML
    
    wget https://gitlab.freedesktop.org/Pipetto-crypto/mesa/-/archive/freedreno/mesa-freedreno.tar.gz
    
    meson build --prefix $PREFIX -Dlibdir=$PREFIX/lib -D platforms=x11 -Dgallium-drivers=freedreno -Dfreedreno-kmds=kgsl,msm -D vulkan-drivers= -D dri3=enabled -D egl=enabled -D gles2=disabled -D glvnd=disabled -D glx=dri -D libunwind=disabled -D shared-glapi=enabled -Dshared-llvm=disabled -D microsoft-clc=disabled -D valgrind=disabled -D gles1=disabled
    ```
    #### 기존 파일 제거
    이 파일들이 존재한다면 제거해 주세요 (이는 ZINK Turnip 설치를 손상시킬 수 있습니다!)
    ```
    rm /data/data/com.termux/files/usr/lib/libgbm.so
    rm /data/data/com.termux/files/usr/lib/libglapi.so
    ```
    
    #### 설치
    ```
    ninja -C build install
    ```
    
    #### freedreno kgsl로 프로그램을 실행하시려면:
    ```
    MESA_LOADER_DRIVER_OVERRIDE=kgsl PROGRAM
    ```

# 문제 해결 및 수정 <a name=troubleshoot></a>

### 게임에서 qwerty/WASD 키가 인식되지 않는 경우
이 문제를 해결하시려면:
```Termux:X11 -> 환경설정 -> "가능하면 스캔코드 선호" 활성화```

(다른 임시 해결책으로는 qwerty 키를 누르는 동안 CTRL 키를 누르는 방법이 있지만, 위의 해결책이 더 편리하며 영구적으로 문제를 해결해 줍니다.)
