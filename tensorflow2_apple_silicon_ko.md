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

이렇게 밀어버린 다음에 brew를 다시 설치하고 위의 pyenv 설치 절차를 다시 따르면 된다.
유의할 것은 일단 brew를 다시 설치하고 나면 brew의 위치가 바뀌어 있을 수 있기 때문에
다음의 위치가 PATH에 있는지 확인할 필요가 있다는 것이다.

```
/opt/homebrew/bin
```

만일 없다면 PATH에 추가해 주도록 하자. 그래야 brew CLI를 불편함 없이 쓸 수 있다. 어떻게 해야 할지 모르겠다면 당신은 직업을 잘 못 고른 것이다.

# tensorflow2 설치

일단 위의 tensorflow2 디렉터리에서 다음을 실행한다.

```
# Requires the latest pip
python -m pip install --upgrade pip

# Current stable release for CPU and GPU
python -m pip install tensorflow
```

다음과 같이 정상 설치를 확인해 보자.

```
❯ python -c "import tensorflow as tf; x = [[2.]]; print('tensorflow version', tf.__version__); print('hello, {}'.format(tf.matmul(x, x)))"
```

다음과 같은 결과가 나오면 된다.

```
tensorflow version 2.15.0
hello, [[4.]]
```
