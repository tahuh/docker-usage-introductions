# 왜 Docker 인가?

학교 연구실(~대학원이라고 쓰고 할말하않...~) 혹은 산업계의 연구소에서 실험한 분들이 밤낮을 고생하여서 실험을 하고 그 결과로 **_소듕한_** NGS데이터를 손에 얻었다.

이제 해야할 일은 NGS 데이터를 목적에 맞게 분석하여 실험자에게 그 결과를 전달해 줘야 한다.

목적은 여러가지가 있겠지만 일단 금방 생각해 볼 수 있는 것들은 아래의 사항들일 것이다.

1. 어떤 생물학적인 현상에 대한 가설에 대한 검증

2. NGS technology를 이용한 실험 기법의 개발 성공 유무

3. 대규모 코호트 분석(Large cohor analysis)를 통한 유의한 발견(Cancer assiciated variants, Disease associated genotypes, etc..)

4. Precision medicine (or personalized medicine)

5. 이런 저런 다른 이유들... ~누가(라고 하고 어려운 그 분들) 그냥 시켰다거나...~

위에 언급된 사항들에서 3번과 4번의 경우 모든 샘플에 대해서 같은 분석법 (우리는 이걸 보통 `pipeline`이라고 한다)을 이용해서 분석을 해야한다.

그 이유는 샘플마다 분석법이 다르면 분석법에 따른 편향 때문에 올바른 결과를 얻을 수 없기 때문이다.

이러한 `pipeline`을 만들기 위해 우리는 여러가지 분석 도구들을 우리의 서버에 설치하게 된다.

**이때 문제가 발생한다**

**상황 1. 라이브러리 충돌 문제** 

이때 특정 도구가 서버에 깔려있는 library와 호환이 되지 않아서 library를 downgrade 혹은 upgrade하게 된다.

NGS 분석에서 빠질 수 없는 라이브러리로는 `htslib` library인 것은 누구나 잘 알 것이다.

서버를 구매하고(이때까지는 매우 신난다) 이제 신나게 `htslib`을 깔았다.

만약 이 라이브러리가 사용하려는 도구와 호환이 안된다면? [여기](https://github.com/dpryan79/MethylDackel/issues/99) , [docker file](https://github.com/tahuh/MethylDackel_Docker)

이러면 문제가 된다.

**상황 2. OS 지원문제**

후우..... _제일 ~빡친다.~_

주로 사용하던 software가 설치한 OS를 지원하지 않는다면?

대부분의 많은 bioinformatics tool은 기본적으로 UNIX/Linux 계열의 OS에서는 어지간해서는 다 성공적으로 설치가 된다.

Bioinformatics의 가장 처음 시작은 보통 sequencing read를 trimming하고 reference genome에 mapping하는 일에서 많은 경우 시작할 것이다.

이럴 때 많은 양의 데이터를 한번에 처리하기 위해 `job scheduler or workload manager`를 설치하고 사용하게 된다 (`slurm`, `gridengine 계열` 등)

설치가 끝나면 이제 마구마구 들어오는 sequencing sample을 최대한 빨리 분석하기 위해 많은 수의 job을 투척해서 CPU와 RAM을 괴롭힌다~(으흐흐!!)~.

**그.런.데.** Sun Grid Engine (SGE)을 Ubuntu에서 특정 버전 이후에 설치하려고 하면 잘 되지 않는 경우가 있다. [여기1](https://shajoezhu.github.io/blog/2019/sge-compile/),[여기2](https://bugs.launchpad.net/ubuntu/+source/gridengine/+bug/1774302), [여기3](https://arc.liv.ac.uk/trac/SGE/ticket/1632)

(사실 나도 이거 설치하다가 1주일 날려먹음은 안 비밀임.)

`slurm`이 [여러OS](https://slurm.schedmd.com/platforms.html)를 지원하지만, install instruction을 보면 `CentOS` 나 `RedHat`계열의 운영체제를 기본으로 설명을 하고 있다

**상황 3. Software version 문제**

분석을 하다보면 같은 데이터를 분석하게 되더라도 사람에 따라 소프트웨어 버전을 다른 것을 사용할 수 있다.

이 상황이 문제가 되는 경우는 다음의 상황에 따른 재현성의 문제이다.

+ 기능 지원의 문제

  + 예를 들어 `samtools`를 보면 예전 samtools가 버전이 올라오면서 없던 기능들이 추가되고 있던 기능도 사용법이 바뀌었다.[링크](https://github.com/samtools/samtools/releases/)

  + 어떤 두 분석가 A씨와 B씨가 sequencing coverage를 구하려고 한다.

  + A 씨는 `1.9 version`의 samtools를 사용하고 B씨는 `1.10 version`의 samtools를 이용해서 분석을 수행한다고 하자. B씨의 경우 samtools 의 최신을 쓰기 때문에 `samtools coverage` 옵션이 있어서 coverage를 계산했고 A씨는 이 기능이 없어서 다른 tool인 bedtools를 이용하여 계산을 했다.

  + 두 분석 프로그램이 다르기 때문에 결과로 주는 output이 format 뿐만 아니라 계산된 수치도 달라질 것이다. 이는 이 coverage를 이용해서 추후 분석(data visualization, sequencing coverage threshold decision)을 할 때 문제가 발생한다.

+ 알고리즘의 변화로 인한 성능 변화
  + 똑같은 software를 사용해도 major algorithm의 변화로 인해 version이 바뀌는 경우에는 그 성능 변화로 인해 똑같은 분석이 안된다.

+ 버그 문제
  + 버그가 있는 부분이 fix가 되지 않고 release되는 software들이 아주 많다. 이러한 버그가 분석에 영향을 주게 된다.

언급된 상황들을 해결하기 위해서 `conda` + `snakemake` 의 조합으로 pipeline이 구성되지만 이 모든 것은 하나의 컴퓨터에서 이루어진다.

### 그럼 conda + snakemake를 가지고 하면 되는걸 왜 굳이 Docker를 써야하는가?

`conda` + `snakemake` 환경에서 내가 생각하는 단점은 바로 **서버를 이전하면 처음부터 다시 해야한다** 라는 것이다.

`conda` + `snakemake` 환경은 결국 한 컴퓨터에 OS를 설치한 후에 `conda`를 설치하고 사용했던 환경을 다시 만드는 과정이 필요하다.

**그런데** `docker`는 이미 그 자체가 OS이자 컴퓨터 하나이기 때문에 Dockerfile으로 기록된 형식, `tar` 파일로 저장된 image, 또는 공개된 image 만 있으면 마치 우리가 windows를 설치할 때 USB를 꼽고 설치하듯이 아무데서나 설치해서 사용할 수 있다.

다시말해 **한번 만들면 어디에서든 쓸 수 있다**

**추가로** 우리는 이제 <u>Cloud의 시대</u>에 접어 들었고 cloud에서 분석 및 서비스 하는 platform들이 `docker`를 기본적으로 지원한다. 대표적으로 Google의 Verily와 Broadinstitute이 합작한 [Terra](https://terra.bio/)가 있다.~(시대가 원한다 이말이야!)~

결론적으로 1) 귀찮은 설치 작업 줄이기 2) 분석의 재현성 확립 3) Cloud system에서의 효율적 운용 을 위해서 Docker system의 bioinformatics로의 도입은 정말 <u>필요</u>하다.

다음부터는 docker의 설치 및 간단한 build 작업 과 실전에서 docker를 어떻게 쓰는지 사용법 위주로 진행해 보고자 한다
