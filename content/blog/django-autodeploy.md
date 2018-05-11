+++
title = "AWS Elastic Beanstalk + CI 를 활용한 Django 배포 자동화"
tags = ["DevOps", "AWS", "CI", "자동화"]
categories = ["DevOps", "AWS", "Elastic Beanstalk", "자동화"]
date = "2018-02-02 15:19:45"
+++

최근에 프로젝트를 진행하다가 알게 된 CI 와 배포 자동화 기법을 사용해서 AWS 배포 자동화를 설정해본 결과, 엄청 편하고 한번만 알아두면 두고두고 사용할 수 있을 것 같아서 이렇게 포스트로 정리하게 되었습니다!

![](/img/aws.png)

<!-- more -->

이 포스트를 쓰기 전까지만 해도 나는 배포를 할때 이런 과정들을 밟았다.

1. 로컬에서 테스트를 완료한다
2. AWS 에 배포환경을 구축한다.
3. 압축해서 한땀한땀 올리거나 command line 을 사용해서 손수 배포한다.

이 과정을 배포할 때 마다 하려니깐 생각보다 은근 귀찮았다. 단순 반복 작업이긴 하지만, github 에 푸시도 하고 배포도 할려니깐 똑같은 일을 두 번 하는 느낌도 들었다.  
그러다가 어느 날 같이 프로젝트를 하는 개발 잘하는 형이 배포를 자동화 할 수 있다는 것을 알려주셔서 약간의 구글링과 ~~수많은 삽질을~~ 통하여 AWS Elastic Beanstalk 에 프로젝트를 자동으로 배포하는 데 성공하였다. 그 다음부터는 `origin/master` 에 푸시만 해도 자동으로 배포가 되니 엄청 편했다. 한 번만 고생하면 두고두고 써먹을 수 있을 것 같아서 이렇게 포스팅까지 하게 되었다. 

이제, CI 를 활용하여 Elastic Beanstalk 에 자동으로 배포하는 방법을 소개한다!  
역시나, 예제는 필자가 가장 좋아하는 `django` 를 활용하였다. (`node` 나 `spring` 같은 프로젝트도 세부 설정만 다르지 전체적인 process 는 동일합니다.)

## 목차

글이 상당히 길어, 스크롤 압박이 있습니다. 다음 목차를 참조하시어 필요하신 부분만 선택적으로 읽으실 수도 있습니다

- CI는 무엇인가
- AWS Elastic Beanstalk 란?
- 배포 설정해보기
    - Elastic Beanstalk 어플리케이션과 환경 만들기
    - CircleCI 빌드 구성하기
    - 배포 설정하기
    - 배포 실행해보기!
- 참고자료
- (이 글보다 더 좋은) 참고해 볼 만한 글들

## CI 는 무엇인가

우선, CI 가 무엇인지, 뭐하는 친구인지부터 알고 갈 필요가 있다.  
CI 는 Continuous Integration 의 약자로, `git` 등의 (요새는 거의 깃이다) 형상관리 시스템에 코드를 커밋(변경 사항 적용이라고 생각하면 쉽습니다) 할 때마다, 코드의 빌드 및 테스트를 지원하는 시스템이다. 

`git` 의 어느 브랜치에 푸시가 되든 간에 CI 는 알아서 빌드를 돌려서, 성공 여부 및 실패원인을 알려준다. 이렇게 지속적으로 테스트를 진행하여 코드의 품질을 높이는 것이라고 이해하면 쉽겠다.  
마지막으로, `master` 브랜치에 최종으로 코드를 합치고 나면 배포 과정을 실행하도록 할 수 있다.

CI의 예를 들어보자면,

- [Jenkins](https://jenkins.io/)
- [Travis CI](https://travis-ci.org/)
- [CircleCI](https://circleci.com/)

등이 있는데, 이 포스트에서는 `CircleCI`를 사용해보도록 하겠다. `BitBucket` 을 지원하고, 설정도 타 CI 에 비해 간편해서 사용하게 되었다. (물론 다른 CI 가 안 좋다는 소리는 절대 아닙니다. Travis CI 를 드랍한 이유는 BitBucket 을 지원하지 않아서 그렇습니다. 추후 포스팅으로 Travis CI 와 CircleCI 를 주관적으로 비교한 내용을 다루도록 하겠습니다.)
 
## AWS Elastic Beanstalk 란?

우선, AWS(아마존 웹 서비스) 에는 가장 많이 쓰이는 세 가지의 서비스가 있는데,

- EC2 : 클라우드 인스턴스이다. 여기서 서버를 설정하면 public ip 및 도메인이 할당된다.
- S3 : 코드 및 정적 리소스(이미지 등) 을 올려둘 수 있는 저장소이다
- RDS : 관계형 데이터베이스 서비스이다. `MySQL`, `PostgreSQL` 등의 관계형 데이터베이스를 둘 수 있다.

이 세 가지를 통합하여 빠르게 시작할 수 있도록 제공해주는 서비스가 바로 Elastic Beanstalk 이다.  
서비스 구성이 상당히 빠르고, 또한 원한다면 각 서비스마다(EC2, RDS 등을 지칭) 세부 설정 또한 가능해서 필자가 상당히 애용하는 서비스이다.

무엇보다도, 필자가 개인적으로 생각하는 Elastic Beanstalk 의 최대 매력은 바로 command line 의 존재이다. 터미널 명령어 몇 줄 만으로 서비스 배포가 가능하기 때문에 상당히 편리하며, 또한 **이 명령어를 스크립트로 만들어서 CI 에서 실행시켜 주면 그것이 바로 자동 배포이다!**

Elastic Beanstalk 를 사용하는 데 추가 요금은 들지 않으며, 서비스를 구성하는 인스턴스들을 사용한 요금만 지불하면 된다.

## 배포 설정해보기

위의 문단들에서 CI 와 Elastic Beanstalk 에 대하여 간단하게 알아보았으니, 이제 본격적으로 배포 자동화 설정을 하는 방법을 알아보자!  
(기본적으로 AWS 계정이 있는 상태라고 간주하고 내용을 작성하겠습니다.)

### Elastic Beanstalk 어플리케이션과 환경 만들기

#### 어플리케이션 만들기

우선, 가장 먼저 Elastic Beanstalk 을 시작해보자. [공식 문서](https://docs.aws.amazon.com/ko_kr/elasticbeanstalk/latest/dg/create-deploy-python-apps.html) 를 참조하여 Elastic Beanstalk 에 파이썬 환경을 구축하면 된다.

"새 애플리케이션" 을 클릭한 다음, 이름을 입력하면 어플리케이션이 만들어진다.

그 다음, "새 환경 생성" 을 클릭하면 환경 설정을 할 수 있는데, 이 포스트에서는 파이썬 환경을 구축해보도록 하겠다.  
아래 사진과 같이, 플랫폼만 `Python` 을 선택해주고, 나머지 설정값들은 그대로 두자.
![](/img/create-new-environment.png)

맨 밑의 "환경 생성" 을 클릭하고 커피 한 잔 하고 오면, 샘플이 배포되어 있을 것이다. 해당 URL 로 들어가보자.
![](/img/sample-deployment.png)

위의 사진과 같이 초록색 체크박스가 뜨고, URL 로 접속했을 때 Congratulations 메시지가 뜨면 성공이다!

#### 환경 설정하기

다음으로, 프로젝트에 Elastic Beanstalk 환경을 설정해보자.  
기존 프로젝트가 존재한다면, 기존 프로젝트의 디렉토리로 이동하여 가상환경을 활성화시키자. 그게 아니라면, 다음 과정을 통하여 새로운 프로젝트를 생성하자.

**Git 저장소로 생성하기**

우리는 git 저장소에 코드가 커밋 되었음을 인지하면, 빌드 및 배포를 돌리는 CI 를 사용하므로, git 과 git 저장소를 사용하는 것이 중요하다. 

다음 명령어로 새로운 git 로컬 저장소를 생성해주자.

```
$ git init
```

로컬 저장소를 생성하였으면 원격 저장소도 만들어 주어야 하는 법. 깃허브 저장소가 있을 것이라 가정하고(없으면 만드세요, 만드는 것이야 어렵지 않습니다), 다음 명령어를 통하여 원격 저장소를 추가해주자.

```
$ git remote add origin <YOUR_REPO_ADDRESS>
```

혹시나 있을 원격 저장소와의 충돌 방지를 위해서,

```
$ git pull origin master
```

명령어로 pull 도 한 번 실행해주자.

**가상환경 생성하기**

파이썬의 의존 관리(dependency management) 를 위하여 거의 필수적으로 생성해 주어야 하는 것이 가상환경이다. 다음 과정을 통하여 가상환경을 생성하자.

우선, 가상환경 패키지(virtualenv) 가 설치되어 있지 않을 경우에는, 다음 명령어로 설치해주자. 

```
$ pip install virtualenv
$ pip3 install virtualenv
```

가상환경을 설치했다면, 다음 명령어로 가상환경을 활성화 시켜주자.

```
$ python3 -m venv venv
$ source venv/bin/activate
```

위의 명령어는 `venv` 라는 가상환경 폴더를 생성해주는 명령어이고, 아래의 명령어는 가상환경을 활성화 시켜주는 명령어이다.  
가상환경을 활성화 시켰다면 다음과 같이 `django` 를 설치해주자!

```
(venv) $ pip install django
```

장고 설치가 완료되었다면, 다음과 같이 새 프로젝트를 생성할 수 있다.

```
(venv) $ django-admin startproject PROJECT_NAME
```

`PROJECT_NAME` 은 임의로 원하는 이름을 설정해 주면 되며, 해당 이름으로 폴더가 하나 생길 것이다. 이 폴더가 우리의 프로젝트 폴더이다!  
(여기서부터 장고 어플리케이션을 개발하는 것은 평소처럼 진행하시면 됩니다. 가상환경 설정은 일반 장고 어플리케이션 구축 때도 웬만해선 빠지지 않고 다루어 지는 내용이기도 합니다.)

**의존성(dependency) 명시하기**

(이 섹션부터는 기존에 생성되어 있던 프로젝트에도 해당되는 사항입니다!)

다음으로는, 해당 프로젝트가 어떤 패키지를 필요로 하는지(어떤 패키지가 사용되었는지 - 장고도 포함됩니다) 명시할 필요가 있다. CI 나 EC2 인스턴스, Elastic Beanstalk 는 명시된 의존성(dependency) 만 찾아서 설치하기 때문이다.

다음 명령어로 "의존성 파일" 인 `requirements.txt` 파일을 생성해주자.

```
(venv) $ cd PROJECT_NAME
(venv) $ pip freeze>requirements.txt
```

여기까지 완료하였으면 기본 프로젝트 폴더 구성은 완료되었다! 다음과 같이 프로젝트 폴더의 디렉토리 구성이 되어있으면 성공이다! (기타 등등 파일들은 기본적으로 장고 프로젝트를 생성하면 기본적으로 나오는 파일들이다!)

```
PROJECT_NAME
├── db.sqlite3
├── manage.py
├── requirements.txt
└── PROJECT_NAME
    ├── __init__.py
    ├── settings.py
    ├── urls.py
    └── wsgi.py
```

### CircleCI 빌드 구성하기

CircleCI 빌드를 구성하기 위해선, 일단 아이디가 있어야 할 것이다. CircleCI 는 `github`, `bitbucket`, `google` 로그인을 지원하므로, 셋 중 아무거나 사용하여 로그인을 해 주면 될 것이다.

로그인을 완료한 후, 왼쪽 상단의 `builds` 라는 탭을 클릭하면, 왼쪽 하단에 "Add projects" 라는 버튼이 보이는데, 클릭해주자!  
클릭하면 나의 원격 저장소 레포지토리들이 전부 보인다. 그 중에서 우리가 빌드를 구성할 레포지토리 이름 옆에, "Setup project" 버튼을 클릭하면, 가이드라인이 나온다.

다음으로, CI를 구성해보자. 역시나 이번에도 [공식 문서](https://circleci.com/docs/2.0/language-python/) 를 참조하여 설정을 해보도록 하자.

설정 파일은, `.circleci` 라는 이름의 숨김 폴더(유닉스 계열 운영체제에서는 디렉토리 앞에 .(점) 이 붙으면 숨김 폴더이다) 안에, `config.yml` 이라는 파일명으로 존재해야 한다. 디렉토리 구조가 다음과 일치하게 디렉토리와 파일을 구성해주자.

```
PROJECT_NAME
├── .circleci
│   └── config.yml
├── db.sqlite3
├── manage.py
├── requirements.txt
└── PROJECT_NAME
    ├── __init__.py
    ├── settings.py
    ├── urls.py
    └── wsgi.py
```

그 다음, 다음과 같이 `config.yml` 파일을 작성해주자.

```YAML
version: 2
jobs:
  build:
    docker:
      - image: circleci/python:latest

    working_directory: ~/repo

    steps:
      - checkout

      - restore_cache:
          keys:
          - v1-dependencies-{{ checksum "requirements.txt" }}
          - v1-dependencies-

      - run:
          name: install dependencies
          command: |
            python3 -m venv venv
            . venv/bin/activate
            pip install -r requirements.txt

      - save_cache:
          paths:
            - ./venv
          key: v1-dependencies-{{ checksum "requirements.txt" }}
```

무언가 엄청 잘 정돈된 설정 파일이 만들어졌다. 기본적으로 YAML 파일은 들여쓰기를 통하여 구역을 구분하므로, 들여쓰기에 주목하여 읽으면 된다.

- `version` : 현재 CircleCI 의 버전을 의미한다. 가장 최신 버전인 2버전으로 설정하였다.
- `jobs` : CI 가 수행한 일들을 나열한다. 위의 문서에 적혀있는 `build` 가 그 예시다.
- `steps` : 하나의 일에 대하여 그 과정을 명시한다.

위의 파일을 풀어쓰면, "CircleCI 2버전을 사용하고, `build` 라는 `job` 이 현재 있다" 라고 할 수 있다.  
이제 저 `build` 상세를 살펴보자.

CircleCI 는 `docker` 기반의 환경 구성을 지원하며, 또 그렇게 하기를 권장한다! (여러분 도커 좋아요 엄청 좋아요) (도커가 무엇인지 궁금하시다면 [공식 문서](https://www.docker.com/) 를 한번 들여다 보세요!)  
CircleCI 에서 제공하는 도커 공식 파이썬 최신 이미지를 다운받도록 위 파일과 같이 구성해주면, CI 가 빌드 환경을 구축할 때 최신 파이썬 버전을 알아서 내려받아 준다.

`working_directory` 란 그냥 프로젝트 파일이 올라갈 디렉토리 명을 말한다. 있어도 그만 없어도 그만인 설정이니 그냥 기본 값으로 두자.

`steps` 가 상당히 중요한 단락인데, 여기서 실제로 이 프로젝트를 빌드하는 과정을 밟아볼 수 있다!  
맨 먼저 나타나는 `checkout` 은, git 저장소에서(CircleCI 와 연동 시킬 수 있다) 코드를 가져오는 역할을 한다.  
`restore_cache` 와 `save_cache` 는, 이전에 빌드 환경을 구축하면서 설치한 의존성 모듈들이 존재할 경우에, 캐싱을 한 후 가져오는 전략을 사용하는 곳이다. 이는 빌드 속도를 더욱 빠르게 해준다. 기본값으로 두자.  

중간중간의 `run` 명령어들은, 실제로 커맨드라인에서 실행할 터미널 명령어들을 뜻한다. 자세히 살펴보면 `run` 단락에서는, 가상환경을 활성화 시키고(아마 CI에서 기본적으로 지원해주는 듯 하다) 의존성 패키지들을 `pip install -r requirements.txt` 명령어로 일괄 설치해주는 작업을 한다.

이제 우리는 저 `config.yml` 파일이 어떤 식으로 작성되어야 하는지 알 수 있을 것 같다. 만약에 파일 형식이 잘못되었으면 CI 는 바로 "빌드 실패" 를 뿜어냄과 동시에 어떤 설정이 잘못되었는지를 우리에게 화면으로 잘 말해주므로, 그런 부분은 손쉽게 해결이 가능하다.

### 배포 설정하기

빌드 설정을 다 했으니 이제 배포 설정을 해보자! 위의 Elastic Beanstalk 소개 단락에서 한번 언급하였듯이, **로컬 환경에서 커맨드 라인으로 실행하는 배포를 CI 에서 원격으로 실행시켜주면 그게 배포 자동화이다.** 따라서, 우리는 일단 우선 로컬에서 Elastic Beanstalk 에 커맨드 라인 명령어를 사용하여 배포하는 방법을 알아야 한다. 그것부터 알아보자.

커맨드 라인에서 Elastic Beanstalk 배포를 수행하려면 aws에서 제공하는 Elastic Beanstalk CLI 를 설치하여야 하는데, pip 모듈을 사용하여 쉽게 설치가 가능하다. 

```
(venv) $ pip install awsebcli
```

(이 과정은 가상환경이 활성화 되어있는 상태에서 실행하는게 좋다. 글로벌하게 설치할 경우 권한 문제가 생길 가능성이 높다)

설치가 완료되었다면, 다음 명령어로 이 프로젝트 폴더가 Elastic Beanstalk 어플리케이션 폴더임을 알려주자.

```
(venv) $ eb init
```

중간중간에 몇 개의 질문이 나타날 것이다. 

```
Select a default region
1) us-east-1 : US East (N. Virginia)
2) us-west-1 : US West (N. California)
3) us-west-2 : US West (Oregon)
4) eu-west-1 : EU (Ireland)
5) eu-central-1 : EU (Frankfurt)
6) ap-south-1 : Asia Pacific (Mumbai)
7) ap-southeast-1 : Asia Pacific (Singapore)
8) ap-southeast-2 : Asia Pacific (Sydney)
9) ap-northeast-1 : Asia Pacific (Tokyo)
10) ap-northeast-2 : Asia Pacific (Seoul)
11) sa-east-1 : South America (Sao Paulo)
12) cn-north-1 : China (Beijing)
13) us-east-2 : US East (Ohio)
14) ca-central-1 : Canada (Central)
15) eu-west-2 : EU (London)
(default is 3): 10

Select an application to use
1) sample-autodeploy
2) [ Create new Application ]
(default is 2): 1
```

다음과 같이, region 을 설정해주고(여기서는 서울 리전을 선택하였지만), 밑의 **어플리케이션 선택 에서, 위에서 만들어 주었던 어플리케이션을 사용한다고 해야 한다!**

그 외로, AWS CodeCommit 을 사용할 것인지, ssh 접속을 위한 키 페어를 생성할 것인지 등을 물어본다. 입맛에 맞게 알아서 선택하여 주면 된다.

여기까지 완료되었으면, 프로젝트 폴더의 구성을 보자. `.elasticbeanstalk` 라는 폴더명으로 숨김 폴더가 하나 만들어졌고, 해당 폴더를 열어보면 또 `config.yml` 이라는 파일이 만들어져 있다. 이는 Elastic Beanstalk 설정 파일이다. `eb init` 명령어로 설정을 구성하여 주면 알아서 자동으로 생성해준다.

다음 해야 할 일은, 전에 만들어 두었던 어플리케이션의 환경을 명시하여 주는 일이다.

```
$ eb list
* SampleAutodeploy-env
$ eb use SampleAutodeploy-env
```

`eb list` 명령어를 실행하면, 지금의 어플리케이션에 어떤 환경이 구성되어 있는지를 볼 수 있다. 아마 위에서 처음 생성한 환경 이름이 뜰 것이다. **이 환경을 기본 환경으로 사용하겠다** 라는 뜻으로, `eb use ENV_NAME` 명령어를 실행시켜 주는 것이다.

다음, `.ebextensions` 폴더를 생성 후, 그 안에 `django.config` 파일을 생성하여 다음을 입력해주자.

```
option_settings:
  aws:elasticbeanstalk:application:environment:
    DJANGO_SETTINGS_MODULE: PROJECT_NAME.settings
  aws:elasticbeanstalk:container:python:
    WSGIPath: PROJECT_NAME/wsgi.py
```

이 설정 파일은 Elastic Beanstalk 가 WSGI 스크립트를 실행할 때 `wsgi.py` 의 경로를 지정해주고, `DJANGO_SETTINGS_MODULE` 환경변수를 설정해준다. (장고 해보신 분들은 익숙하실 겁니다)

다음으로, `.gitignore` 파일을 열어서, 맨 마지막 줄에 다음을 추가해주자.

```
!.elasticbeanstalk/config.yml
```

이 줄을 추가해 주어야지만 `config.yml` 파일이 무시가 안되어, Elastic Beanstalk 가 `config.yml` 파일을 인지할 수 있다.

이제 기나긴 설정은 거의 다 했다! 여기서 `eb deploy` 명령어만 입력해 주면 배포가 되지만, 우리는 CI 에게 이 작업을 물리도록 하자.  
CircleCI 의 `config.yml` 에서 우리는 새로운 `deploy` 라는 일을 명시하여 줄 것이다. `config.yml` 파일에 다음을 추가해주자.

```YAML
deploy:
    docker:
      - image: circleci/python:latest

    steps:
      - checkout

      - restore_cache:
          keys:
          - v1-dependencies-{{ checksum "requirements.txt" }}
          - v1-dependencies-

      - run:
          name: install dependencies
          command: |
            python3 -m venv venv
            . venv/bin/activate
            pip install -r requirements.txt

      - save_cache:
          paths:
            - ./venv
          key: v1-dependencies-{{ checksum "requirements.txt" }}

      - run:
          name: install aws client and elasticbeanstalk client
          command: |
            . venv/bin/activate
            pip install awsebcli --upgrade
            bash ./setup-eb.sh
      - run:
          name: deploy
          command: |
            . venv/bin/activate
            eb deploy
```

배포 과정의 설정은 빌드 설정과 별 다를게 없다만, 마지막에 `eb deploy` 라는 명령어로 배포를 수행한다는 점이 큰 차이라고 보면 될 것이다.  
다음, `setup-eb.sh` 라는 이름의 쉘 스크립트를 작성해주자.

```shell
#!/usr/bin/env bash
set -x
set -e

mkdir ~/.aws
touch ~/.aws/config
chmod 600 ~/.aws/config
echo "[profile eb-cli]" > ~/.aws/config
echo "aws_access_key_id=$AWS_ACCESS_KEY_ID" >> ~/.aws/config
echo "aws_secret_access_key=$AWS_SECRET_ACCESS_KEY" >> ~/.aws/config
```

리눅스 명령어들이 보인다.  
로컬에서, `HOME/.aws/config` 의 경로에 파일을 만들어서, aws 계정의 키를 보관하는 과정을 CI 에서도 수행하도록 해 주는 쉘 스크립트이다.  

잠시만, 스크립트를 자세히 살펴보면, `$AWS_ACCESS_KEY_ID`, `$AWS_SECRET_ACCESS_KEY` 라는 환경변수들이 눈에 띄는데, CI상에선 이를 어디서 설정할 수 있을까?  
좌측의 `projects` 탭을 클릭하면, 우리가 추가한 프로젝트 원격 저장소 레포지토리 이름이 보이는데, 맨 오른쪽에 `settings` 라는 이름으로 톱니바퀴가 하나 있다. 클릭해주자.  
좌측의 `permissions` 라는 카테고리의 `AWS Permissions` 로 들어가면, 다음과 같이 ID와 비밀 키를 지정할 수 있는 공간이 나오는데, 여기에 입력해 준 값을 CI는 `$AWS_ACCESS_KEY_ID`, `$AWS_SECRET_ACCESS_KEY` 라는 환경변수 값으로 읽어오는 것이다!  

![](/img/set-aws-key.png)

마지막으로, CI 에게 `workflow` 라는 것을 제시해주어야 하는데, 설정 파일을 보면서 이해하는 것이 더 빠를 수 있으니 우선 파일을 보자. 다음과 같이 `.circleci/config.yml` 파일의 맨 밑에 추가해주자.

```YAML
workflows:
  version: 2
  build-deploy:
    jobs:
      - build
      - deploy:
          requires:
            - build
          filters:
            branches:
              only: master
```

`workflow` 란 말 그대로, "작업의 흐름". **CI 에게 "다음과 같은 순서로 작업을 수행해라" 라고 말해주는 것이다**. 여기서도 CI의 버전을 명시하는 것을 볼 수 있고, `build-deploy` 라는 플로우를 명시하는데, **"빌드 후 배포"** 라는 순서를 확실하게 말해주는 과정이다.  
맨 밑의 `filters` 를 보면, `master` 브랜치에 푸시되었을 때만 배포를 실행한다 라고 명시하는 것을 볼 수 있는데, 이는 매우 중요하다! 이유야 말할 것도 없다. 각기 다른 브랜치의 변경 사항을 모아서 `master` 브랜치에 최종 푸시가 되는 것이기 때문에 배포는 오직 `master` 브랜치에 커밋이 적용되었을 때만 실행되어야 하는 것이다.

### 배포 실행해보기!

좋다, 이제 기나긴 설정이 끝났고 실제로 배포 과정을 밟아보는 수순만 남았다! 다음 두 가지가 잘 실행되는지 살펴보자.

1. 마스터 브랜치 외에 타 브랜치로 푸시할 때 빌드가 잘 적용이 되는지
2. 마스터 브랜치로 합쳤을때 빌드 및 배포가 잘 되는지

우선, 다음과 같이 브랜치를 새로 생성하고 빌드를 올려보자.  

```
$ git checkout -b develop
$ git add --all && git commit -m "Initial build & deployment configuration"
$ git push -u origin/develop
```

그리고 CircleCI 페이지로 들어가보면,

![](/img/build-success.png)

짜잔! 이렇게 "빌드가 성공했습니다" 라고 메시지를 띄워준다.

다음으로, `master` 브랜치로 merge 했을 때 빌드 및 배포가 잘 이루어지는지 확인해보자. 우선, 마스터 브랜치로 PR을 날리는 것부터 시작해보자.

![](/img/pr.png)

오... 이렇게 깃허브 에서도 CI 연동이 잘 되었구나 를 느낄 수 있다. 이제 Merge Pull Request 버튼을 눌러보면, CI 가 정상적으로 돌고 배포를 자동으로 실행하고 있는 것을 볼 수 있다!

## 참고자료

이번 포스트에서 사용한 예시를 깃허브 레포지토리 https://github.com/gyukebox/sample-django-autodeploy 에 올려두었습니다. 참고하세요!  
(새로운 장고 어플리케이션을 생성하실 때 이 레포를 그대로 클론하셔도 됩니다)

## (이 글보다 더 좋은) 참고해 볼 만한 글들

쓰다보니 글이 무한정 길어졌는데요, 여러 곳의 공식 문서에 적혀있는 내용을 풀어쓰기가 쉽지가 않았습니다 죄송합니다. (제가 글재주가 없어서)

우선, 이 글은 다음 SlideShare 의 내용에서 상당한 영감을 받아 쓰여졌습니다:  
https://www.slideshare.net/ssuseraaed82/aws-elastic-beanstalk-ci-django

CI 가 무엇인지 더 자세히 알고 싶으시다면 다음 내용을 한 번 읽어보신 후, 구글링을 해보세요:  
https://www.visualstudio.com/ko/learn/what-is-continuous-integration

AWS Elastic Beanstalk 가 무엇인지 자세히 알고 싶으시다면, 공식 사이트의 가이드 만한 자료가 없다고 생각합니다. 시간 내어서 다음의 내용을 읽어보신다면 훨씬 많은 정보를 습득하실 수 있을 것 같습니다:  
https://docs.aws.amazon.com/ko_kr/elasticbeanstalk/latest/dg/Welcome.html

또한, `django` 만의 배포 자동화 방법으로 `fabric` 을 사용하는 방법이 있는데, 이와 관련된 좋은 글을 한 편 소개드립니다:  
https://beomi.github.io/2017/03/20/Deploy-Django-with-Fabric

마지막으로, 긴 글 읽어주셔서 감사합니다!