# qemu 위에서 실행중인 CentOS 가상 머신에 ssh를 하는 방법

qemu 위에서 CentOS 가상 머신을 돌리고 있다고 해 봅시다. 이 머신은 "Minimal" 버전이라고 하죠. 네트워크 기능은 활성화 되어 있다고 하겠습니다. 
어떻게 활성화 하는지 모르겠다면 이 문서의 "[네트워크 설치](https://github.com/bjlee72/devops/blob/main/mac_qemu_buildah_ko.md#%EB%84%A4%ED%8A%B8%EC%9B%8C%ED%81%AC-%EC%84%A4%EC%B9%98)" 부분을 참고하세요. 

아니 이미 가상 머신을 돌리고 있고 그 화면도 뻔히 볼 수 있는데 왜 굳이 거기에 ssh를 해야 하나요 같은 질문을 하실 수도 있는데요.
일단 CentOS가 마니멀 버전이라 UI가 뜨지 않거나 하면 터미널을 여럿 띄울 수 없다는 문제가 있을 수 있고요.
게다가 설상가상으로 그 화면을 텍스트로 캡처라도 떠야 하는 상황이라면 아마 ssh가 가능하도록 하는게 제일 저렴한 솔루션일 거에요.

## sshd 설치

일단 CentOS에 sshd를 설치해야 합니다. [이 문서](https://phoenixnap.com/kb/how-to-enable-ssh-centos-7)를 참고하세요. 

## qemu 세션 재시작

```
qemu-system-x86_64 -m 1G -smp 4 -accel tcg -hda linux_hd.qcow2 -boot d -net user,hostfwd=tcp::2222-:22 -net nic,model=virtio
```

`hostfwd` 부분은 호스트에서의 포워딩 규칙을 설정하는 부분입니다. 즉, 호스트의 2222번 포트가 게스트 OS의 22번 포트로 매핑됩니다. 

이렇게 하면 다음 ssh 접속 시도는 성공할 것입니다.

```
ssh bjlee72@localhost -p 2222
```

물론 여기서 bjlee72는 여러분 자신의 계정 이름으로 바꿔야 합니다 설마 그것도 모른다고 하진 않겠지 만일 그렇다면 당신은 잘못된 직업을 택한 것입니다  
