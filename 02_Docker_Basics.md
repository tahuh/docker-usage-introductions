# 도커의 기본

## 0. 개념

Docker는 일종의 VM Ware와 같은 가상환경을 제공해 주는 도구이다. 그러나 VM Ware와 같은 도구들은 host machine(내가 가지고 있는 물리적인 컴퓨터)의 리소스를 사용하는 방식이 Guest OS위에서 돌아가기 때문에 overhead가 크다는 단점이 있다

반면 Docker는 그런것이 없어서 overhead가 없어서 동작이 빠르다.

여기 [링크](https://geekflare.com/docker-vs-virtual-machine/)에서 자세한 사항을 확인하자.

참고로 docker를 이용해서 만든 가상환경을 `docker container` 라고 한다. Docker 홈페이지에 들어가면 왠 고래 한마리(!)가 등에 뭘 짊어지고 가는데 그게 `container`다

~(은근히 근데 고래 귀여움!)~

## 1. 설치

운영체제에 맞게 잘 설치하자.

- [Windows](https://docs.docker.com/docker-for-windows/install/)
- [MacOS](https://docs.docker.com/docker-for-mac/install/)
- [Ubuntu](https://docs.docker.com/engine/install/ubuntu/)
- [CentOS](https://docs.docker.com/engine/install/centos/)

NOTE. Windows의 경우에는 WSL2가 깔려 있어야 함

WSL2를 설치하는 방법은 다음의 [링크](https://docs.microsoft.com/en-us/windows/wsl/install-win10)를 참고하자.


## 2. 간단한 Docker container 만들기

이제 우리는 docker container를 만들기 위해 `Dockerfile`이라고 부르는 단 한장의 파일을 만들 것이다.

다음의 내용을 텍스트 에디터로 기록하고 어디 경로에 저장해 보자

```Dockerfile
FROM ubuntu:16.04 # We load base OS image as ubuntu-1604

RUN echo "Hello docker file" # Docker file will say hello
```

사실 위의 docker파일은 아무것도 하지 않고 오로지 build할 때 `Hello docker file`만 출력하게 하는 예제이다.

## 3. Bioinformatics에서 사용하는 software를 docker 구동하는 만들기 - BWA docker

이제 실제로 우리가 써먹을 도커를 만들어보기로 한다.

제일 먼저 bioinformatics를 할때 많이 쓰는 aligner인 BWA를 구동하는 docker container를 만들어 보자.

이를 위해 먼저 우리는 아래와 같은 Dockerfile을 만들 것이다

실습에 앞서 BWA의 repository를 clone해 두자.

```bash
git clone https://github.com/lh3/bwa
```

```Dockerfile
FROM ubuntu:16.04

MAINTAINER Thomas Sunghoon Heo

# Set-up environment required for installtion steps
ENV PATH /usr/local/bin:$PATH
ENV LANG C.UTF-8

# Set-up work directory
WORKDIR /bwawd
# Now copy all the things in here to bwawd
COPY . /bwawd
# Now update APT
RUN apt-get update
RUN apt-get install -y software-properties-common
RUN apt-get update

# We need some tools to compile BWA
# including external libraries

RUN DEBIAN_FRONTEND=noninteractive; \
apt install -y build-essential \
autoconf automake perl \
zlib1g-dev libbz2-dev \
liblzma-dev libcurl4-gnutls-dev \
libssl-dev libncurses5-dev git

# Installation following github instruction

RUN cd /bwawd/bwa && make
RUN ln -s /bwawd/bwa/bwa /usr/local/bin/bwa 
```

위의 도커 파일에 사용되는 `ENV`, `WORKDIR`, `COPY`, `RUN` 등은 Docker의 공식 홈페이지에서 사용법을 찾아보면 좋지만 같단히 적으면 아래와 같다

- ENV : ENVironment setting으로 환경변수 설정이다
- WORKDIR : 현재 Docker가 작업할 working directory로 파일들을 local host file을 copy하거나 다른 데이터 파일을 copy 할 때 지정해서 사용한다
- COPY : 보이는대로 host의 데이터를 복사하는 명령어 이다
- RUN : 뒤에 나오는 명령어를 실행한다.

위의 Dockerfile을 가지고 아래의 명령어를 입력하면 docker container가 만들어 진다

주의 사항은 맨 마지막에 있는 `.`을 빼먹지 말자

`docker build -t container/bwa .`

이제 docker container를 구동해 보자

`$ docker run --rm -ti container/bwa bwa`

그려면 이제 docker의 결과로 우리는 bwa를 구동한 것이된다.

이 뒤에 평소처럼 `bwa`의 옵션을 넣고 돌리면 된다

여기에 우리는 docker의 `tag`라는 특성을 이용해서 software의 버전을 지정 할 수 있다.

software의 버전에 따라 분석의 결과치가 달라질 수 있기 때문에 이런 버전을 명시하는 것이 중요하다.

이를 위해 아까의 Dockerfile을 이용하는데 내용은 바꿀 필요가 없다.

단지 아래의 `-t` 옵션을 잘 이용해 보면 된다

명령어는 다음과 같다 (Github의 bwa의 버전이 0.7.17 이므로 이걸 쓰자)

`docker build -t container/bwa:v0.7.17 .`

이렇게 하면 나중에 Cloud에서 docker기반으로 구현할 때도 같은 version을 이용할 수 있다.

또 이 방법을 통해 버전이 바뀔 때 버전에 해당하는 docker container들 각각을 얻을 수 있다.

## 마치며...
오늘 우리가 해 본 것은 software 하나를 docker container로 만든 것이다.

다른 software들도 개인적으로 해 보는 것을 추천한다
