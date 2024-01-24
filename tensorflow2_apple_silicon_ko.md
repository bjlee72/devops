# 애플 실리콘에 tensorflow2 설치

일단 tensorflow2가 요구하는 버전의 파이썬에 맞추기 위해 3.11.7 버전의 파이썬을 설치해보도록 하겠다.

```
brew install pyenv
pyenv install 3.11.7
...
mkdir tensorflow2
cd tensorflow2
pyenv local 3.11.7
```

여기까지 하면 tensorflow2 디렉터리 안에서는 파이썬 버전 3.11.7로 작업할 준비가 끝난다. 다만 여기까지 오는데 굉장히 장애물이 많을 수 있는데
그런 사람들은 디렉터리 어딘가가 단단히 꼬인 것이다. 대표적인 원인 가운데 하나로는 brew의 인텔 버전 패키지와 애플 실리콘 패키지가 뒤섞인 것을 들 수 있다.
그런 일이 벌어지면 일단 다음의 디렉터리를 전부 밀어버린 다음 brew의 설치 관리자를 (*.pkg) 다운받아서 처음부터 다시 설치해 볼 것을 권장한다.

```
sudo \rm -Rf /usr/local/bin/brew
sudo \rm -Rf /usr/local/Homebrew
sudo \rm -Rf /usr/local/Cellar/py* (파이썬 관련 디렉터리밖에 없는지 Cellar 디렉터리 내용을 다시 한번 확인할 것)
```

# tensorflow2 설치

