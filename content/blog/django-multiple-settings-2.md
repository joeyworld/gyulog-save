+++
title = "Django 설정 더욱 세부적으로 분리하기"
tags = ["Python", "Django"]
categories = ["Python", "Django"]
date = "2018-05-30T01:11:16+09:00"
+++

# 들어가기에 앞서

이전 포스팅에서, Django 설정을 분기하는 방법에 대해서 포스팅한 적이 있습니다. [해당 포스팅 링크](https://gyukebox.github.io/blog/django-%EC%97%90%EC%84%9C-%EC%97%AC%EB%9F%AC-%EA%B0%80%EC%A7%80%EC%9D%98-%EC%84%A4%EC%A0%95%EC%9D%84-%ED%95%A0-%EC%88%98-%EC%9E%88%EB%8A%94-%EB%B0%A9%EB%B2%95/)  
하지만, 설정의 카테고리별로 완벽하게 분리하는 데에는 한계가 있었습니다.  
이번 포스팅에서는 설정을 카테고리별로, 더욱 세부적으로 분리하는 방법에 대해서 다룹니다.

장고의 설정은 형식 안에서, 어떤 식으로 분리하든 깔끔하기만 하면 상관이 없기에, 이 포스팅은 그냥 참고용으로만 보시고, 본인에게 맞는 설정방법으로 설정하시면 됩니다 :)

# 기존 단순 설정 분리 방식과의 비교

이전 포스팅에서 다룬 설정 분리는, 환경에 따른 설정 분리만 하였다. 그 결과로, 설정 관련 파일의 디렉토리 구조는 다음과 같았다.

```
settings
├── __init__.py
├── base.py
├── dev.py
└── prod.py
```

그렇다. `base.py` 에 기본 설정들을 넣고, 개발 환경에서의 설정은 `dev.py` 에, 그리고 프로덕션(실 서버) 환경에서의 설정은 `prod.py` 에 하는 방식이었다.

하지만, 이 방법은 기본 설정이 무지하게 커질 수 있다는 한계가 있다. 간단한 크기의 장고 앱을 만든다면 문제가 되지 않지만, 복잡한 앱을 만들기 위해서 third-party 앱을 많이 쓰면, 그만큼 우리가 해 줘야 하는 설정도 늘어날 것이고, 설정 파일의 크기 또한 커지기 때문이다.

따라서, 설정을 카테고리 별로 세부적으로 분리하면 더욱 깔끔하고, 관리도 잘 된다.  
이번 포스팅에서 소개할 방법은, 설정을 다음과 같은 구조로 관리할 수 있게 해준다.

```
settings
├── __init__.py
├── components
│   ├── app.py
│   ├── database.py
│   ├── locale.py
│   ├── middleware.py
│   ├── security.py
│   ├── static.py
│   ├── template.py
│   └── url.py
└── environments
    ├── dev.py
    └── prod.py
```

`components` 디렉토리 밑의 파일에는 설정의 카테고리 별로 각각의 파일에 설정을 하고, `environments` 디렉토리 밑에는 환경 관련한 설정을 각각의 파일에 한다.  
마지막으로, `__init__.py` 파일에서는 이렇게 분리한 설정을 통합하는 스크립트를 짠다.

# 사용할 라이브러리 설치

이렇게 설정을 세부적으로 분리하기 위해서 우리가 사용할 라이브러리는 바로 [django-split-settings](https://github.com/sobolevn/django-split-settings) 이다. 다음 명령어로 설치하자.

```
(venv) $ pip install django-split-settings
```

(장고 뿐만 아니라, 파이썬 프로젝트를 만드실 때는 가상환경을 사용하시는게 좋습니다.)

또한, 프로젝트의 requirements 에도 명시하자.

```
(venv) $ pip freeze>requirements.txt
```

# 설정 분리하기

처음 장고 프로젝트를 생성하게 되면, `settings.py` 라는 기본 설정 파일이 자동으로 생성된다.  
해당 파일이 있는 디렉토리에, `settings` 라는 디렉토리를 새로 만들고, `settings` 디렉토리 밑에 `components` 디렉토리와 `environments` 디렉토리를 새로 만들자.  
다음 명령어를 사용해도 된다.

```
$ mkdir -p settings settings/compoments settings/environments
```

## 카테고리별 설정 분리

## app.py

`components` 디렉토리에 `app.py` 파일을 만들고, 기본 설정 파일에서 `INSTALLED_APPS` 항목을 복사하자.  
그 다음에, 다음과 같이 세 개의 파이썬 리스트를 새로 만들자.

```python
DJANGO_APPS = []
THIRD_PARTY_APPS = []
PROJECT_APPS = []
```

`DJANGO_APPS` 리스트에는 장고 기본 앱들(프로젝트 생성 시 자동으로 추가되는 앱들) 을 넣으면 된다. 대부분 다음과 같다.

```python
DJANGO_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
]
```

`THIRD_PARTY_APPS` 에는 따로 설치하는 (`pip` 를 통해) 라이브러리들을 넣으면 된다. `rest_framework` 등이 그 예시다.

`PROJECT_APPS` 에는 `python manage.py startapp` 명령어로 생성한 앱들을 넣으면 된다.

마지막으로, 파일의 맨 아래에 다음과 같이 `INSTALLED_APPS` 리스트를 만들자.

```python
INSTALLED_APPS = DJANGO_APPS + THIRD_PARTY_APPS + PROJECT_APPS
```

해석하면, "이 프로젝트에 설치된 앱들은, 기본 장고 앱과, 외부 라이브러리와, 우리가 프로젝트에서 설정해 준 앱이 있다" 는 뜻이다!

## database.py

`components` 디렉토리에 `database.py` 파일을 만들고, 다음과 같이 작성하자.

```python
DATABASE_LIST = {
    'dev': {
        'default': {
            # 개발 환경 데이터베이스 설정
        }
    },
    'prod': {
        'default': {
            # 프로덕션 환경 데이터베이스 설정
        }
    }
}

DATABASES = DATABASE_LIST[ENV]
```

위와 같이 작성하면, 현재 이 프로젝트가 처한 환경에 따라 데이터베이스 설정을 자동으로 가져오게 된다.  
(민감한 username, password 등의 개인 정보는 알아서 환경변수로 잘 대체해주자)

오류가 뜨더라도 놀라지 마시라. `ENV` 이라는 변수를 아직 안 만들었기 때문이다.  
나중에 통합 스크립트에서 설정할 예정이므로 에러는 무시하고 넘어가자.

## locale.py

`components` 디렉토리에 `locale.py` 파일을 만들자.  
이 파일에는 시간대에 관한 설정을 할 것이다. 다음과 같이 작성하자.

```python
LANGUAGE_CODE = 'ko-kr'

TIME_ZONE = 'Asia/Seoul'

USE_I18N = True

USE_L10N = True

USE_TZ = True
```

기본 언어를 한국어로 설정함과 동시에 시간대를 한국의 시간대와 똑같이 만들어주었다.

## middleware.py

`components` 디렉토리에 `middleware.py` 파일을 만들고, 기본 설정 파일에서 `MIDDLEWARE` 항목을 복사해서 붙여넣기하자.  
이 파일에는 미들웨어들을 추가한다. 앞으로 커스텀 미들웨어, 혹은 써드 파티 라이브러리에서 만들어진 미들웨어를 추가할 때도 이 파일에 추가하면 된다.

## security.py

`components` 디렉토리에 `security.py` 파일을 만들고, `AUTH_PASSWORD_VALIDATORS` 항목을 기본 설정 파일에서 복사해서 붙여넣기하자.  
이 파일에는 보안 관련 설정을 추가한다. 앞으로 프로젝트를 크게 만들면서 보안 관련 설정을 할 일이 있을 때마다 이 파일에 설정하면 될 것이다.

## static.py

`components` 디렉토리에 `static.py` 파일을 만들고, `STATIC_URL` 항목을 기본 설정 파일에서 복사해서 붙여넣기하자.  
이 파일에는 정적 파일 관련 설정을 추가한다. (정적 파일이란 CSS, 자바스크립트, 그리고 이미지 파일을 일컫는다.) 정적 파일 관련 추가 설정을 할 때마다 (예를 들면 `STATIC_ROOT` 이라던가) 이 파일에 하면 된다.

## template.py

`components` 디렉토리에 `template.py` 파일을 만들고, `TEMPLATE` 항목을 기본 설정 파일에서 복사해서 붙여넣기하자.  
이 파일에는 장고 템플릿 관련 설정을 추가한다. 프로젝트를 만들면서 장고 템플릿을 쓸 일이 없다면 추가하지 않아도 된다.

## url.py

마지막으로, `components` 디렉토리에 `url.py` 파일을 만들고, 다음과 같이 작성해주자.

```python
ROOT_URLCONF = '<PROJECT_NAME>.urls'

WSGI_APPLICATION = '<PROJECT_NAME>.wsgi.application'
```

(`PROJECT_NAME` 은 알아서 잘 바꿔주자)  
이 파일에는 url 관련 설정을 추가한다.

여기까지 완료하였으면 카테고리 별 기본 설정은 끝이다.  
추가 설정이 필요할 시 카테고리별로 분류하여, 기존의 파일에 설정을 추가해도 되고, 아니면 새로운 파일을 `components` 에 만들어서 설정해도 된다.  
(예시 : `django rest framework` 설정을 `components/rest-framework.py` 파일을 만들어서 추가)

## 환경별 설정 분리

환경별로 설정을 분리해야 하는 항목은 `DEBUG`, 그리고 `ALLOWED_HOSTS` 이 둘이 있다.  
(데이터베이스 설정도 여기서 해도 되지만 여기서는 그렇게 하지 않았다. 어떤 방식을 사용해도 상관없다.)

먼저, `environmens` 디렉토리에 `dev.py` 파일을 만들고, 개발 환경에서의 설정을 하자.

개발 환경이므로 당연히 디버깅 모드를 켤 것이고, 모든 호스트를 허용할 것이다. 다음과 같이 작성하자.

```python
DEBUG = True
ALLOWED_HOSTS = ['*']
```

다음, `environmens` 디렉토리에 `prod.py` 파일을 만들고, 프로덕션 환경에서의 설정을 하자.
디버깅 모드는 당연히 꺼야 하고, 허용할 호스트도 선별해야 한다. 다음과 같이 작성하고, `ALLOWED_HOSTS` 는 배포하는 서버의 도메인에 맞게 설정하자.

```python
DEBUG = False
ALLOWED_HOSTS = []
```

여기까지 완료하였으면 환경별 설정까지 끝이다!

# 통합 스크립트 짜기

설정을 카테고리별로, 그리고 환경별로 분리하니 나름 깔끔하고 보기 좋은 설정이 나왔다. 하지만!
장고 프로젝트를 처음 만들었을 때 설정은 통합되어 나오지 않았던가. **결국에는 이 설정들을 통합해 주어야 한다.**

설정 통합을 위한 스크립트를 짜기 위해, `settings` 디렉토리에 `__init__.py` 파일을 만들자.

## `BASE_DIR`, 그리고 `SECRET_KEY`

`__init__.py` 파일 안에 다음과 같이 작성하자.

```python
import os

BASE_DIR = os.path.dirname(
    os.path.dirname(
        os.path.dirname(
            os.path.abspath(__file__)
        )
    )
)

SECRET_KEY = os.environ['PROJECT_SECRET_KEY']
```

`BASE_DIR` 은 해당 프로젝트의 기본 경로인데, 우리는 이 경로를 프로젝트의 루트로 설정해 줄 필요가 있다.  
현재 설정 디렉토리는 `PROJECT_NAME/PROJECT_NAME/settings/__init__.py` 이므로,  
"기본 경로는 해당 파일의 디렉토리의 부모 디렉토리의 부모 디렉토리의 부모 디렉토리다" 라고 설정해준다.

그리고, 프로젝트의 `SECRET_KEY` 를 감추기 위해, 다음과 같이 환경변수로 대체한다.

## "현재 속해있는 환경" 환경변수 설정하기

그리고, `__init__.py` 에 다음 줄을 추가해주자.

```python
ENV = os.environ.get('PROJECT_ENV', 'dev')
```

`os.environ.get('ENV_NAME', 'default')` 가 의미하는 것은, "환경변수 중에서 `ENV_NAME` 이 가지는 값을 읽어오거나, 해당 환경변수가 없으면 그 값을 `default` 로 설정하라" 이다.  
따라서, 현재 환경을 `stage` 나 `prod` 등의 환경으로 설정하지 않는 이상, 이 값은 `dev`, 즉 개발 환경이 되도록 만든다.

그리고, `components/database.py` 의 맨 위에 다음을 추가해주자.

```python
from PROJECT_NAME.settings import BASE_DIR, ENV
```

`__init__.py` 에서 설정해준 `BASE_DIR` 과 `ENV` 를 임포트 해주면, 전에 나타났던 에러가 말끔히 사라질 것이다!

## 분리된 설정 끌어모으기

이제 분리된 설정을 끌어모으는 일만 남았다. `__init__.py` 에 다음을 추가해주자.

```python
from split_settings.tools import include

COMPONENTS_DIR = os.path.join(
    BASE_DIR,
    'PROJECT_NAME',
    'settings',
    'components'
)

COMPONENTS = [
    'components/{}'.format(os.path.basename(component))
    for component in glob.glob(os.path.join(COMPONENTS_DIR, '*.py'))
]

ENVIRONMENTS = ['environments/{}.py'.format(ENV)]

SETTINGS = COMPONENTS + ENVIRONMENTS

include(*SETTINGS)
```

이전에 설정을 `components` 랑 `environments` 로 분리하였으므로, 분리한 파일들을 전부 끌어 모으는 코드이다.  
`COMPONENTS` 리스트에,

```python
COMPONENTS = [
    'components/app.py'
    'components/database.py'
    # ...
]
```

다음과 같이 일일이 다 쓰는 것은 당장 귀찮기도 하고, 추후에 설정이 추가될 때 갱신해야 하므로 좋지 않다.  
따라서 파이썬의 기본 모듈인 `glob` 모듈과 리스트 컴프리헨션(List Comprehension) 을 이용해 자동으로 파일을 긁어 모으자.

환경 관련 파일은 현재 환경에 따라 하나만 선택해야 하므로 위와 같이 `ENV` 변수에 따라서 선택하도록 설정한다.

마지막으로, "설정은 카테고리별 설정과 환경에 대한 설정이 있다" 를 나타내주고, `django-split-settings` 라이브러리를 사용하여 합쳐주면 통합 스크립트는 끝이다!

# 실행해보기

좋다. 이제 잘 실행되나 확인해보자. 다음 명령어로 실행해보자.

```
(venv) $ python manage.py runserver

Performing system checks...

System check identified no issues (0 silenced).
May 30, 2018 - 16:04:26
Django version 2.0, using settings 'festival_api.settings'
Starting development server at http://127.0.0.1:8000/
Quit the server with CONTROL-C.
```

아래와 같이 뜨고, URL 로 들어갔을 때 페이지가 정상적으로 뜨면 성공이다!

# 마치며

필자는 "쪼갤 수 있으면 최대한 쪼개자" 는 개발 마인드를 가지고 있어서, 기존의 장고 설정 분리 방법에 아쉬움을 느끼고, 구글링을 하다가 이런 방법을 알아내고, 적용해보니 만족감을 얻을 수 있었다.  
하지만, 도입부에도 언급하였듯이, 해당 설정법이 안 맞는 사람들도 있을 것이다. 따라서 이 포스트의 내용을 **신뢰하지 말고 능동적으로 수용**해주시길 부탁드린다.

기회가 될 때마다 장고의 숨겨진 기능들을 꾸준히 포스팅 해보고 싶다.
