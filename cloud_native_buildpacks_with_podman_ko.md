# 포드맨 환경에서 클라우드 네이티브 빌드팩으로 컨테이너 이미지 빌드하기

포드맨(podman.io)은 도커의 drop-in-replacement로 동작할 수 있도록 설계된 컨테이너/이미지 관리자입니다. 
그러니까 무슨 말이냐면 도커 데스크탑을 설치한 분은 도커 데스크탑을 끄고 포드맨 데스크탑을 돌려도 거의 차이를 느낄 수 없어야 한다는 이야기죠.
명령어도 거의 호환됩니다. 그러니까 도커에 `docker images` 명령이 있다면 포드맨에는 `podman images` 명령어가 있습니다.
따라서 셸 환경에서 podman의 alias로 docker를 지정해도 어지간하면 안 되는 부분은 없어야 합니다.

클라우드 네이티브 빌드팩은 도커파일 없어도 이미지를 만들 수 있도록 해 주긴 하는데, 이미지가 로컬 캐시에 저장될 떄 도커를 가정하고 만들어져 있습니다.
따라서 포드맨 같은 도구를 설치해두면 유용하죠. 게다가 포드맨 데스크탑은 qemu 기반이긴 하지만 맥에서도 돌아가도록 만들어져 있습니다.
따라서 포드맨을 설치하면 여러가지 이점이 있습니다. 

## 포드맨 설치

맥에서 포드맨 설치는 홈브루로 할 것을 권장하지 않습니다. podman.io로 가서 Mac용 데스크탑을 다운로드 받고 설치합시다.

<img width="776" alt="Screenshot 2024-01-22 at 8 59 01 PM" src="https://github.com/bjlee72/devops/assets/4746751/01623f07-4de9-4465-ad7a-9a33b3bf7c77">

설치하고 실행해 보면 다음과 같은 화면이 뜹니다. "Go to Podman Desktop"을 누릅시다. 

<img width="1031" alt="Screenshot 2024-01-22 at 9 07 43 PM" src="https://github.com/bjlee72/devops/assets/4746751/eee00e16-6535-4b82-8375-9b05f5e2804f">

Podman을 설치한 적이 없으므로 설치하라고 나옵니다. 두 개를 따로 설치해야 한다니 웃기는 짬뽕이라는 생각이 들지만 잠자코 따라해 보겠습니다. 인스톨 버튼을 누르면 됩니다.

<img width="793" alt="Screenshot 2024-01-22 at 9 08 27 PM" src="https://github.com/bjlee72/devops/assets/4746751/3b6f7c54-ab60-473b-ab84-ca54609edb2a">

우리에게 친숙한 맥 설치 화면이 떠서 Podman 설치 과정을 안내할 것입니다. 그 상태에서 뭘 어찌해야 할 지 모르겠다면 당신에게는 맥이 어울리지 않는 것입니다.

<img width="798" alt="Screenshot 2024-01-22 at 9 10 41 PM" src="https://github.com/bjlee72/devops/assets/4746751/217d2534-7c9d-4a30-a5b7-f041c36e1f9e">

초기화하고 시작하라니까 잠자코 시키는 대로 하기로 합시다. 버튼을 좀 누르고 기다리면 아래 세 화면이 순서대로 보이면서 podman이 running 상태가 됩니다. 

<img width="791" alt="Screenshot 2024-01-22 at 9 11 03 PM" src="https://github.com/bjlee72/devops/assets/4746751/45ca10e7-66c8-46e1-8cfb-de28a21d0fe1">
<img width="796" alt="Screenshot 2024-01-22 at 9 11 40 PM" src="https://github.com/bjlee72/devops/assets/4746751/0e7e4c46-b266-489f-a630-5cee34240245">
<img width="795" alt="Screenshot 2024-01-22 at 9 12 44 PM" src="https://github.com/bjlee72/devops/assets/4746751/5b12a13a-248f-4a1a-8e45-cf26cb748c35">

그 상태에서 설정 버튼을 누르면 podman machine이 실행중임을 확인할 수 있습니다. 

<img width="903" alt="Screenshot 2024-01-22 at 9 13 24 PM" src="https://github.com/bjlee72/devops/assets/4746751/76750657-4bbb-44ab-8545-1aa0ed6ccdc0">

이제 가장 간단하게 제대로 돌고 있음을 확인하는 방법은 무엇일까요? 앞서 말한 대로 `podman images`나 `podman info`를 실행해서 정상적으로 메시지가 출력되는지 보는 것입니다.

<img width="428" alt="Screenshot 2024-01-22 at 9 15 06 PM" src="https://github.com/bjlee72/devops/assets/4746751/50f6bef7-32bc-4221-953d-851133fe2eb3">

`podman images` 정도가 생각나지 않는다면 거듭 말하지만 당신은 직업을 잘못 택한 것입니다. 

## 클라우드 네이티브 빌드팩을 통해 예제 프로그램 이미지 빌드

### 빌드팩 설치

설치 방법은 https://buildpacks.io/docs/tools/pack/ 에 잘 나와 있습니다. 

### 예제 프로그램 빌드

예제 프로그램으로는 https://github.com/gitops-cookbook/chapters/tree/main/chapters/ch03/nodejs-app 에 나와 있는 예제를 시도해보겠습니다. 

해당 리파지터리를 로컬에 클론(clone)한 다음에 해당 디렉터리로 이동합시다. 그런 다음 아래의 명령어를 일단 시도합니다. 

```
pack builder suggest
```

이 소스코드에 맞는 이미지 빌더를 소개해달라는 뜻입니다. 대략 다음과 같은 메시지가 화면에 출력될 것입니다.

```
Suggested builders:
	Google:                gcr.io/buildpacks/builder:v1                            Ubuntu 18 base image with buildpacks for .NET, Go, Java, Node.js, and Python
	Heroku:                heroku/builder:20                                       Heroku-20 (Ubuntu 20.04) base image with buildpacks for Go, Java, Node.js, PHP, Python, Ruby & Scala.
	Heroku:                heroku/builder:22                                       Heroku-22 (Ubuntu 22.04) base image with buildpacks for Go, Java, Node.js, PHP, Python, Ruby & Scala.
	Paketo Buildpacks:     paketobuildpacks/builder-jammy-base                     Ubuntu 22.04 Jammy Jellyfish base image with buildpacks for Java, Go, .NET Core, Node.js, Python, Apache HTTPD, NGINX and Procfile
	Paketo Buildpacks:     paketobuildpacks/builder-jammy-buildpackless-static     Static base image (Ubuntu Jammy Jellyfish build image, distroless-like run image) with no buildpacks included. To use, specify buildpacks at build time.
	Paketo Buildpacks:     paketobuildpacks/builder-jammy-full                     Ubuntu 22.04 Jammy Jellyfish full image with buildpacks for Apache HTTPD, Go, Java, Java Native Image, .NET, NGINX, Node.js, PHP, Procfile, Python, and Ruby
	Paketo Buildpacks:     paketobuildpacks/builder-jammy-tiny                     Tiny base image (Ubuntu Jammy Jellyfish build image, distroless-like run image) with buildpacks for Java, Java Native Image and G
```

이 가운데 `paketobuildpacks/builder-jammy-base`를 빌더로 이용해 이미지 빌드를 시도해 보겠습니다.

```
pack build nodejs-app --builder paketobuildpacks/builder-jammy-base
```

다음과 비슷한 메시지가 화면에 출력되면서 빌드가 끝납니다.

```
...
===> BUILDING

Paketo Buildpack for CA Certificates 3.6.6
  https://github.com/paketo-buildpacks/ca-certificates
  Launch Helper: Contributing to layer
    Creating /layers/paketo-buildpacks_ca-certificates/helper/exec.d/ca-certificates-helper
Paketo Buildpack for Node Engine 3.0.1
  Resolving Node Engine version
    Candidate version sources (in priority order):
                -> ""
      <unknown> -> ""

    Selected Node Engine version (using ): 20.9.0

  Executing build process
    Installing Node Engine 20.9.0
      Completed in 18.428s

  Generating SBOM for /layers/paketo-buildpacks_node-engine/node
      Completed in 3ms

  Configuring build environment
    NODE_ENV     -> "production"
    NODE_HOME    -> "/layers/paketo-buildpacks_node-engine/node"
    NODE_OPTIONS -> "--use-openssl-ca"
    NODE_VERBOSE -> "false"

  Configuring launch environment
    NODE_ENV     -> "production"
    NODE_HOME    -> "/layers/paketo-buildpacks_node-engine/node"
    NODE_OPTIONS -> "--use-openssl-ca"
    NODE_VERBOSE -> "false"

    Writing exec.d/0-optimize-memory
      Calculates available memory based on container limits at launch time.
      Made available in the MEMORY_AVAILABLE environment variable.

Paketo Buildpack for NPM Install 1.3.1
  Resolving installation process
    Process inputs:
      node_modules      -> "Not found"
      npm-cache         -> "Not found"
      package-lock.json -> "Found"

    Selected NPM build process: 'npm ci'

  Executing launch environment install process
    Running 'npm ci --unsafe-perm --cache /layers/paketo-buildpacks_npm-install/npm-cache'

      up to date, audited 1 package in 1s

      found 0 vulnerabilities
      Completed in 2.599s

  Configuring launch environment
    NODE_PROJECT_PATH   -> "/workspace"
    NPM_CONFIG_LOGLEVEL -> "error"
    PATH                -> "$PATH:/layers/paketo-buildpacks_npm-install/launch-modules/node_modules/.bin"

  Generating SBOM for /layers/paketo-buildpacks_npm-install/launch-modules
      Completed in 27ms


Paketo Buildpack for Node Start 1.1.3
  Assigning launch processes:
    web (default): node server.js

===> EXPORTING
Adding layer 'paketo-buildpacks/ca-certificates:helper'
Adding layer 'paketo-buildpacks/node-engine:node'
Adding layer 'paketo-buildpacks/npm-install:launch-modules'
Adding layer 'buildpacksio/lifecycle:launch.sbom'
Adding 1/1 app layer(s)
Adding layer 'buildpacksio/lifecycle:launcher'
Adding layer 'buildpacksio/lifecycle:config'
Adding layer 'buildpacksio/lifecycle:process-types'
Adding label 'io.buildpacks.lifecycle.metadata'
Adding label 'io.buildpacks.build.metadata'
Adding label 'io.buildpacks.project.metadata'
Setting default process type 'web'
Saving nodejs-app...
*** Images (e325aac437d9):
      nodejs-app
Adding cache layer 'paketo-buildpacks/node-engine:node'
Adding cache layer 'paketo-buildpacks/npm-install:npm-cache'
Adding cache layer 'buildpacksio/lifecycle:cache.sbom'
Successfully built image nodejs-app
```

이제 이미지 빌드가 제대로 되었는지는 무슨 명령으로 확인해야 하죠? 그 명령어가 생각났다면 다음과 같은 메시지를 보게 될 것입니다.

```
REPOSITORY                                     TAG         IMAGE ID      CREATED       SIZE
docker.io/paketobuildpacks/run-jammy-base      latest      4414f36bf863  3 hours ago   113 MB
docker.io/paketobuildpacks/builder-jammy-base  latest      07a12e865f08  44 years ago  1.59 GB
docker.io/library/nodejs-app                   latest      e325aac437d9  44 years ago  303 MB
```

