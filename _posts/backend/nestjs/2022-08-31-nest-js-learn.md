---
title: "[NestJS] NestJS로 배우는 백엔드 프로그래밍"
last_modified_at: 2022-08-31T23:50:00+09:00
categories:
    - Back End
    - NestJS
tags:
    - Back End
    - NestJS
    - Node.js
toc: true
toc_sticky: true
toc_label: "목차"
---

NestJS : NestJS 를 이용한 백엔드 프로그래밍 관련하여 학습한 내용을 간단하게 정리해보자.
{: .notice--info}

# NestJS 소개

- Node.js에 기반을 둔 웹 API 프레임워크
- Express or Fastify 프레임워크를 래핑하여 동작
- DB, ORM, Configuration, 유효성 검사 등 수많은 기능 제공
- 라이브러리 설치에 용이한 Node.js의 확장성을 그대로 가지고 있음
- Angular 로 부터 영향을 많이 받음
- 모듈/컴포넌트 기반으로 프로그램을 작성하여 재사용성이 높음
- IoC, DI, AOP 와 같은 객체지향 개념을 도입
- 타입스크립트 사용
- 모두싸인, 당근마켓 등 여러 회사에서 사용

## Express vs NestJS
  - Express: 자유도가 높음.(그만큼 발품 필요)
  - NestJS: 이미 잘짜여진 틀에서 문서참고를 통해 활용이 가능

## 설치 및 실행

```bash

$ npm i -g @nestjs/cli # NestJS 설치
$ nest new project-name # 프로젝트 생성

$ npm install
$ npm run start

$ npm run start:dev # 개발 서버(--watch 옵션 : 소스코드 변경 감지하여 코드 저장할 때 마다 서버를 다시 구동
```

<br/>

# Node.js

## 장점

- 단일 쓰레드 non-blocking I/O 이벤트 기반 비동기 방식
- 서버 자원에 크게 부하 X
- 코드 작성에 유리

## 단점
- 컴파일러 언어의 처리속도에 비해 성능이 떨어짐 -> 서버 성능이 꾸준히 발전하고 V8(자바스크립트 처리 엔진) 성능이 계속 향상 중
- 콜백 지옥(가독성 떨어짐) -> Promise 도입(ES6)되면서 간결한 표현으로 작성 가능, async/await 기능 추가(ECMAScript 2017)되면서 비동기를 동기처리하는 것처럼 코드로 작성 가능

## 이벤트 루프

단일 쓰레드인 Node.js가 어떻게 비동기 처리를 할 수 있는지 이벤트 루프의 원리를 통해 알아보자.

- 6개의 단계(Phase)
  - 각 단계마다 콜백 함수를 담기 위한 큐를 가짐
  - idle & prepare 단계를 제외한 단계에서든 자바스크립트 코드 실행 가능
- `node main.js` 명령어로 Node.js 애플리케이션을 콘솔에 실행하면 Node.js는 먼저 이벤트 루푸를 생성한 다음 메인 모듈인 main.js를 실행
  - 이 과정에서 생성된 콜백들이 각 단계에 존재하는 큐에 들어감
  - 메인 모듈의 실행을 완료한 다음 이벤트 루프를 계속 실행할 것인지 결정
  - 만약 큐가 모두 비어서 더이상 수행할 작업이 없으면 Node.js가 루프를 빠져나오고 프로세스 종료

![image](https://user-images.githubusercontent.com/53864640/187722594-ee0a030f-4125-4503-af97-d035bb5f2054.png){: .align-center}

### Timer 단계
  
- 이벤트 루프의 시작
- `setTimeout, setInterval`과 같은 함수를 통해 만들어진 타이머들을 큐에 넣고 실행
- 최소 힙(Min Heap, 최솟값을 찾기 위해 완전 이진 트리를 사용하는 자료 구조)으로 관리 됨. -> 실행할 시각이 가장 적게 남은 타이머가 힙의 루트가 됨. Timer 단계에서 최소 힙에 들어 있는 타이머들을 순차적으로 찾아 실행한 후 힙을 재구성함
- 시간이 지난 타이머들의 콜백은 실행한도(Hard Limit)에 도달하면 다음 단게로 넘어감

### Pending (i/o) 콜백 단계

- 현재 돌고 있는 루프 이전의 작업에서 큐에 들어온 콜백

### Idle, Prepare 단계

- Idle 단계는 매 틱(Tick, 매 단계가 이동하는 것을 의미) 마다 실행 됨.
- Prepare 단계는 매 폴링마다 그 전에 실행 됨
- 이 두 단계 Node.js의 내부 동작을 위한 것

### Poll 단계
- 이벤트 루프 중 가장 중요한 단계
- Poll 단계에서는 새로운 I/O 이벤트를 가져와서 관련 콜백을 수행
- watch_queue를 가지고 있음
- watch_queue가 비어 있지 않다면 큐가 비거나 시스템 실행 한도를 다다를 때까지 동기적으로 모든 콜백을 실행
- 만약 큐가 비게되면 Node.js는 곧바로 다음 단계로 이동하지 않고 check_queue, pending_queue, closing_callbacks_queue에 남은 작업이 있는지 검사한 다음 작업이 있으면 다음 단계로 이동
- 만약 큐가 모두 비어서 해야할 작업이 없다면 잠시 대기
  - 이때 대기시간은 타이머의 최소 힙의 첫번째 타이머를 꺼내 지금 실행할 수 있는 상태라면 그 시간만큼 대기한 후 다음 단계로 이동
  - 이유는 바로 타이머 단계로 넘어간다 해도 어차피 첫번째 타이머를 수행할 시간이 되지 않았기 때문에 이벤트 루프를 한 번 더 돌아야 하므로 Poll 단계에서 시간을 보내는 것

### Check 단계

  - `setImmediate`의 콜백만을 위한 단계
  - 큐가 비거나 시스템 실행 한도에 도달할 때 까지 콜백 수행

### Close 콜백 단계

- `socket.on('close', () => {})`과 같은 close나 destory 이벤트 타입의 콜백이 여기서 처리
- 이벤트 루프는 Close 콜백 단계를 마치고 나면 다음 루프에서 처리해야 할 작업이 남아있는지 검사
- 작업이 남아 있다면 Timer 단계부터 한 번 더 루프를 돌게 되고 아니라면 루프를 종료

### nextTickQueue & microTaskQueue

- nextTickQueue는 `process.nextTick()` API의 콜백들을 가지고 있음
- microTaskQueue는 Resolver된 Promise의 콜백을 가지고 있음
- 이 둘은 이벤트 루프의 일부가 아님
- 즉, libuv 라이브러리에 포함된 것이 아니라 Node.js에 포함된 기술
- 이 두 큐에 들어 있는 콜백은 단계를 넘어가는 과정에서 먼저 실행 됨
- 우선순위: nextTickQueue > microTaskQueue

<br/>

# 타입스크립트

- 마이크로소프트에서 개발
- 런타임 에러가 발생할 가능성이 있는 코드를 정적 분석
- `tsc` 명령으로 컴파일하여 자바스크립트로 코드 변환 가능

<br/>

# 데코레이터

## 데코레이터 합성

- `f(g(x))`와 같이 동작
- 위에서 아래로 평가(evaluate) 됨
- 결과는 아래에서 위로 함수 호출(call) 됨

```javascript
@f
@g
test
```

```javascript
function first() {
  console.log("first(): factory evaluated");
  return function (target: any, propertyKey: string, descriptor: PropertyDescriptor) {
    console.log("first(): called");
  };
}

function second() {
  console.log("second(): factory evaluated");
  return function (target: any, propertyKey: string, descriptor: PropertyDescriptor) {
    console.log("second(): called");
  };
}

class ExampleClass {
  @first()
  @second()
  method() {
    console.log('method is called');
  }
}
```

```text
출력 결과: 
first(): factory evaluated
second(): factory evaluated
second(): called
first(): called
method is called
```

## 클래스 데코레이터

- 클래스의 생성자에 적용되어 클래스 정의를 읽거나 수정할 수 있음
- 선언 파일(타입스크립트 소스코드를 컴파일 할 때 생성되는 파일로 타입시스템의 타입추론을 돕는 코드가 포함되어 있다. 소스파일의 이름은 d.ts로 끝난다.)과 선언 클래스 내에서는 사용할 수 없음

```javascript
function reportableClassDecorator<T extends { new (...args: any[]): {} }>(constructor: T) {
  return class extends constructor { // 클래스 데코레이터는 생성자를 리턴하는 함수여야 한다.
    reportingURL = "http://www.example.com"; // 클래스 데코레이터가 적용되는 클래스에 새로운 `reportingURL`이라는 속성을 추가한다.
  };
}

@reportableClassDecorator
class BugReport {
  type = "report";
  title: string;

  constructor(t: string) {
    this.title = t;
  }
}

const bug = new BugReport("Needs dark mode");
console.log(bug);
```

```text
출력 결과:
{type: 'report', title: 'Needs dark mode', reportingURL: 'http://www.example.com'}
```

> 클래스의 타입이 변경되는 것은 아니다. 타입 시스템은 reportingURL을 인식하지 못하기 때문에 bug.reportingURL과 같이 직접 사용할 수 없다.

## 메서드 데코레이터

- 메서드의 속성 디스크립터(Property Descriptor. 속성의 특성을 설명하는 객체)에 적용되고 메서드의 정의를 읽거나 수정할 수 있음
- 선언 파일, 오버로드 메서드, 선언 클래스에 사용할 수 없음
- 다음 세 개의 인수를 가짐
  1. 정적 맴버가 속한 클래스의 생성자 함수이거나 인스턴스 멤버에 대한 클래스의 프로토타입
  2. 멤버의 이름
  3. 멤버의 속성 디스크립터. PropertyDescriptor 타입을 가짐
- 메서드 데코레이터가 값을 반환한다면 이는 해당 메서드의 속성 디스크립터가 된다.

다음 예시는 함수 실행 중 에러가 발생했을 때 이 에러를 잡아서 처리하는 로직을 구현한다.

```typescript
function HandleError() {
  return function(target: any, propertyKey: string, descriptor: PropertyDescriptor) { // 메서드 데코레이터가 가져야 할 3개의 인자. PropertyDescriptor는 객체 속성의 특성을 기술하는 객체
    console.log(target) // {constructor: ƒ, greet: ƒ}
    console.log(propertyKey) // hello
    console.log(descriptor) // {value: ƒ, writable: true, enumerable: false, configurable: true}

    const method = descriptor.value; // 디스크립터의 value 속성으로 원래 정의된 메서드를 저장

    descriptor.value = function() {
      try {
        method(); // 원래의 메서드를 호출
      } catch (e) { // 만약 원래의 메서드를 수행하는 과정에서 발생한 에러를 핸들링하는 로직을 이 곳에 구현
        // 에러 핸들링 로직 구현
        console.log(e); // Error: 테스트 에러가 출력
      }
    }
  };
}

class Greeter {
  @HandleError()
  hello() {
    throw new Error('테스트 에러');
  }
}

const t = new Greeter();
t.hello();
``` 

```typescript
interface PropertyDescriptor {
  configurable?: boolean;  // 속성의 정의를 수정할 수 있는지 여부
  enumerable?: boolean;    // 열거형인지 여부
  value?: any;             // 속성 값
  writable?: boolean;      // 수정 가능 여부
  get?(): any;             // getter
  set?(v: any): void;      // setter
}
```

## 접근자 데코레이터

- 접근자(getter, setter)의 속성 디스크립터에 적용되고 접근자의 정의를 읽거나 수정할 수 있음
- 선언 파일과 선언 클래스에 사용할 수 없음
- 접근자 데코레이터가 반환하는 값은 해당 멤버의 속성 디스크립터가 된다.

특정 멤버를 역러가 가능한 지 결정하는 데코레이터의 예를 보자.

```typescript
function Enumerable(enumerable: boolean) {
  return function (target: any, propertyKey: string, descriptor: PropertyDescriptor) {
    descriptor.enumerable = enumerable; // 디스크립터의 enumerable 속성을 데코레이터의 인자로 결정
  }
}

class Person {
  constructor(private name: string) {} // name은 외부에서 접근하지 못하는 private 멤버

  @Enumerable(true) // 게터 getName 함수는 열거가 가능하도록
  get getName() {
    return this.name;
  }

  @Enumerable(false) // 세터 setName 함수는 열거가 불가능하도록
  set setName(name: string) {
    this.name = name;
  }
}

const person = new Person('Dexter');
for (let key in person) { // 결과를 출력하면 getName은 출력되지만 setName은 열거하지 못하게 되었기 때문에 for문에서 key로 받을 수가 없음
  console.log(`${key}: ${person[key]}`);
    /**  출력 결과:
     *  name: Dexter
     *  getName: Dexter
     */
}

```

## 속성 데코레이터

- 선언 파일, 선언 클래스에서 사용 못함
- 다음 두 개의 인수를 가지는 함수이다.
  1. 정적 맴버가 속한 클래스의 생성자 함수이거나 인스턴스 멤버에 대한 클래스 프로토타입
  2. 멤버의 이름

```typescript
function format(formatString: string) {
  return function (target: any, propertyKey: string): any {
    let value = target[propertyKey];

    function getter() {
      return `${formatString} ${value}`; // 게터에서 데코레이터 인자로 들어온 formatString을 원래의 속성과 조합한 스트링으로 바꿈
    }

    function setter(newVal: string) {
      value = newVal;
    }

    return {
      get: getter,
      set: setter,
      enumerable: true,
      configurable: true,
    }
  }
}

class Greeter {
  @format('Hello') // 데코레이터에 formatString을 전달
  greeting: string;
}

const t = new Greeter();
t.greeting = 'World';
console.log(t.greeting); // 속성을 읽을 때 게터가 호출되면서 Hello World가 출력
```

## 매개변수 데코레이터

- 생성자 또는 메서드의 파라미터에 선언되어 적용
- 선언 파일, 선언 클래스에서 사용할 수 없음
- 3가지 인자와 함께 호출
  1. 정적 맴버가 속한 클래스의 생성자 함수이거나 인스턴스 멤버에 대한 클래스의 프로토타입
  2. 멤버의 이름
  3. 매개변수가 함수에서 몇 번째 위치에 선언되었는지를 나타내는 인덱스

파라미터가 제대로 된 값으로 전달되었는지 검사하는 데코레이터를 만들어 보자. 매개변수 데코레이터는 단독으로 사용하는 것보다 함수 데코레이터와 함께 사용할 때 유용하다.

> NestJS에서 API 요청 파라미터에 대한 유효성 검증을 할 때 이와 유사한 데코레이터를 많이 사용한다.

```typescript
import { BadRequestException } from '@nestjs/common';

function MinLength(min: number) { // 파라미터의 최소값을 검사하는 파라미터 데코레이터
  return function (target: any, propertyKey: string, parameterIndex: number) {
    target.validators = {   // target 클래스(여기서는 User)의 validators 속성에 유효성을 검사하는 함수를 할당
      minLength: function (args: string[]) { // args 인자는 18번 라인에서 넘겨받은 메서드의 인자이다.
        return args[parameterIndex].length >= min; // 유효성 검사를 위한 로직입니다. parameterIndex에 위치한 인자의 길이가 최소값보다 같거나 큰지 검사
      }
    }
  }
}

function Validate(target: any, propertyKey: string, descriptor: PropertyDescriptor) { // 함께 사용할 메서드 데코레이터
  const method = descriptor.value; // 메서드 데코레이터가 선언된 메서드를 method 변수에 임시 저장

  descriptor.value = function(...args) { // 디스크립터의 value에 유효성 검사 로직이 추가된 함수를 할당
    Object.keys(target.validators).forEach(key => { // target(User 클래스)에 저장해 둔 validators를 모두 수행한다. 이때 원래 메서드에 전달된 인자(args)들을 각 validator에 전달
      if (!target.validators[key](args)) {
        throw new BadRequestException();
      }
    })
    method.apply(this, args); // 원래의 함수를 실행
  }
}

class User {
  private name: string;

  @Validate
  setName(@MinLength(3) name: string) {
    this.name = name;
  }
}

const t = new User();
t.setName('Dexter'); // 파라미터 name의 길이가 5이기 때문에 문제가 없음
console.log('----------')
t.setName('De'); // 파라미터 name의 길이가 3보다 작기 때문에 BadRequestException이 발생
```

<br/>

# Controller

```bash
$ nest g controller [name] # 컨트롤러 자동 생성 ($ nest g co [name])
$ nest g resource [name] # CRUD 자동 생성. module, controller, service, entity, dto 코드와 테스트 코드를 자동 생성

$ nest -h # 그 외 명령어 확인
```

## 라우팅

- `@Contorller` 데코레이터를 클래스에 선언
  - 인자로 넘긴 값이 라우팅 경로 지정가능
  - ex) `@Controller('app')` -> `http://localhost:3000/app/...`
  
## 와일드카드 사용

`*` 문자를 사용하여 문자열 가운데 어떤 문자가 와도 상관없이 라우팅 패스를 구성할 수 있다.

```typescript
@Get('he*lo')
getHello(): string {
  return this.appService.getHello();
}
```

`*`외에도 `?, +, ()` 문자 역시 정규 표현식에서의 와일드카드와 동일하게 동작한다.

단, `-, .`은 문자열로 취급한다.

와일드카드는 컨트롤러의 패스를 정할 때만 사용하는 것이 아닌 많은 컴포넌트에서 이름을 정할 때 사용할 수 있다.


## 요청 객체

Nest는 요청과 함께 전달되는 데이터를 핸들러(컨트롤러)가 다룰 수 있도록 `@Req()` 데코레이터를 이용하여 변환한다.

```typescript
import { Request } from 'express';
import { Controller, Get, Req } from '@nestjs/common';
import { AppService } from './app.service';

@Controller()
export class AppController {
  constructor(private readonly appService: AppService) {}

  @Get()
  getHello(@Req() req: Request): string {
    console.log(req);
    return this.appService.getHello();
  }
}
```

Nest는 `@Query(), @Param(key?: string) , @Body` 데코레이터를 이용해서 요청에 포함된 쿼리 파라미터, 패스 파라미터, 본문을 쉽게 받을 수 있도록 한다.

## 응답

Nest는 이렇게 응답을 어떤 방식으로 처리할 지 미리 정의해 두었다. 

string, number, boolean과 같이 자바스크립트 원시 타입을 리턴할 경우 직렬화 없이 바로 보내지만, 객체를 리턴한다면 직렬화를 통해 JSON 으로 자동 변환해 준다.

이 방법이 권장하는 방법이긴 하지만 라이브러리별 응답 객체를 직접 다룰 수도 있다. 예를 들어 Express를 사용한다면 Express response object를 `@Res` 데코레이터를 이용해서 다룰 수 있다.

```typescript
@Get()
findAll(@Res() res) {
  const users = this.usersService.findAll()

  return res.status(200).send(users);
}
```

만약 상태코드를 다른 값으로 바꾸길 원한다면 데코레이터 `@HttpCode`를 사용하면 된다.

```typescript
import { HttpCode } from '@nestjs/common';

@HttpCode(202)
@Patch(':id')
update(@Param('id') id: string, @Body() updateUserDto: UpdateUserDto) {
  return this.usersService.update(+id, updateUserDto);
}
```

> HTTP 202 Accepted는 요청이 성공적으로 접수되었으나, 아직 해당 요청에 대해 처리 중이거나 처리 시작 전임을 의미한다. 요청이 처리 중 실패할 수도 있기 때문에 요청은 실행될 수도 실행되지 않을수도 있다. 이 상태 코드는 비확약적, 즉 HTTP가 나중에 요청 처리 결과를 나타내는 비동기 응답을 보낼 방법이 없다는 것을 의미한다. 이는 다른 프로세스나 서버가 요청을 처리하는 경우 또는 일괄 처리를 위한 것이다

예외처리하는 방법은 아래와 같다.

```typescript
@Get(':id')
findOne(@Param('id') id: string) {
  if (+id < 1) {
    throw new BadRequestException('id는 0보다 큰 값이어야 합니다.');
  }

  return this.usersService.findOne(+id);
}
```

```text
$ curl -X GET http://localhost:3000/users/0
{
  "statusCode": 400,
  "message": "id는 0보다 큰 값이어야 합니다.",
  "error": "Bad Request"
}
```

## 헤더

응답에 커스텀 헤더를 추가하고 싶다면 `@Header` 데코레이터를 사용하면 된다.

인자로 헤더 이름과 값을 받는다. 라이브러리에서 제공하는 응답객체를 사용해서 `res.header()` 메서드로 직접 설정도 가능하다.

```typescript
import { Header } from '@nestjs/common';

@Header('Custom', 'Test Header')
@Get(':id')
findOneWithHeader(@Param('id') id: string) {
  return this.usersService.findOne(+id);
}
```

## 리디렉션

`@Redirect` 데코레이터를 사용하여 리디렉션 사용이 가능하다.

데코레이터의 두번째 인자는 상태코드이다. `301 Moved Permanently`는 요청한 리소스가 헤더에 주어진 리소스로 완전히 이동됐음을 뜻한다.

`200`과 같은 다른 값으로 상태코드를 변경할 수 있으나 `301, 307, 308`과 같은 redirect로 정해진 응답코드가 아닐 경우 브라우저가 제대로 반응하지 않을 수 있다.

```typescript
import { Redirect } from '@nestjs/common';

@Redirect('https://nestjs.com', 301)
@Get(':id')
findOne(@Param('id') id: string) {
  return this.usersService.findOne(+id);
}
```

만약 요청 처리 결과에 따라 동적으로 리디렉트 하고 싶다면 아래와 같이 응답 객체를 같이 리턴하면 된다.

```typescript
// 응답 객체
{
  "url": string,
  "statusCode": number
}
```

```typescript
@Get('redirect/docs')
@Redirect('https://docs.nestjs.com', 302)
getDocs(@Query('version') version) {
  if (version && version === '5') {
    return { url: 'https://docs.nestjs.com/v5/' };
  }
}
```

참고로 Nest는 자바스크립트 객체를 리턴하면 JSON 스트링으로 직렬화 해서 보내준다.

## 라우트 파라미터

라우트 파라미터는 패스 파라미터라고도 한다.

`@Param` 데코레이터로 주입받을 수 있다.

라우트 파라미터를 전달받는 방법은 2가지 있다.

1. 파라미터가 여러 개 전달될 경우 객체로 한번에 받는 방법
   - 이 방법은 params의 타입이 any가 되어 권장하지 않음
   - 라우트 파라미터는 항상 타입이 string 이기 때문에 명시적으로 `{ [key: string]: string }` 타입을 지정해 주어도 된다.
```typescript
@Delete(':userId/memo/:memoId')
deleteUserMemo(@Param() params: { [key: string]: string }) {
return `userId: ${params.userId}, memoId: ${params.memoId}`;
}
```
2. 일반적인 방법은 라우팅 파라미터를 따로 받는 것
   - REST API 구성 시 라우팅 파리미터의 개수가 너무 많아지지 않게 설계하는 것이 좋으므로 따로 받아도 코드가 많이 길어지지는 않음.
```typescript
@Delete(':userId/memo/:memoId')
deleteUserMemo(
  @Param('userId') userId: string,
  @Param('memoId') memoId: string,
) {
  return `userId: ${userId}, memoId: ${memoId}`;
}
```

## 하위 도메인(Sub-Domain) 라우팅

현재 회사가 사용하고 있는 도메인은 `example.com`이고, API 요청은 `api.example.com`으로 받기로 했다고 하자.

`http://example.com`, `http://api.example.com`로 들어온 요청을 서로 다르게 처리하거나 하위 도메인에서 처리하지 못하는 요청은 원래의 도메인에서 처리하도록 하고 싶을 경우 **하위 도메인 라우팅 기법**을 사용한다.

```typescript
@Module({
  controllers: [ApiController, AppController],
    ...
})
export class AppModule { }
```

- `ApiController`가 `AppController` 보다 우선순위가 높도록 설정

`@Controller` 데코레이터는 ControllerOptions 객체를 인자로 받는데 host 속성에 하위 도메인을 기술하면 된다.

```typescript
@Controller({ host: 'api.example.com' }) // 하위 도메인 요청 처리 설정
export class ApiController {
  @Get() // 같은 루트 경로
  index(): string {
    return 'Hello, API'; // 다른 응답
  }
}
```

`@HostParam` 데코레이터를 이용하면 서브 도메인을 변수로 받을 수 있다.

API 버저닝을 하는 방법이 여러가지 있지만 하위 도메인을 이용하는 방법을 많이 사용한다.

아래와 같이 하위 도메인 라우팅으로 쉽게 API를 버전별로 분리할 수 있다.

```typescript
@Controller({ host: ':version.api.localhost' })
export class ApiController {
  @Get()
  index(@HostParam('version') version: string): string {
    return `Hello, API ${version}`;
  }
}
```

host param이 없는 host로 요청을 하면 기존 도메인으로 요청이 처리된다.

## 페이로드 다루기

POST, PUT, PATCH 요청은 보통 처리에 필요한 데이터를 함께 실어 보내는데 이 데이터 덩어리(페이로드)를 본문(body)라고 한다.

NestJS는 본문을 DTO를 정의하여 쉽게 다룰 수 있다.

```typescript
export class CreateUserDto {
  name: string;
  email: string;
}
```

```typescript
@Post()
create(@Body() createUserDto: CreateUserDto) {
  const { name, email } = createUserDto;

  return `유저를 생성했습니다. 이름: ${name}, 이메일: ${email}`;
}
```

GET 요청에 대해 `/users?offset=0&limit=10`와 같은 쿼리 파라미터는 `@Query` DTO로 묶어 처리 가능하다.

```typescript
export class GetUsersDto {
  offset: number;
  limit: number;
}

```

<br/>

# 참고

- [NestJS로 배우는 백엔드 프로그래밍](https://wikidocs.net/book/7059)