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

