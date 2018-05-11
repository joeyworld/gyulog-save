+++
title = "Python 함수의 docstring 과 decorator"
categories = ["Python"]
tags = ["Python"]
date = "2017-10-03"
+++

![](/img/python.png)

파이썬의 함수는 엄청 강력합니다. 파이썬의 함수의 강력함을 배가시켜주는 두 요소, docstring과 decorator에 대한 소개입니다.

<!-- more -->

이번 포스팅은, 파이썬의 함수 중 함수를 서식하는 기능을 하는 `docstring`과 `decorator`에 관한 내용이다. 특히 데코레이터 같은 경우에는 처음 접하는 경우에는 "저게 뭐지?" 하고 느낄 수 있지만, 잘 사용하면 매우 강력한 도구가 되는 만큼 잘 알고 가는 것이 좋다.  
(이번 포스팅에서는 함수에 적용되는 데코레이터만 다룹니다. 클래스를 데코레이터로 만들 수도 있어요, 하지만 여기서는 다루지 않고 오로지 함수에만! 집중해 보겠습니다.)

## Docstring

함수가 어떤 일을 하는지에 대한 설명은 보통 코드 내에서는 주석으로 쓰고, 외부에서는 wiki 페이지를 이용한다던가 해서 문서화시켜서 정리하곤 한다. 하지만, 파이썬에서는, 함수에 대한 설명을 함수 내에 넣을 수 있는 기능이 있는데, 그게 바로 `docstring` 이다.

일단 코드를 살펴보자

```python
def piglatin(word):
    """
    :param word: word that would be changed into piglatin
    :return: piglatin version of word
    """
    return word[1:] + word[0] + 'ay'
```

눈치챘겠지만, docstring 은 document string 을 줄인 말이다. 위의 코드와 같이 함수 시작 부분에 큰따옴표 3 개를 연달아 붙임으로써 docstring 을 정의할 수 있다.

함수의 docstring 을 출력하고 싶으면 다음과 같이 하면 된다.

```python
>>> piglatin('word')
'ordway'
>>> help(piglatin)
Help on function piglatin in module __main__:

piglatin(word)
    :param word: word that would be changed into piglatin
    :return: piglatin version of word

>>> print(piglatin.__doc__)

    :param word: word that would be changed into piglatin
    :return: piglatin version of word
```

`help` 함수를 활용하면 함수의 docstring 을 출력할 수 있다. 위와 같이 서식이 갖추어져서 나온다.  
또한, 서식 없이 docstring 을 있는 그대로 보고 싶으면, 함수의 `__doc__` 필드를 출력하면 된다.

### 함수에 필드가 있다?

사실 docstring 은 간단한 주석이 아니라, 함수의 여러가지 필드(변수) 중 `__doc__` 변수에 들어간다 (`__doc__` 필드는 docstring 을 위하여 만들어졌다). 함수에도 여러가지 변수가 있는데(위의 `__doc__` 과 함수 이름이 저장되는 `__name__` 이 대표적이다), 왜냐하면 함수도 하나의 클래스이기 때문이다!

```python
>>> type(piglatin)
<class 'function'>
```

## Decorator

데코레이터, 이름을 딱 보고 무언가를 꾸며주는 것이구나 라고 생각했다면 그게 바로 정답이다! 데코레이터는 기존의 코드에 여러가지 추가 기능을 적용시키는 파이썬의 한 문법인데, **주로 함수의 형태로 많이 쓰인다**.

데코레이터를 보다 잘 알기 위해서는 [일급 객체](http://www.gyuveloper.com/post?6)가 무엇인지, 그리고 파이썬의 함수가 왜 일급 객체인지를 이해하고 있어야 한다. 만약 일급 객체라는 단어가 생소하거나, 파이썬 함수가 왜 일급 객체인지 아리송하다면 해당 내용을 먼저 보고 오는 것을 추천드린다.

가장 좋은 방법은 역시 눈으로 보고 이해하는 것. 다음 코드를 함께 살펴보자.  
(파일 이름을 `decorator.py` 라고 하겠다)

### 데코레이터 함수의 일반적인 형식, 그리고 특징

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
```

(`document_it` 함수를 잘 살펴보면, **매개변수도 함수이고, 반환값도 함수**인 것을 눈치챌 수 있는데, **파이썬의 함수는 일급 객체임을 단편적으로 보여주는 예시가 된다**.)

`document_it` 과 같은 형태를 가지는 함수를 **데코레이터 함수**라고 부르는데, 데코레이터 함수의 특징을 살펴보면,

* 함수를 매개변수로 받는다(위의 예제에서는 `func`). 해당 함수는 데코레이터가 적용되는 대상이 된다.
* 데코레이터 함수 내에서 내부 함수가 정의되고, 정의한 내부 함수를 리턴한다. 데코레이터 함수에 인자로 넘어온 함수를, **내부 함수가 꾸며주게 된다**.

한 문장으로 위의 예시를 들어서 요약하면, **`func`을 `new_function`이 꾸며준다!**

위의 `document_it`의 경우에는 어떻게 될까? `new_function` 을 살펴보면 함수에 대한 정보들을 출력해주고, 함수를 실행한 후 결과를 리턴하는데, 아마도 `func` 에 대한 정보를 출력해준 다음 `func` 을 실행한 결과를 리턴하게 되겠구나 라는 것을 예상할 수 있다.

### 수동으로 데코레이터 붙이기

데코레이터를 적용시켜보면,

```python
def add_ints(a, b):
    return a + b

# add_ints(3, 5) = 8
decorated_add_ints = document_it(add_ints)
print(decorated_add_ints(3, 5))
```

위와 같이 두 정수를 더하는 `add_ints` 에 `document_it` 데코레이터 함수를 붙여서, `decorated_add_ints` 변수에 저장하였다(함수를 변수에 저장할 수 있는건 파이썬에서 함수는 일급 객체니까!). `decoreated_add_ints` 에 3 과 5 를 넣어서 실행해보면, `add_ints` 에 대한 정보가 쭉 나오고, 결괏값인 8 이 출력될 것임을 예상할 수 있는데,  
아니나 다를까, 실행 결과는,

```
$ python decorator.py
Running function: add_ints
Positional arguments: (3, 5)
Keyword arguments: {}
Result: 8
8
```

만약 그냥 `add_ints` 만 호출했다면 그냥 8 이 리턴되었겠지만, `document_it` 데코레이터 함수를 붙여서 실행하면 위와 같이 **추가로 기능이 붙어서 실행된다**.

아마 눈치 빠른 사람들은 알아챘겠지만, **추가 기능을 붙이기 위해서 원래 함수를 수정하지 않아도 된다는 점** 이 데코레이터의 제일 큰 장점 중 하나이다! 현실적인 예시를 들어보자. 함수를 디버깅하기 위해서 함수 내에 전달된 인자의 변화를 관찰하고 싶을 때, `print` 문을 추가하면 쉽게 볼 수 있지만, 원래 함수를 수정해야 하는 불편함이 있다. 이 때, `print` 문으로 함수의 인자를 출력하게 해주는 데코레이터 함수를 작성하여 붙인다면, 원함수의 수정 없이 더욱 편리하게 디버깅을 할 수 있다.

### 보다 편리한 데코레이터 사용법

위의 예제에서는 수동으로 데코레이터를 붙이는 방법을 살펴보았는데, 이번에는 보다 편리하게 데코레이터를 함수에 사용해보자. `decorator.py`의 `add_ints` 와 그 밑의 부분을 다음과 같이 수정해보자.

```python
@document_it
def add_ints(a, b):
    return a + b

print(add_ints(5, 4))
```

수동으로 데코레이터 함수를 붙이는 부분이 없어진 대신에, `add_ints` 위에 한 줄이 추가되었다.
그렇다, `@decorator_function` 을 사용하면, 함수에 데코레이터 함수를 붙일 수 있다. 위의 예제에서는 `add_ints` 함수에 `document_it` 데코레이터 함수가 자동으로 붙게 된다. 따라서, `add_ints` 함수를 그냥 실행해도,

```
$ python decorator.py
Running function: add_ints
Positional arguments: (5, 4)
Keyword arguments: {}
Result: 9
9
```

위와 같이 데코레이터가 적용된 모습을 확인할 수 있다!

### 두 개 이상의 데코레이터

만약 `decorator_py`에서 `add_ints`의 결과를 제곱하는 기능을 추가하고 싶다고 하면 어떻게 될까? 물론 `add_ints`를 수정할 수도 있겠지만, 데코레이터를 이용하여 수정 없이 기능을 추가하여 보자.

`decorator_py` 에 다음과 같은 수정을 가해보자.

```python
def square_it(func):
    def new_function(*args, **kwargs):
        result = func(*args, **kwargs)
        return result * result
    return new_function

@document_it
@square_it
def add_ints(a, b):
    return a, b
```

한 번 봤으니 두 번은 어렵지 않다. `square_result` 는 데코레이터 함수이고, `add_ints` 에 두 개의 데코레이터를 붙인 경우이다.

```
$ python decorator.py
Running function: add_ints
Positional arguments: (5, 4)
Keyword arguments: {}
Result: 81
81
```

어, 실행 결과는 예상했던 것과는 살짝 다르다. `add_ints` 의 실행 결과는 9 지만, 이를 제곱하여 출력하고 싶어서 `square_it` 데코레이터를 붙였는데, `add_ints` 의 결과 자체가 81 이 되어버렸다! 이유는 바로 **데코레이터의 실행 순서** 에 있다. 함수에 2 개 이상의 데코레이터가 붙은 경우, **함수 바로 위에 붙은 데코레이터부터(def 바로 위) 역순으로 실행된다**. 위의 경우에는, `square_it` 데코레이터가 먼저 실행된 후, `document_it` 데코레이터가 실행되었으므로, `add_ints` 의 결과 자체가 81 로 바뀌어버린 것이다.

그렇다면, 두 데코레이터의 실행순서를 바꿔주면...

```python
@square_it
@document_it
def add_ints(a, b):
    return a, b
```

```
$ python decorator.py
Running function: add_ints
Positional arguments: (5, 4)
Keyword arguments: {}
Result: 9
81
```

우리가 원했던 그림이 나왔다!

## 요약

이번 포스팅의 내용을 간추려보면,

* `docstring` 은 함수 내부에 설명을 추가한 문자열이고, 함수의 `__doc__` 변수에 저장된다.
* Docstring 에 관한 컨벤션 - [PEP 257](https://www.python.org/dev/peps/pep-0257/)
* `decorator` 은 함수에 붙어서, 붙여진 함수를 꾸며주는 역할을 한다.
* 데코레이터 함수의 기본 형식은,

```python
def decorator_function(original_function):
    def wrapper_function(*args):
        # YOUR_CODE_HERE
    return wrapper_function
```

이다. `original_function`에 `wrapper_function` 의 기능을 덧붙여준다.

* 함수에 두 개 이상의 데코레이터가 붙은 경우, 함수에서 가까운 데코레이터부터 차례대로 실행된다.
* Decorator 에 관한 컨벤션 - [PEP 318](https://www.python.org/dev/peps/pep-0318/)

이제 `docstring`을 이용하여 함수에 쉽게 주석을 달고, `decorator`을 이용하여 함수에 추가 기능들을 덧붙여보자!
