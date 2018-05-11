+++
title = "Django 에서 여러 가지의 설정을 할 수 있는 방법"
tags = ["Python", "Django"]
categories = ["Python", "Django"]
date = "2018-03-05 01:35:53"
+++


때로는, 장고 어플리케이션에서 세팅 파일 하나로는 모든 설정을 다 할 수 없는 경우가 존재합니다. 이런 경우에, 여러 개의 설정 파일을 만들어서 다중 설정을 할 수 있는 방법을 소개합니다!

![](/img/django.png)

<!-- more -->

## Intro: 처음 django 프로젝트를 만들면...

처음 django 프로젝트를 생성할 때 다음과 같은 명령어로 생성하는 것이 대부분이다. (PyCharm 같은 IDE를 사용한다 한들 결과는 똑같거나 비슷하다.)

```
$ django-admin startproject PROJECT_NAME
```

해당 명령어로 프로젝트를 만든 후 `PROJECT_NAME` 이라는 이름의 프로젝트 폴더로 들어가면,

```
PROJECT_NAME/
├── PROJECT_NAME
│   ├── __init__.py
│   ├── settings.py
│   ├── urls.py
│   └── wsgi.py
└── manage.py
```

다음과 같은 디렉토리 구조를 보여준다.  
이 프로젝트의 설정은 당연하게도, `PROJECT_NAME/settings.py` 파일에서 이루어질 테니, 해당 파일을 살펴보자.

```
BASE_DIR
SECRET_KEY 
DEBUG
ALLOWED_HOSTS
INSTALLED_APPS
MIDDLEWARE
ROOT_URLCONF
TEMPLATES
WSGI_APPLICATION
DATABASES
AUTH_PASSWORD_VALIDATORS 
LANGUAGE_CODE
TIME_ZONE 
USE_I18N 
USE_L10N
USE_TZ 
STATIC_URL
```

설정 파일의 항목들만 나열해보았다. 

보아하니, 하나의 설정 파일에서 여러 환경의 설정을 모두 포함하기는 힘들어 보이는 항목들이 여러 개 보인다. 대표적으로, `DEBUG` 이랑 `DATABASES` 항목이 그렇다.  
우리는 로컬 개발 환경에서 `DEBUG` 옵션을 `TRUE` 로 설정해 에러가 뜨는 원인을 찾고 싶지만, 실제 서비스 환경에서는 traceback 을 절대로 보여주면 안 될 것이다. 또한, 로컬 데이터베이스 환경이랑 실 서비스의 데이터베이스 환경이 같을 리가 없다!

이 포스팅에서는, 설정 파일을 여러개로 분리하여 다중 설정을 하는 방법을 소개한다.

## 세팅 파일 여러개로 분리하기

현재 설정 파일은 `settings.py` 하나의 파일만이 존재하는데, 우리는 이것을 다음과 같은 구조를 가지도록 쪼갤 것이다.

![](/img/settings-structure.png)

`settings` 라는 폴더 안에, `base.py` 라는 기본 설정 파일이 존재하고, 로컬 개발 환경 / 테스트 환경 / 실제 서비스 환경 에 따라 각각 `dev.py`, `test.py`, `prod.py` 라는 이름으로 설정 파일을 만들어주면 된다.  
눈치 빠르신 분들은 눈치 채셨겠지만, `dev.py`, `test.py`, `prod.py` 파일은 모두 `base.py` 를 import 하는 구조를 가지게 될 것이다.

우선, 기존의 `settings.py` 의 이름을 `base.py` 로 변경하자.  
그 다음, `settings` 라는 디렉토리를 새로 만들어준 후, `base.py` 를 `settings` 디렉토리로 옮기고, `dev.py`, `test.py`, `prod.py` 라는 이름으로 세 개의 설정 파일을 새로 만들어주자.

따라서, 우리의 프로젝트 폴더의 구조는 다음과 같이 변경된다.

```
PROJECT_NAME/
├── __init__.py
├── settings
│   ├── base.py
│   ├── dev.py
│   ├── prod.py
│   └── test.py
├── urls.py
└── wsgi.py
```

## 서로 다른 설정 해주기

파일을 만들었으니 이제는 파일에 설정을 넣어줘야 되지 않겠는가!  
하지만, 세부 설정을 하기 전에, `base.py` 에서 바꿔야 할 내용이 몇 가지 있다.

### `BASE_DIR` 재정의하기

`base.py` 의 맨 윗부분에는 다음과 같이 `BASE_DIR` 이 정의되어 있다.

```python
import os
BASE_DIR = os.path.dirname(os.path.dirname(os.path.abspath(__file__)))
```

하지만, 우리는 설정 파일의 위치를 `settings` 라는 디렉토리 내부에 넣어두었으므로, 한 번 더 `os.path.dirname` 으로 파일의 절대경로를 감싸주어 `BASE_DIR` 이 프로젝트 폴더를 향하게 다시 만들어 주어야 한다.  
따라서, 위의 `BASE_DIR` 을 다음과 같이 변경해주자.

```python
import os
BASE_DIR = os.path.dirname(os.path.dirname(os.path.dirname(os.path.abspath(__file__))))
```

#### `os.path.dirname` 과 `os.path.abspath`

- `os.path.dirname` 은 인자로 들어온 파일이 속해있는 디렉토리명을 리턴한다.
- `os.path.abspath` 은 인자로 들어온 파일의 절대경로를 리턴한다.

따라서, 예를 들어, 파일이 `/foo/bar/baz/foo.txt` 라면,
`os.path.dirname(os.path.abspath(__file__))` 은, `/foo/bar/baz` 이 되는 것이다!

자세한 내용은 [파이썬 공식문서의 os 모듈 파트](https://docs.python.org/3/library/os.html#module-os) 를 참조하시길 바란다.

### `SECRET_KEY` 숨기기

또 다른 중요한 하나는, 바로 `SECRET_KEY` 를 공개되지 않게 숨기는 일이다. `base.py` 파일의 `BASE_DIR` 항목 밑을 살펴보면,

```python
SECRET_KEY = 'foobarbaz'
```

이런 형태로 `SECRET_KEY` 가 제공되어 있을텐데, 나 말고 이 비밀 키를 알게 하고 싶은 사람은 전혀 없을 것이다!  
이 `SECRET_KEY` 는 환경변수로 대체하는 방법을 사용하여 숨기도록 하자.

```python
SECRET_KEY = os.environ['APP_SECRET_KEY']
```

`APP_SECRET_KEY` 라는 이름의 환경변수로 실제 키 값을 등록하고, 위와 같이 환경변수로부터 읽어오게끔 설정하면, 키 값을 숨길 수 있다.

### 로컬 환경 설정 - `dev.py` 파일 건드리기

여기까지 왔으면 기본 환경 설정은 어느 정도 마무리되었다. 지금부터는 로컬 개발환경 설정을 시작해보자. `dev.py` 파일을 열어 맨 처음에 다음 줄을 추가한다.

```python
from PROJECT_NAME.settings.base import *
```

(`PROJECT_NAME` 은 알아서 잘 바꿔주자)  
이 한 줄은 바로 기본 설정인 `base.py` 의 모든 내용들을 import 해준다. 로컬 환경과 실 서버 환경의 설정이 달라야 하는 경우, 이 파일에서 재정의 해 주면 되는 것이다.  

로컬 환경 설정에서 가장 중요한 두 가지는 역시,

1. `DEBUG` 를 `True` 로 설정
2. 로컬 데이터베이스 환경설정

일 테니, `dev.py` 파일에서 위의 항목들을 수행해주도록 하자.

### 원격 환경 설정 - `prod.py` 파일 건드리기

이제, 실제 서버 환경설정을 해보자. 로컬 환경설정과 똑같이 하되, `DEBUG`를 `False` 로 설정해줘야 한다!  
그 다음, `ALLOWED_HOSTS` 를 지정해 주어야 한다. Django 에서 debug mode 를 끌 경우, `ALLOWED_HOSTS` 를 지정하도록 되어 있기 때문이다.

위 두 가지를 잘 해결하였으면, 실제 서버 데이터베이스 환경설정까지 끝마쳐주자.

## 어떤 세팅으로 서버를 실행할까?

이제, 설정도 다 했겠다, 서버를 실행해보는 일만 남았다! 하지만, 설정이 여러 가진데, 어떤 설정을 적용해서 서버를 실행시켜야 할까?

### 서버를 실행하기 전에

서버를 실행하기 전에, 먼저 해야 할 작업들이 몇 가지 있다.  
프로젝트 폴더랑 같은 레벨에서 `manage.py` 라는 파일을 열어보자.

```python
if __name__ == "__main__":
    os.environ.setdefault("DJANGO_SETTINGS_MODULE", "PROJECT_NAME.settings")
    # ...
```

아마 이런 코드가 보일 것이다.  
`DJANGO_SETTINGS_MODULE` 환경변수를 설정해 주는 코드인데, 생각해 보니 우리는 `settings.py` 라는 파일이 없다! 따라서, 다음 줄을,

```python
os.environ.setdefault("DJANGO_SETTINGS_MODULE", "PROJECT_NAME.settings.base")
```

다음과 같이, "기본 설정은 `base` 입니다" 라고 알게 하도록 바꿔야 한다!

### 로컬 서버 시작 시 환경을 바꿔주는 방법

여기까지 완료하였으면, 로컬 서버를 시작하여 보자.

```
$ python manage.py runserver
```

웰컴 페이지가 잘 뜨는 것을 확인하였다면, 터미널에 뜬 문구를 주목하자.

```
$ python manage.py runserver
Performing system checks...

System check identified no issues (0 silenced).
March 04, 2018 - 16:16:34
Django version 2.0.2, using settings 'PROJECT_NAME.settings.base'
Starting development server at http://127.0.0.1:8000/
Quit the server with CONTROL-C.
```

중간에, 어떤 설정으로 서버가 구동되었는지를 확인할 수 있다! 방금은 기본 설정인 `base` 설정을 적용한 것을 볼 수 있다.  

로컬 서버를 실행할 때 설정을 바꾸어 줄 수 있는데,  

```
$ python manage.py runserver --settings=PROJECT_NAME.settings.dev
```

이런 식으로 `--settings` 옵션을 주면 가능하다!  
위의 예시는 `dev` 설정으로 로컬 서버를 실행 시킨 것을 의미한다.

### 배포 시에는 어떻게 하나요?

여기까지 무사히 진행되었다면 개발할 때는 아무 문제가 없지만, 가장 중요한 것은, 배포 시에 `prod` 설정을 지정해 주어야 한다는 것이다!

파이썬은 WSGI 를 통하여 웹 서버를 실행시키므로, `wsgi.py` 파일을 찾아서 설정하여 주면 된다.  
`wsgi.py` 파일을 들여다보면,

```python
os.environ.setdefault("DJANGO_SETTINGS_MODULE","django_sample_settings.settings")
```

역시나 이런, `DJANGO_SETTINGS_MODULE` 환경변수를 설정해주는 코드가 있는데, 

```python
os.environ.setdefault("DJANGO_SETTINGS_MODULE", "django_sample_settings.settings.prod")
```

위와 같이 `prod` 설정을 환경변수의 값으로 지정하여 주면, 실 서버에서 `prod` 설정을 활용할 수 있게 된다!
