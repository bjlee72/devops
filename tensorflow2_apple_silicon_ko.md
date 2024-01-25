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
위의 명령에서 한가지 눈여겨 볼 부분은 tensorflow2 디렉터리 안에 들어가면 자동으로 python 3.11.7을 쓰도록 설정해 두었다는 것이다(`python local 3.11.7`).
만일 여러분이 위의 과정을 실행하기 전에 pyenv로 3.7.x의 낮은 버전 파이썬을 pyenv로 설치하고 global로 지정해 두었다고 해 보자(가령 `pyenv global 3.7.11` 등으로).
아마 회사 전체의 python 환경이 3.7.11이라던가 하는 다양한 이유가 있을 것이다.

```
blee-mbp-w2c :: ~ » pyenv install 3.7.11
python-build: use openssl from homebrew
python-build: use readline from homebrew
Downloading Python-3.7.11.tar.xz...
-> https://www.python.org/ftp/python/3.7.11/Python-3.7.11.tar.xz
Installing Python-3.7.11...
patching file 'Misc/NEWS.d/next/Build/2021-10-11-16-27-38.bpo-45405.iSfdW5.rst'
patching file configure
patching file configure.ac
python-build: use readline from homebrew
python-build: use zlib from xcode sdk
Installed Python-3.7.11 to /Users/bjlee72/.pyenv/versions/3.7.11
blee-mbp-w2c :: ~ » pyenv global 3.7.11
blee-mbp-w2c :: ~ » which python
/Users/bjlee72/.pyenv/shims/python
```

그러면 이제 여러분은 tensorflow2 디렉터리 안에서만 3.11.7 파이썬을 쓰게 된다. 

```
Last login: Thu Jan 25 15:48:36 on ttys001
blee-mbp-w2c :: ~ » python --version
Python 3.7.11
blee-mbp-w2c :: ~ » cd tensorflow2
blee-mbp-w2c :: ~/tensorflow2 » python --version
Python 3.11.7
blee-mbp-w2c :: ~/tensorflow2 »
```

다만 여기까지 오는데 굉장히 장애물이 많을 수 있는데 그런 사람들은 디렉터리 어딘가가 단단히 꼬인 것이다. 
대표적인 원인 가운데 하나로는 brew의 인텔 버전 패키지와 애플 실리콘 패키지가 뒤섞인 것을 들 수 있다.
그런 일이 벌어지면 일단 애플 실리콘 사용자는 다음과 같이 하여 인텔 디렉터리를 전부 밀어버린 다음 brew의 설치 관리자를 (*.pkg) 다운받아서 처음부터 다시 설치해 볼 것을 권장한다.

```
# 홈브루 인텔 버전 파일은 /usr/local 밑에 깔림
sudo \rm -Rf /usr/local/bin/brew
sudo \rm -Rf /usr/local/Homebrew
sudo \rm -Rf /usr/local/Cellar/py* (py로 시작하는 디렉터리는 오직 파이썬 관련 디렉터리 뿐인지 지우기 전에 다시 한번 확인할 것)
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
python -c "import tensorflow as tf; x = [[2.]]; print('tensorflow version', tf.__version__); print('hello, {}'.format(tf.matmul(x, x)))"
```

다음과 같은 결과가 나오면 된다.

```
tensorflow version 2.15.0
hello, [[4.]]
```
