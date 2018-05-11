---
title: "Django 커스텀 유저 모델 만들기"
date: 2018-05-11T21:39:21+09:00
categories:
- Python
- Django
tags:
- Python
- Django
---

Django 에서는 기본 유저 모델을 제공해 준다. 해당 유저 모델은 관리자 생성, 로그인 등 다양한 방향으로 확장해서 사용할 수 있다.  
하지만, 그 중에서도 가장 빛나는 기능은 바로 "유저 모델 자체" 도 확장할 수 있다는 것이다. 이번 포스트에서는 기존 유저 모델을 확장한 커스텀 유저 모델을 만드는 방법을 소개한다.

## 유저 모델 만들기

장고는 기본적으로 authentication 기능이 포함된 `User` 모델을 제공해 준다. (`django.contrib.auth.models.User`)  
해당 모델을 확장해서 사용할 수도 있고, 아예 새로 만들어서 대체할 수 있다. 후자의 경우, `settings` 에,

```python
AUTH_USER_MODEL = 'users.MyUser'
```

이런 설정을 넣어줘야 한다. [관련 공식문서 링크](https://docs.djangoproject.com/en/2.0/topics/auth/customizing/#substituting-a-custom-user-model)

해당 설정을 추가해주어야 나중에 `python manage.py migrate` 명령어로 마이그레이션을 실행할 때 기본 authentication model 이 아닌 우리의 커스텀 유저 모델이 마이그레이션 된다.

실제로 공식문서 상에서도 기본 `User` 모델을 그대로 가져가는 것 보다는 커스터마이징 하는 것을 추천한다.

커스텀 유저 모델을 만들 때는, 모델 클래스가 `AbstractUser` 을 상속하거나 `AbstractBaseUser` 를 상속해야 한다. 물론 `AbstractBaseUser` 이 더 추상화 레벨이 높다.  
`AbstractBaseUser` 는 단순히 **유저 모델의 기본** 만을 제공해준다. (예시 : 암호화된 비밀번호, 정규화된 이메일) 
대충 이런 식으로 유저 모델을 정의해 준 다음에, 해 줘야 하는 작업이 존재한다:  

```python
class MyUser(AbstractBaseUser):
    uuid = models.UUIDField(
        primary_key=True, 
        unique=True,
        editable=False, 
        default=uuid.uuid4, 
        verbose_name='PK'
    )
    email = models.EmailField(unique=True, verbose_name='이메일')
    name = models.CharField(max_length=20, verbose_name='이름')
    date_joined = models.DateTimeField(auto_now_add=True, verbose_name='가입일')
    is_active = models.BooleanField(default=True, verbose_name='활성화 여부')
    is_admin = models.BooleanField(default=False, verbose_name='관리자 여부')
```

### `USERNAME_FIELD`

해당 커스텀 유저 모델의 unique identifier 이다. 보통 `username` 인 경우가 많지만, 이메일이 될 수도 있고, 혹은 어떤 `str` 타입이든 가능하다. `unique=True` 옵션이 설정되어 있어야 한다.  

```python
USERNAME_FIELD = 'email'
```

위와 같이 필드 이름을 넣어 주기만 하면 된다.

### `REQUIRED_FIELDS`

우리는 기본 authentication model 을 우리의 `MyUser` 모델로 대체하고 있기 때문에, 다음 명령어로 관리자 계정을 만들 때 필요한 필드들을 새로 명시해 주어야 한다.

```
$ python manage.py superuser
```

이 때 `REQUIRED_FIELDS` 를 설정하여 명시해 줄 수 있다.

```python
REQUIRED_FIELDS = ['name']
```

`USERNAME_FIELD` 에 명시된 필드와, 패스워드는 기본적으로 요구하기 때문에, 따로 명시하지 않아도 된다.

### Meta Class

다음에는, 모델의 메타데이터를 다음과 같이 추가해주자.  

```python
class Meta:
    db_table = 'users'
    verbose_name = '유저'
    verbose_name_plural = '유저들'
```

`db_table` 은 해당 모델과 매핑되는 데이터베이스 테이블의 이름을 지정해준다.  
`verbose_name` 은 모델 자체의 이름을 뜻하고, `verbose_name_plural` 은 복수형이다.  
`verbose_name` 이 정의되어 있는 상태에서 `verbose_name_plural` 이 정의되지 않았으면, 자동으로 뒤에 s 하나를 붙여준다.

좋다. 우리의 커스텀 유저 모델은 완성되었다!

## Manager 만들어주기

이제 우리가 해야 할 일은, 모델을 관리하는 클래스인 매니저를 만들어 주는 일이다.

모든 장고 모델은 Manager 를 통하여 QuerySet 을 받는다. (= 데이터베이스에서 쿼리할 때는 무조건 manager 를 통한다는 소리)   
Manager 는 무조건 모든 모델에 `objects` 라는 이름으로 존재한다.

장고에는 기본적으로 유저 모델에 사용할 수 있는 `UserManager` 이라는 매니저 클래스를 제공해 준다. 만약 유저 모델이`username`, `email`, `is_staff`, `is_active`, `is_superuser`, `last_login`, `date_joined` 필드를 정의한다면 그냥 `UserManager` 를 가져다 쓸 수 있다.  
하지만, 그게 아니고 다른 필드들이 존재한다면 `BaseUserManager` 를 상속한 매니저 클래스를 만들어 주어야 한다.

`BaseUserManager` 클래스를 상속받아 만드는 커스텀 매니저는 `create_user`, `create_superuser` 두 개의 메소드를 구현하여야 하며, 형식은 다음과 같아야 한다:  

```python
def create_user(*username_field*, password=None, **other_fields):
    pass

def create_superuser(*username_field*, password, **other_fields):
    pass
```

우리가 위에서 만들어준 유저 모델의 `USERNAME_FIELD` 는 email 이므로, 다음과 같이 만들어 주면 될 것이다.  
`create_user` 이랑 `create_superuser` 둘을 만들어주고, user 자체를 만들어주는 메소드 하나를 더 만들어 주었다.

```python
class MyUserManager(BaseUserManager):
    def _create_user(self, email, password=None, **kwargs):
        if not email:
            raise ValueError('이메일은 필수입니다.')
        user = self.model(email=self.normalize_email(email), **kwargs)
        user.set_password(password)
        user.save(using=self._db)

    def create_user(self, email, password, **kwargs):
        """
        일반 유저 생성
        """
        kwargs.setdefault('is_admin', False)
        return self._create_user(email, password, **kwargs)

    def create_superuser(self, email, password, **kwargs):
        """
        관리자 유저 생성
        """
        kwargs.setdefault('is_admin', True)
        return self._create_user(email, password, **kwargs)
```

그 다음에, "우리는 모델의 매니저로 이걸 쓸 거야" 라는 것을 명시하기 위해서, 유저 모델에:  

```python
objects = MyUserManager()
```

이 라인을 추가해주면 된다.

## 관리자 생성 및 로그인 해보기

기존 장고의 유저 모델은 이메일과 비밀번호만을 필드로 가졌다. 하지만 우리는 유저 모델을 확장하여 `name` 이라는 추가 필드를 가지도록 만들었다.

이제 superuser 를 생성하면,  

```
$ python manage.py createsuperuser
이메일: asdf@asdf.com
이름: adsfasdf
Password:
superuser created successfully
```

이런 식으로 이메일(`USERNAME_FIELD`), 이름(`REQUIRED_FIELDS` 에 명시), Password 를 요구하는 것을 볼 수 있다.  

다음 명령어로 서버를 실행하고,
```
$ python manage.py runserver
``` 
http://127.0.0.1:8000/admin 으로 접속하면 우리가 만든 superuser 로 관리자 로그인이 되는 것 또한 확인할 수 있다!

## 추가 응용 방식

아직 끝나지 않았다! 커스텀 유저 모델은 관리자 생성 뿐만 아니라,  

- 장고 기본 로그인
- `JWT` 및 토큰 인증 방식의 로그인

의 방식으로도 활용될 수 있다. 추후 포스팅에서 로그인 API 만드는 방식을 소개하도록 하겠다.
