# 애플 실리콘 기반 Mac에서 buildah를 구동하는 방법

빌다의 인스톨 가이드를 보면 확인할 수 있는 사실이지만, [빌다는 애플 실리콘 상에서의 설치를 지원하지 않습니다](https://github.com/containers/buildah/blob/main/install.md). 
사실 애플 플랫폼의 패키지 매니저 가운데 하나인 brew를 아예 지원하지 않는 것 처럼 보이죠. 

그래서 애플 실리콘과 맥 OS에서 빌다를 시험해 보려면 x86 아키텍처를 에뮬레이션 하고 그 위에 운영체제를 설치할 필요가 있습니다. 

x86 아키텍처를 emulation하는 좋은 방법은 다음 명령어로 qemu를 설치하는 것입니다.

```
brew install qemu
```

대체로 이 설치 과정은 별 문제 없이 끝납니다. 그렇다면 이 위에 CentOS 7를 설치해 봅니다. 빌다는 이 오래된 운영체제 위에서도 잘 동작합니다. 설치를 하려면 다음과 같이 합니다.

## Centos 7 다운로드

우선 [https://www.centos.org/download/](https://www.centos.org/download/)https://www.centos.org/download/ 에 방문하여 x86_64 버전의 CentOS 7을 내려받읍시다. 

<img width="692" alt="Screenshot 2024-01-20 at 10 43 34 AM" src="https://github.com/bjlee72/devops/assets/4746751/c7e02312-a8e7-43d5-b2a8-267cb0644c55">

x86_64를 클릭해보면 미러 리스트가 나오는데 저는 페이스북 미러로 가서 [CentOS-7-x86_64-Minimal-2009.iso](https://mirror.facebook.net/centos/7.9.2009/isos/x86_64/CentOS-7-x86_64-Minimal-2009.iso)를 받았습니다. 

## 가상 하드디스크 생성

다운로드가 끝나면 다음으로 해야 할 일은 운영체제를 설치할 가상 하드디스크를 만드는 것입니다. 저는 20G짜리 디스크를 만들었습니다.

```
qemu-img create -f qcow2 linux_hd.qcow2 20G
```

## 운영체제 설치

그리고 나면 이제 해당 하드디스크 위에 설치를 진행합니다. x86 에뮬레이션 계층 위에 방금 내려받은 이미지를 cdrom 형식으로 마운트해서 설치를 진행한다는 것을 알 수 있습니다. 

```
qemu-system-x86_64 -m 1G -smp 4 -accel tcg -hda linux_hd.qcow2 -cdrom CentOS-7-x86_64-Minimal-2009.iso -boot d
```

여기까지 하면 이제 여러분의 맥 화면에 친숙한(?) CentOS 설치 화면이 뜰 것입니다. 어지간하면 전부 디폴트로 설정한 다음에 운영체제 설치를 완료합시다. 
루트 사용자 패스워드, 그리고 개인 계정 이름과 패스워드는 잊지 않도록 메모해 두는 것이 좋습니다.

설치를 완료하면 재부팅이 될텐데 아마 재부팅 후에는 다시 설치화면이 나올 것입니다. 아까 cdrom을 마운트한 상태가 그대로 남아있기 때문인데요. 
설치가 끝나서 다시 부팅된 운영체제를 종료한 다음 위의 명령을 다음과 같이 변경한 다음에 다시 실행하면 이제 설치 메시지 없이 방금 설치한 운영체제가 시동될 것입니다.

```
qemu-system-x86_64 -m 1G -smp 4 -accel tcg -hda linux_hd.qcow2 -boot d
```

## 네트워크 설치

재부팅 되고 난 뒤에는 아직 네트워크는 동작하지 않는 상태입니다. 가상 머신 위에서 도는 CentOS VM이 호스트 운영체제의 네트워크에 접속할 수 있도록 해 주면 되는데요.
그러려면 이더넷 인터페이스를 활성화 해 주고 DHCP로 IP 주소를 받아올 수 있도록 해 줘야 합니다. 다음 링크를 참조하면 됩니다. 

[https://whmcsglobalservices.com/how-to-setup-network-after-rhel-centos-7-minimal-installation/
](https://whmcsglobalservices.com/how-to-setup-network-after-rhel-centos-7-minimal-installation/)

## 빌다 설치

이제 빌다를 설치해 볼 순서인데요. su 한 다음, 아래 명령어를 실행합니다. 
왜 그래야 하는지를 이해하지 못하겠다면 당신은 직업을 잘못 고른 것입니다.

```
yum install -y buildah
```

별 문제 없이 설치는 잘 끝나야 합니다.

## 이미지 생성 확인

이제 빌다로 이미지 레이어 생성이 가능한지 확인해 봅시다. 다음 명령어를 실행해 봅니다. 역시 CentOS에서 실행해야 합니다. 
주의할 것은 루트 계정에서 해야 한다는 점입니다. 일반 사용자 계정에서 진행하려면 네임스페이스 활성화 등 매만져야 할 부분이 더 있는데,
본 안내서의 목적과는 다른 부분이어서 여기서는 설명하지 않습니다.

```
buildah from centos
```

이렇게 하면 다음과 같은 메시지를 보게 될 것입니다. 이미지 하단에는 `buildah images` 명령어로 캐시에 보관된 이미지 목록을 출력한 결과도 포함되어 있으므로 유의하시기 바랍니다. 

<img width="602" alt="Screenshot 2024-01-20 at 10 59 23 AM" src="https://github.com/bjlee72/devops/assets/4746751/ce6d7e53-e3c2-4773-9116-a9063a466a8e">

