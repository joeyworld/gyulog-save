+++
title = "Django 커스텀 미들웨어 만들기 + Rest Framework 를 위한 HTTP Response Formatting"
date = "2018-06-08T19:44:30+09:00"
tags = ["Python", "Django"]
categories = ["Python", "Django", "Djangorestframework"]
+++

# 들어가며

이 포스팅은 다음에 관해서 다룬다:

* 미들웨어란?
* 미들웨어 설정법
* 나만의 미들웨어를 어떻게 작성하는가, 그리고 그 예시

아쉽게도, 이 포스팅에서 사용되는 예제 코드에 대한 파이썬 문법 및 standard library 사용법은 다루지 않는다 (예시: 정규식 사용을 위한 `re` 모듈). 포스팅을 보시다 막히시면 파이썬 공식 문서 혹은 구글링을 통하여 지식을 습득하시길 권장드린다.

또한, 이 포스팅에서 사용하는 예제 코드는 [Rest Framework](http://www.django-rest-framework.org/) 를 활용한다. 따라서, Rest Framework 를 사용하지 않으시는 독자분께서는 "커스텀 미들웨어 작성법" 단락까지만 읽으셔도 무방하다.

# 미들웨어란?

공식 문서를 읽어보자.

> Middleware is a framework of hooks into Django’s request/response processing. It’s a light, low-level “plugin” system for globally altering Django’s input or output.

쉽게 말해서, http 요청 / 응답 처리 중간에서 작동하는 시스템이다.  
장고는 http 요청이 들어오면 **미들웨어를 거쳐서** 해당 URL 에 등록되어 있는 뷰로 연결해주고, http 응답 역시 **미들웨어를 거쳐서** 내보낸다.

이를 사진으로 표현하면 다음과 같다.

![장고의 http request/response 처리 흐름도](/img/django-middleware.png)

장고에서 미들웨어는 http 요청 혹은 응답의 전처리에 사용된다.

# 미들웨어 등록, 설정에 관하여

미들웨어를 등록하는 방법은 간단하다. 설정에서 `MIDDLEWARE` 항목에 추가하고자 하는 미들웨어의 full python path 를 추가해주면 끝이다.  
`django-admin startproject` 명령어로 장고 프로젝트를 생성하면 기본적으로 다음과 같은 미들웨어들이 등록되어 있다.

```python
MIDDLEWARE = [
    'django.middleware.security.SecurityMiddleware',
    'django.contrib.sessions.middleware.SessionMiddleware',
    'django.middleware.common.CommonMiddleware',
    'django.middleware.csrf.CsrfViewMiddleware',
    'django.contrib.auth.middleware.AuthenticationMiddleware',
    'django.contrib.messages.middleware.MessageMiddleware',
    'django.middleware.clickjacking.XFrameOptionsMiddleware',
]
```

## 미들웨어 순서의 의미

미들웨어 등록에서 가장 중요한 것은 바로 **미들웨어를 등록하는 순서** 이다.  
위의 사진을 한번 더 들여다보자. 눈치 빠르신 분들은 눈치 채셨겠지만, 장고 미들웨어 등록 순서가 가지는 의미는 다음과 같다:

* 장고는 http request 가 들어오면 **위에서부터 아래로** 미들웨어를 적용시킨다.
* 장고는 http response 가 나갈 때 **아래서부터 위로** 미들웨어를 적용시킨다.

# 커스텀 미들웨어 작성법

장고에서 커스텀 미들웨어는 아래와 같이, 함수 혹은 클래스로 작성할 수 있다.

## 함수로 작성

미들웨어를 함수로 작성하게 될 경우에는, 팩토리 형식을 사용한다.

미들웨어 팩토리(아래 예시에서는 `my_middleware`) 는 `get_response` 함수를 받는 하나의 함수이며, `middleware` 이라는 내부 함수를 반환하게 된다.  
또한, `my_middleware` 함수 상단에서는, 최초 설정 및 초기화를 진행하게 된다.

`middleware` 이라는 내부 함수에서는, request 를 받아서 최종적으로 response 를 반환한다.  
이 함수는, 중간의 `get_response` 함수를 호출하기 전과 후로 나누어서 생각할 수 있는데,  
view 가 호출되기 전 / view 가 호출되고 난 후 처리할 일들을 나누어서 작성해 주면 된다.

```python
def my_middleware(get_response):
    # 최초 설정 및 초기화

    def middleware(request):
        # 뷰가 호출되기 전에 실행될 코드들

        response = get_response(request)

        # 뷰가 호출된 뒤에 실행될 코드들

        return response

    return middleware
```

## 클래스로 작성

클래스로 미들웨어를 작성하게 되면, 보다 더욱 구조화된 미들웨어를 작성할 수 있게 된다.

클래스 미들웨어는 아래와 같이, 초기 생성자(`__init__` 함수) 와 호출 함수(`__call__` 함수) 두 부분으로 나뉜다.

`__init__` 함수에서는 최초 설정 및 초기화를 하게 되며, `__call__` 함수는 request 를 받아서 response 를 리턴하게 되는 함수이다.  
`__call__` 함수 역시, `get_response` 가 호출되기 전과 후로 나누어서 생각할 수 있다.

```python
class MyMiddleware:
    def __init__(self, get_response):
        self.get_response = get_response
        # 최초 설정 및 초기화

    def __call__(self, request):
        # 뷰가 호출되기 전에 실행될 코드들

        response = self.get_response(request)

        # 뷰가 호출된 뒤에 실행될 코드들

        return response
```

## `get_response` callable

`get_response` 는 장고에서 미들웨어를 호출할 때 넘겨주는 하나의 함수이며, view 일 수도 있고, 혹은 다른 미들웨어 일 수도 있다.

## 미들웨어 훅(Middleware Hook)

클래스 형식으로 미들웨어를 정의하게 되면, http 요청 혹은 응답에 대한 처리를 하게 되는 메소드를 추가 정의할 수 있는데, 이를 미들웨어 훅(Middleware Hook) 이라고 한다.

만약 미들웨어가 http 요청에 대한 미들웨어라면, 다음 두 가지의 훅 중 하나를 구현하면 된다:

```python
def process_request(request)

# 장고가 view 를 호출하기 바로 직전에 불리는 훅이다
# None 이나 HttpResponse 객체를 리턴해야 한다.
# None 을 리턴하면, view 를 호출하고, HttpResponse 객체를 리턴하면,
# 해당 HttpResponse 를 미들웨어로 다시 쏘아 올린다.
def process_view(request, view_func, view_args, view_kwargs)
```

만약 미들웨어가 http 응답에 대한 미들웨어라면, 다음 세 가지의 훅 중 하나를 구현하면 된다:

```python
# view 가 exception 을 발생시키면 호출된다.
def process_exception(request, exception)

# response 가 템플릿을 반환하는 경우에만
def process_template_response(request, response)

def process_response(request, response)
```

# 예시 : Rest Framwork 를 위한 HTTP Response Formatting

지금까지 미들웨어가 무엇인지, 그리고 어떤 식으로 만드는지에 대하여 알아보았으니, 이번에는 직접 만들어보자!

## 개요

이번에 만들어 볼 미들웨어는 "HTTP 응답 형식을 정해주는 미들웨어" 이다.
많은 상용 API 를 살펴보면, http 응답 형태가 매우 일관성 있다. 예시로 Slack 의 API 를 살펴보면,

```json
{
  "ok": true,
  "stuff": "This is good"
}
```

위와 같이 "성공했는지 여부", 그리고 "응답 데이터 혹은 에러 메시지" 와 같은 (혹은 비슷한) 형식의 일관성 있는 응답을 리턴하도록 하는 경우가 많다.

Django 에서 API 응답을 보내줄 때에는 대부분 이런 형태로 코딩을 하는 경우가 많다.

```python
class SampleView(APIView):
    def get(self, request, *args, **kwargs):
        # ...
        return Response({
            'success': True,
            'data': 'sample data'
        })
```

하지만 모든 view 함수에서 일일이 이런 식으로 http 응답 형태를 정하는 것은 매우 귀찮은 일이며, 중복되는 코드가 많이 생기므로 구조적으로도 깔끔하지 못하다.  
따라서, **미들웨어를 활용하여** 우리의 http 응답이 **미들웨어를 거쳐서 자동으로 formatting 되어서 나가게** 하는 방법을 사용해보자!

## 우리가 원하는 결과

에를 들어, http 응답으로 `{"some": "data"}` 같은 JSON 데이터를 보내고자 할 때, 다음과 같은 형식으로 보내질 것이다.

```json
{
  "success": true,
  "result": {
    "some": "data"
  },
  "message": null
}
```

## 미들웨어 클래스 만들기

우리의 미들웨어 클래스 이름을 `ResponseFormattingMiddleware` 이라고 정하고, 다음의 클래스 변수를 만들자.

```python
import re
from rest_framework.status import is_client_error, is_success

class ResponseFormattingMiddleware:
    METHOD = ('GET', 'POST', 'PUT', 'PATCH', 'DELETE')
```

Http 요청 중 `METHOD` 에 포함되어 있는 형식에 대해서만 미들웨어가 작동하도록 하기 위함이다.  
그리고, 다음과 같이 initializer 를 만들어주자.

```python
def __init__(self, get_response):
    self.get_response = get_response
    self.API_URLS = [
        re.compile(r'^(.*)/api'),
        re.compile(r'^api'),
    ]
```

`re` 는 정규식(regex) 계산을 위한 파이썬 모듈이다.  
URL 에 `api` 가 포함되어 있는 요청에 대해서만 미들웨어가 작동하도록 하기 위해서, `API_URLS` 라는 인스턴스 변수를 만들었다.

## `__call__` 메소드 만들기

그 다음 우리가 작성할 메소드는 바로 `__call__` 메소드이다.  
공식 문서를 읽어보자. 이 메소드는 "클래스의 인스턴스가 함수처럼 호출될 때" 불리는 메소드이다.

> object.**call**(self[, args...])  
> Called when the instance is “called” as a function; if this method is defined, x(arg1, arg2, ...) is a shorthand for x.**call**(arg1, arg2, ...).

```python
def __call__(self, request):
    response = self.get_response(request)
    if hasattr(self, 'process_response'):
        response = self.process_response(request, response)
    return response
```

우선 `get_response` 를 호출하여 http 응답을 가져온 뒤,  
`process_response` 가 정의되어 있다면, `process_response` 함수를 호출해 http 응답을 처리하는 형식이다.

## `process_response` 메소드 만들기

우리의 미들웨어는 "http 응답을 처리하는 미들웨어" 이므로, `process_response` 메소드를 정의하여 http 응답을 처리하도록 만들어야 한다.  
눈치 빠르신 분들은 눈치 채셨겠지만, 우리는 여기서 **http 응답을 포맷팅 해주는 로직**을 짤 것이다.

우선, "현재 미들웨어가 작동해야 하는 조건" 인지를 검사해야 한다. 미들웨어가 작동할 조건은 다음과 같다.

* 유효한 URL
* 유효한 http 요청

이는 다음과 같은 코드를 `process_response` 최상단에 삽입하여서 검사할 수 있다.

```python
path = request.path_info.lstrip('/')
valid_urls = (url.match(path) for url in self.API_URLS)

if request.method in self.METHOD and any(valid_urls):
    response_format = {
        'success': is_success(response.status_code),
        'result': {},
        'message': None
    }
```

그 다음은, view 에서 넘어온 response 에 데이터가 있는지를 검사해야 한다.  
만약 있다면, 데이터를 우리가 정한 응답 format 에 맞춰서 재구성해야 하며, 없다면 그냥 response format 그대로 http 응답 데이터를 구성하면 된다.

다음과 같은 코드를 삽입함으로써 실행할 수 있다.

```python
if hasattr(response, 'data') and \
        getattr(response, 'data') is not None:
    data = response.data
    try:
        response_format['message'] = data.pop('message')
    except (KeyError, TypeError):
        response_format.update({
            'result': data
        })
    finally:
        if is_client_error(response.status_code):
            response_format['result'] = None
            response_format['message'] = data
        else:
            response_format['result'] = data

        response.data = response_format
        response.content = response.render().rendered_content
else:
    response.data = response_format
```

그리고, 마지막으로 `response` 를 리턴해 주면 우리의 미들웨어는 완성된다!

## 미들웨어 등록 후 테스트 해보기

우리의 미들웨어 클래스는 다음과 같이 구현되었다.

```python
import re
from rest_framework.status import is_client_error, is_success


class ResponseFormattingMiddleware:
    """
    Rest Framework 을 위한 전용 커스텀 미들웨어에 대해 response format 을 자동으로 세팅
    """
    METHOD = ('GET', 'POST', 'PUT', 'PATCH', 'DELETE')

    def __init__(self, get_response):
        self.get_response = get_response
        self.API_URLS = [
            re.compile(r'^(.*)/api'),
            re.compile(r'^api'),
        ]

    def __call__(self, request):
        response = None
        if not response:
            response = self.get_response(request)
        if hasattr(self, 'process_response'):
            response = self.process_response(request, response)
        return response

    def process_response(self, request, response):
        """
        API_URLS 와 method 가 확인이 되면
        response 로 들어온 data 형식에 맞추어
        response_format 에 넣어준 후 response 반환
        """
        path = request.path_info.lstrip('/')
        valid_urls = (url.match(path) for url in self.API_URLS)

        if request.method in self.METHOD and any(valid_urls):
            response_format = {
                'success': is_success(response.status_code),
                'result': {},
                'message': None
            }

            if hasattr(response, 'data') and \
                    getattr(response, 'data') is not None:
                data = response.data
                try:
                    response_format['message'] = data.pop('message')
                except (KeyError, TypeError):
                    response_format.update({
                        'result': data
                    })
                finally:
                    if is_client_error(response.status_code):
                        response_format['result'] = None
                        response_format['message'] = data
                    else:
                        response_format['result'] = data

                    response.data = response_format
                    response.content = response.render().rendered_content
            else:
                response.data = response_format

        return response
```

그 다음, 설정의 `MIDDLEWARE` 로 가서 우리의 미들웨어를 **맨 밑에다** 추가하자.  
맨 밑에다 추가하는 이유는, **장고의 기본 미들웨어들이 요청들을 처리해 준 후, view 를 호출한 뒤, 바로 http 응답을 포맷팅해서 내보내는 순서를 지켜주기 위함**이다.

```python
MIDDLEWARE = [
    'django.middleware.security.SecurityMiddleware',
    'django.contrib.sessions.middleware.SessionMiddleware',
    'django.middleware.common.CommonMiddleware',
    'django.middleware.csrf.CsrfViewMiddleware',
    'django.contrib.auth.middleware.AuthenticationMiddleware',
    'django.contrib.messages.middleware.MessageMiddleware',
    'django.middleware.clickjacking.XFrameOptionsMiddleware',
    'PYTHONPATH_TO_MIDDLEWARE.ResponseFormattingMiddleware',
]
```

(`PYTHONPATH_TO_MIDDLEWARE` 는 알아서 잘 바꿔주자)

여기까지 성공적으로 완성하였으면, 다음과 같은 view 가 있을 때,

```python
from rest_framework.views import APIView
from rest_framework.permissions import AllowAny

class TestAPIView(APIView):
    permission_classes = [AllowAny]

    def get(self, request, *args, **kwargs):
        return Response({
            'message': 'some',
            'some': 'data'
        })
```

해당 view 로 향하는 http GET 요청을 보낼 시,

```json
{
  "success": true,
  "message": "some",
  "result": {
    "some": "data"
  }
}
```

위와 같이 우리가 정해준 형식의 http 응답이 오는 것을 확인할 수가 있다!

# 마치며

장고를 공부해오면서, 커스텀 미들웨어를 만들어서 http 요청 혹은 응답을 조작할 수 있을 것이라고는 상상을 못했는데, 스터디 때 팀원분께서 이 내용을 친절하게 다루어 주셔서 너무 많은 것을 얻었고, 잊어버리고 싶지 않아서 이렇게 블로그 포스팅으로 정리하게 되었다.  
**역시 개발자는 잘하는 사람들을 만나서 계속 배워야 한다**. 정말이지 배움은 끝이 없는 것 같다.

장고와 Rest Framework 를 사용해서 개발하는 누군가가 이 포스팅을 통해서, 커스텀 미들웨어로 Http 응답을 형식화 할 수 있다는 점을 알아간다면 더할 나위 없을 것 같다.

앞으로도 장고에 관한 기능들을 꾸준히 포스팅 하고 싶고, 블로그 포스팅으로나마 간접적으로 내가 배운 지식을 다른 누군가에게 공유하고 싶다.
