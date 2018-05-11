+++
title = "TypeScript(0) - 설치와 설정"
tags = ["TypeScript", "JavaScript"]
categories = ["TypeScript"]
date = "2018-01-13 15:15:19"
+++


최근에 TypeScript 를 들여다 보면서 깨달은 내용을 적어봅니다. 첫번째는 역시나 설치와 설정이네요.

![](/img/typescript.png)

<!-- more -->

## TypeScript 란?

> JavaScript that scales.

> TypeScript is a language for application-scale JavaScript. TypeScript adds optional types, classes, and modules to JavaScript. TypeScript supports tools for large-scale JavaScript applications for any browser, for any host, on any OS. TypeScript compiles to readable, standards-based JavaScript. Try it out at the playground, and stay up to date via our blog and Twitter account.

사실 타입스크립트는 자바스크립트다(?) 정확하게는, "자바스크립트의 확장판" 이라고 생각하면 쉽다. 컴파일하여 자바스크립트가 되는 compile-to-javascript 언어이며, ES6을 지원하는 것은 물론이고, 이에 확장하여 타입 시스템, generics, decorators 등을 지원해주는 언어이다.  
타입 시스템을 도입하면 자바스크립트의 dynamic 한 장점을 없애버리는 것이 아닌가 생각했지만, 선택적으로 사용할 수 있고, 무엇보다 다수의 ES 버전으로 컴파일할 수 있다는 면에서 상당한 매력을 느꼈다.

## TypeScript 설치하기

npm을 이용하여 설치할 수 있다. 다음과 같이 글로벌하게 설치할 수 있지만,
```
$ npm install -g typescript
```

TS가 필요없는 프로젝트가 있을수 있으므로, 그냥 프로젝트 생성 후 로컬로 설치하는 것이 더 좋아 보인다.
```
$ npm init
$ npm install typescript --save-dev
```

그 다음, ESLint 처럼 TypeScript 코드를 바로잡아 줄 수 있는 TSLint 도 설치해주면 좋다.  

만약 yarn 을 사용할 경우,
```
$ yarn global add typescript tslint
$ yarn add typescript tslint --dev
```
(첫번째 줄은 글로벌, 두번째 줄은 로컬)  
위의 명령어를 사용하면 설치할 수 있다.

## TypeScript 기본 설정하기

설치를 완료하였다면 기본 설정 몇 가지가 남았는데, 타입스크립트 컴파일러 설정과 TSLint 설정이다.  
다음 두 명령어를 이용하면 공들이지 않고 설정의 기본을 잡을 수 있다.

```
$ ./node_modules/.bin/tsc --init
$ ./node_modules/.bin/tslint --init
```

일일이 디렉토리를 입력하는 것이 귀찮다면, [npx](https://github.com/zkat/npx)를 도입하는 방법도 나쁘지 않다. npx 는 로컬로 설치한 패키지를 글로벌 패키지처럼 사용할 수 있게 해주는 패키지이다. Webpack 같은 툴 스크립트 쓸 때 사용하면 좋을 것 같기도 하다.

다음과 같이 npx를 글로벌하게 설치해준 다음 명령어로 똑같은 작업을 실행할 수 있다.
```
$ npm install -g npx
$ npx tsc --init
$ npx tslint --init
```

## Hello World!

(이 문단은 공식 사이트의 [TypeScript in 5 minutes](https://www.typescriptlang.org/docs/handbook/typescript-in-5-minutes.html) 를 의역한 문단입니다.)
이제 설정도 다 했으니 Hello world 를 작성해보자! `greeter.ts` 파일을 편집기에서 열고 다음과 같이 코드를 작성해보자.

```javascript
function greeter(person) {
  return "Hello " + person;
}

var user = "Jane User";
document.body.innerHTML = greeter(user);
```

위의 코드는 그냥 평범한 자바스크립트 형식의 코드이다. 타입스크립트의 타입 시스템을 도입하여, 다음과 같이 타입을 붙여 보자.

```typescript
function greeter(person: string): string {
  return "Hello " + person;
}

let user: string = "Jane User";
document.body.innerHTML = greeter(user);
```

이제 컴파일러를 사용해보자. 

```
$ tsc greeter.ts
```

그러면 이제 `greeter.js` 라는 파일이 생성된 것을 확인할 수 있다!

### Interface 와 Class

예전의 `greeter.ts` 를 타입스크립트의 특성인 인터페이스와 클래스를 이용하여 확장하여 보자. (추후 포스팅에서 다룰 내용입니다)  
`greeter.ts` 를 다음과 같이 수정하여 보자.

```typescript
class Student {
  fullName: string;
  constructor(public firstName: string, public middleInitial: string, 
    public lastName: string) {
    this.fullName = `${firstName} ${middleInitial} ${lastName}`;
  }
}

interface Person {
  firstName: string;
  lastName: string;
}

function greeter(person: Person) {
  return `Hello, ${person.firstName} ${person.lastName}`;
}

let user = new Student("Jane", "M.", "User");
document.body.innerHTML = greeter(user);
```

이제 HTML 코드를 작성하여 우리가 컴파일하여 만들 스크립트를 실행시켜 보자. 다음 파일을 `greeter.html` 이라는 이름으로 작성하고,

```html
<!DOCTYPE html>
<html>
    <head><title>TypeScript Greeter</title></head>
    <body>
        <script src="greeter.js"></script>
    </body>
</html>
```

브라우저에서 열면 처음으로 타입스크립트를 컴파일하여 만든 자바스크립트가 실행되는 것을 볼 수 있다!

## 더 알아보면 좋은 내용들

Microsoft 에서는 TypeScript로 새로운 프로젝트를 시작하는 방법 및 기존 프로젝트에서 TypeScript 로 마이그레이션(migration, 넘어가는 것. 여기서는 JS에서 TS로 변경하는 것이 되겠네요) 에 대한 가이드를 충분히 제공한다.  
이런 가이드들은 [공식 사이트](https://www.typescriptlang.org/samples/index.html) 에서 확인이 가능하다.

공식 문서 또한, [공식 사이트](https://www.typescriptlang.org/docs/home.html) 에서 확인할 수 있다.