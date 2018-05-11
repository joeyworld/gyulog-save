+++
title = "Django 2.0 의 주 변경사항 살펴보기!"
date = "2017-12-27"
categories = ["Python", "Django"]
tags = ["Python", "Django"]
+++

![](/img/django.png)

Django 2.0 이 릴리즈 되었습니다! 1.11 버전과 비교해 다른 주요 상황들을 살펴봄과 동시에, 2.0에서만 지원하는 핵심 기능들을 소개합니다!

<!-- more --> 

`Django` 2.0 이 12 월 2 일 자로 정식 출시 되었습니다! ~~그런데 왜 포스팅은 12 월 27 일?~~  
써본 결과, 확실히 1.11 버전 보다 편해진 점이 많은 것 같은데, 이전 버전과 비교함과 동시에 2.0 버전의 핵심 기능을 소개하고자 해당 포스팅을 준비하게 되었습니다.

## 호환되는 Python 버전

Django 2.0 은 파이썬 3.4, 3.5, 3.6 과 호환됩니다(현재 가장 최신 파이썬 버전은 3.6.4 입니다).  
네 그렇습니다. **파이썬 2 의 지원을 django 2.0 에서는 공식적으로 중단**하게 된 것입니다! (아마 1.11 에서 2.0 으로 버전이 뛴 큰 원인 중 하나인 것 같습니다)

또한, 파이썬 3.4 를 지원하는 마지막 버전이라고 합니다. 또한, 2019 년 3 월까지만 파이썬 3.4 를 지원 할 예정이라고 하므로, 파이썬 3.4 버전을 배포하려고 하시는 분들은 1.11 버전에 머물러 게시는 게 좋다고 합니다(2020 년 4 월까지 지원 예정).

구 파이썬 버전에 대한 내용을 [Django 공식 문서](https://docs.djangoproject.com/en/2.0/releases/2.0/#python-compatibility)에서 발췌하였습니다.

> The Django 1.11.x series is the last to support Python 2.7.

> Django 2.0 will be the last release series to support Python 3.4. If you plan a deployment of Python 3.4 beyond the end-of-life for Django 2.0 (April 2019), stick with Django 1.11 LTS (supported until April 2020) instead. Note, however, that the end-of-life for Python 3.4 is March 2019.

(얼른 파이썬 3 으로 넘어오세요!)  
이제부터는 장고 2.0 의 주요 변경사항을 살펴보도록 하겠습니다!

## URL 패턴의 단순화

(아마 거의 모든 분들이 같은 생각이시겠지만) 장고 2.0 을 쓰면서 가장 편해졌다고 느낀 변경사항이 아닐 까 합니다!

```python
from django.conf.urls import url

urlpatterns = [
    url(r'^articles/(?P<year>[0-9]{4})/$', views.year_archive),
    # ...
]
```

위 예시처럼 정규표현식을 사용해야 했던 url 을,

```python
from django.urls import path

urlpatterns = [
    path('articles/<int:year>/', views.year_archive),
    # ...
]
```

다음과 같이 간단하게, 훨씬 쉽게 사용할 수 있습니다!  
변경 사항을 살펴보면,

* 기존 `django.conf.urls` 모듈에 있던 url 관련 함수들은, `django.urls` 로 옮겨졌습니다.
* `django.conf.urls.url()` 대신, `django.urls.path()` 함수를 사용할 수 있습니다!
* `include()` 함수 또한 `django.urls` 로 옮겨졌습니다.

이 정도입니다.  
기존 정규식을 사용한 url 표현을 하지 못하는 것은 아닙니다. `django.urls.re_path()` 함수를 사용하시면 여전히,

```python
re_path(r'^articles/(?P<year>[0-9]{4})/$', views.year_archive),
```

다음과 같이 정규식을 사용하실 수 있습니다.

개인적인 의견) 기존에 정규식을 많이 사용해보신 분들이라면 `re_path` 가 편하실 수 있다고 생각합니다. 하지만, 가독성 부분에서는 새로운 방식인 `path` 를 사용하는 것이 더 낫다고 생각합니다.

## 더욱 친화적으로 변한 모바일 `admin` 사이트

모바일 관리자 페이지가 반응형으로 바뀌었습니다!

|                      아이폰 화면                      |                     안드로이드 화면                     |
| :---------------------------------------------------: | :-----------------------------------------------------: |
| ![](/img/django-admin-mobile-iphone.png) | ![](/img/django-admin-mobile-android.png) |

## 새로 생긴 사항 요약

지금까지는 장고 2.0 에서 새로 생긴 사항들을 살펴보았습니다. 간단하게 요약하자면,

* 파이썬 2.7 버전의 지원이 공식적으로 중단되었다.
* 정규표현식을 사용하지 않는, 간단한 url 매핑이 가능해졌다.
* `admin` 페이지가 반응형으로, 이쁘게 변하였다.

이 정도 되겠습니다.

## 삭제된 사항들

지금부터는 2.0 으로 오면서 삭제된 사항들을 살펴보도록 하겠습니다.

### `bytestring` 에 대한 지원 일부 삭제

파이썬 2 에 대한 지원 중단과 동시에, `bytestring` 에 대한 지원도 일부 삭제되었습니다. (예시: `reverse()` 함수는 `force_text()` 대신에 `str()` 함수를 사용합니다)

파이썬 3 에서의 모든 문자열은 **유니코드** 입니다! (파이썬 2 에서는 바이트) 바이트를 계속 사용하고 싶으시다면, `bytes` 타입을 사용하셔야 할 것입니다.  
이제 `b'some text'`는 , `"b'some_text"` 처럼 읽혀질 것입니다.

### 오라클 11.2 에 대한 지원 중단

오라클 데이터베이스 11.2 버전에 대한 지원이 중단되었습니다. (가장 최신 버전은 12.x)

## Last, but not least...

처음 장고 어플리케이션을 만들고 서버를 실행하면,
![](/img/django-at-start.png)

기존에, It works! 문구가 사라지고, 다음과 같이 예쁜 시작 화면이 나오네요!

## 기타 변경사항

기타 변경사항은 [장고 2.0 공식 릴리즈 문서](https://docs.djangoproject.com/en/2.0/releases/2.0/) 에서 확인하실 수 있습니다.

이번 포스트에서는 간단하게, 장고 2.0 이 출시되면서 기존의 장고와 비교하여 달라진 점을 살펴보았습니다. 개인적인 한줄 요약은, "초보자가 사용하기 편해진 것 같다" 입니다. (URL 매핑 변화가 이걸?)

+) 시일 내에, 장고 입문 시리즈를 연재할 계획입니다. 많이 봐주시면 감사하겠습니다! :D