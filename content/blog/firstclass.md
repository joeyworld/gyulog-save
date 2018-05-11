+++
title = "파이썬으로 알아보는 일급 객체(first-class citizen)"
categories = ["Python", "Functional Programming"]
tags = ["Python", "함수형 프로그래밍"]
date = "2017-09-30"
+++

![](/img/python.png)

함수형 프로그래밍(functional programming) 을 공부하다보면, 가장 많이 나오는 말이 "일급 객체", 혹은 "일급 시민" 일 겁니다. 일급 객체가 무엇인지 파이썬의 함수를 예시로 들어 알아보는 포스팅입니다.

<!-- more -->

일급 객체, 일급 함수라는 개념은 영국의 크리스토퍼 스트레이치라는 컴퓨터 과학자가 1960 년에 처음 소개한 개념이다. 일급 객체가 되려면 다음과 같은 조건을 만족해야 한다.

* 변수나 데이터 구조 안에 담을 수 있다.
* 매개변수로 전달이 가능하다.
* 리턴값으로 사용될 수 있다.

## 파이썬의 함수는 일급 객체이다

해당 조건을 보면, **파이썬의 함수는 일급 객체이다** 라는 결론을 쉽게 도출할 수 있다. 왜냐하면 파이썬의 함수는 파라메터로 넘길 수 있고, 리턴값으로 사용될 수 있기 때문이다.

### 변수나 데이터 구조 안에 담을 수 있다

파이썬의 함수는 변수에 할당할 수 있다. 다음 `greeting.py` 코드를 보면서 눈으로 확인해보자!

```python
def hello():
    print('Hello friend!')


def bye():
    print('See you later!')


greeting = hello
greeting()
print(greeting)
print()

greeting = bye
greeting()
print(greeting)
print()

print(type(greeting))
```

greeting 은 function 타입의 변수이고, 처음에는 hello 였다가 나중에는 bye 로 바뀐 것을 볼 수 있을 것이다. 실행 결과와 greeting 의 타입을 확인해 보면 다음과 같다.

```
$ python greeting.py
Hello friend!
<function hello at 0x101863c80>

See you later!
<function bye at 0x101863d08>

<class 'function'>
```

### 매개변수로 전달이 가능하다

또한, 파이썬의 함수는 매개변수로 전달이 가능하다. 다음의 `addition.py` 코드를 보면서 눈으로 알아보자.

```python
def add_two(a, b):
    return a + b


def calculate(func, arg1, arg2):
    print('calculation:', func.__name__)
    print('result:', func(arg1, arg2))

calculate(add_two, 4, 10)
```

`calculate` 함수에 `add_two` 함수를 넘긴 것을 볼 수 있다!  
위의 코드를 실행하면 당연하게도, 다음과 같은 결과가 나온다.

```
$ python addition.py
calculation: add_two
result: 14
```

이런 표현법의 장점은, 기능을 수정하고 싶을 때에도 기존 코드를 전혀 수정하지 않고 기능 수정이 가능하는 점이다. 예를 들어, `calculate` 으로 두 수를 곱하는 연산을 행하고 싶다고 하면, `calculate` 함수를 전혀 수정하지 않은 채로, 두 수를 곱하는 함수 `multiply_two` 를 새로 정의한 후 `calculate` 함수에 인자로 넘겨주면 그만이다.

### 리턴값으로 사용될 수 있다.

마지막으로, 파이썬의 함수는 리턴값으로 사용될 수 있다. 이것도 역시 눈으로 보면 이해가 쉽다.

```python
def document_it(func):
    def new_function(*args, **kwargs):
        print('Running function:', func.__name__)
        print('Positional arguments:', args)
        print('Keyword arguments:', kwargs)
        result = func(*args, **kwargs)
        print('Result:', result)
        return result
    return new_function


def add_ints(a, b):
    return a + b

# add_ints(3, 5) = 8
decorated_add_ints = document_it(add_ints)
print(decorated_add_ints(3, 5))
```

(어쩌다 보니 예제에서 일급객체의 세 가지 요소가 다 드러났다)  
`add_ints` 함수에 간단한 설명을 붙이려고 `document_it` 함수를 정의하였다. `document_it` 함수는 내부함수인 `new_function` 을 리턴하는 구조이다.  
맨 밑의 두 줄도 유심히 살펴보자. `document_it` 함수에 `add_ints` 함수를 인자로 넘긴 후에, `decorated_add_ints` 에 저장하였다! 실행 결과를 살펴보면 다음과 같다.

```
$ python decorator.py
Running function: add_ints
Positional arguments: (3, 5)
Keyword arguments: {}
Result: 8
8
```

(이 예제는 파이썬의 또 다른 문법인 데코레이터(`decorator`) 입니다. 데코레이터에 대해서는 추후 포스팅으로 찾아뵙겠습니다.)

## 추가 - 함수가 일급 객체인 언어와 그렇지 않은 언어

이제 "일급 객체" 란 무엇이고, 파이썬의 함수가 왜 일급 객체인지 알 수 있게 되었다. 그렇다면 다른 언어는 어떨까?

* 함수가 일급 객체인 언어 : Javascript, Scala, Go
* 함수가 일급 객체가 아님 : C, Java

(위의 언어가 전부가 아닙니다! 실제로 함수형 프로그래밍 언어는 많이 존재합니다)
