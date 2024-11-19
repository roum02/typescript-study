## 타입

### 2.1 타입이란

1. 자료형으로서의 타입
- 컴퓨터의 메모리 공간은 한정적이기 때문에, 값을 효율적으로 저장하기 위해선 메모리 공간을 차지할 값의 크기를 알아햐 함
- 자바스크립트는 7가지 자료형(data type)을 정의한다.

| 데이터 타입           | 설명                                                                                     |
|--------------------|-----------------------------------------------------------------------------------------|
| **Number**         | 숫자 데이터 타입으로 정수와 부동소수점 숫자를 모두 포함합니다.                                         |
| **String**         | 텍스트 데이터를 나타내는 타입으로, 큰따옴표(`"`)나 작은따옴표(`'`), 또는 백틱(````)으로 감쌉니다.           |
| **Boolean**        | 논리적 데이터를 표현하며, `true` 또는 `false` 값을 가집니다.                                       |
| **Undefined**      | 값이 정의되지 않았음을 나타냅니다.                                                               |
| **Null**           | 값이 없음을 나타내는 타입으로, 의도적으로 "비어있음"을 표현합니다.                                      |
| **Symbol**         | 고유하고 변경 불가능한 값으로, 주로 객체의 키(key)로 사용됩니다.                                      |
| **Object**         | 객체 또는 참조 타입으로, 키-값 쌍을 저장하는 컨테이너 역할을 합니다.                                    |

- 데이터타입은 여러 종류의 데이터를 식별하는 분류체계로 컴파일러에 값의 형태를 알려준다.

    | **컴파일러(Compiler)**는 프로그래밍 언어로 작성된 소스 코드를 컴퓨터가 실행할 수 있는 기계어로 변환해주는 프로그램
    | TypeScript Compiler, tsc : TypeScript 코드를 JavaScript 코드로 변환하는 컴파일러
    | 변환된 JS 는 인터프리터에 의해 실행



2. 집합으로서의 타입
- 타입: 값이 가질 수 있는 유효한 범위의 집합
- 타입 스스템은 코드에서 사용되는 유효한 값의 범위를 제한하여 런타임에 발생할 수 있는 에러를 방지한다.
    | **런타임(Runtime)**은 프로그램이 실행되고 있는 실시간 환경


3. 정적 타입과 동적 타입

: 자바스크립트에서도 타입이 존재하나, 개발자가 컴파일 이전에 타입을 직접 정의해줄 필요가 없었을 뿐이다.
타입을 결정하는 시점에 따라 정적 타입과 동적 타입으로 나눌 수 있다.

- 정적 타입 시스템: 모든 변수의 타입이 컴파일타임에 결정
- 동적 타입 시스템: 변수 타입이 런타임에서 결정


4. 강타입과 약타입

- 암묵적 타입 변환: 개발자가 의도적으로 타입을 바꾸지 않았음에도 컴파일러/엔진 등에 의하여 런타임에 타입이 자동으로 변경되는 것
- 암묵적 타입 반환 여부에 따라 타입 시스템을 강타입/약타입으로 분류할 수 있다.

- 강타입: 서로 다른 타입을 갖는 값끼리 연산을 시도하면 컴파일러/인터프리터 에러가 발생한다.
- 약타입: 서로 다른 타입을 갖는 값끼리 연산시 컴파일러/인터프리터가 내부적으로 판단해서 타입 변환하여 연산 수행한다.

- 타입스크립트는 직접 타입 명시 / 타입스크립트의 타입 추론 -> 두 방식 모두에 영향을 받았음 (뒤에서 살펴봄)

5. 컴파일 방식
- 컴파일: 사람이 이해할 수 있는 언어(high-level)를 컴퓨터가 이해 가능한 기계어(binary cord)로 바꿔주는 과정
- 타입스크립트의 컴파일 결과물은 여전히 사람이 이해할 수 있는 방식인 자바스크립트 파일이다. 타입스크립트의 탄생 이유는, 사람이 이해하기 쉬운 방식으로 코드를 작성하기 위해서가 아니라, 자바스크립트 컴파일타임에 런타임 에러를 사전에 잡아내기 위한 것이다.


### 2.2 타입스크립트의 타입 시스템

1. 타입 애너테이션 방식 type annotation
: 변수나 상수, 함수의 인자와 반환 값에 타입을 명시적으로 선언해서 어떤 타입 값이 저장될 것인지를 컴파일러에 직접 알려주는 문법

- 타입스크립트는 기존 자바스크립트 코드에 점진적으로 타입을 적용할 수 있는 특징을 가지고 있어, type 선언부를 제거해도 코드가 정상 동작함.

```
let color: string = 'blue';
```

2. 구조적 타이핑
: **구조적 타이핑(Structural Typing)**은 객체나 데이터의 **구조와 형태(Shape)**를 기반으로 타입 호환성을 판단하는 타입 시스템. 즉, 데이터의 속성과 구조가 맞으면 같은 타입

c.f. 명목적 타이핑: 
타입 이름이나 선언 자체를 기준으로 타입 호환성을 판단.
구조가 같더라도 다른 타입으로 선언된 경우, 호환되지 않음.
대표적 언어: Java, C#


3. 구조적 서브타이핑
: 객체가 가지고 있는 속성을 바탕으로 타입을 구분하는 것. 이름이 다르더라도, 속성이 동일하다면 호환이 가능한 타입으로 여긴다.


** 구조적 서브타이핑은 구조적 타이핑의 특정한 동작 방식: 

구조적 타이핑: 두 객체의 구조가 같으면 같은 타입으로 간주.
구조적 서브타이핑: 한 객체가 다른 객체의 구조를 포함하고 있으면, **하위 타입(subtype)**으로 간주.

```
interface Person {
    name: string;
    age: number;
}

const person = { name: "Alice", age: 25 };
const user: Person = person; // 구조적으로 같으므로 호환 가능

_

interface Animal {
    name: string;
}

interface Dog extends Animal {
    breed: string;
}

const dog: Dog = { name: "Buddy", breed: "Golden Retriever" };
const animal: Animal = dog; // Dog은 Animal의 서브타입으로 호환 가능

```

4. 자바스크립트를 닮은 타입스크립트
: 타입스크립트가 구조적 타이핑을 채택한 이유는 자바스크립트를 모델링한 언어이기 때문. (duck typing)

5. 구조적 타이핑의 결과

```
interface Cube {
  width: number;
  height: number;
  depth: number;
}

function addLines(c: Cube) {
  let total = 0;

  for (const axis of Object.keys(c)) {
    // 🚨 Element implicitly has an 'any' type
    // because expression of type 'string' can't be used to index type 'Cube'
    // 🚨 No index signature with a parameter of type 'string'
    // was found on type 'Cube'
  }
}

```

-> c에 들어올 객체는 width, height, depth 외에도 (number type이 아닌) 다른 속성을 얼마든지 가질 수 있기 때문에 에러가 발생한다.

=> 이러한 한계를 극복하고자, 명목적 타이핑 언어의 특징을 가미한 유니온 방법이 생겨났다.

6. 타입스크립트의 점진적 타입 확인

: 점진적 타입 검사란 컴파일 타임에 타입을 검사하면서, 필요에 따라 타입 선언 생략을 허용하는 방식이다. (타입스크립트는 점진적 타입 확인 언어이다)
타입을 지정한 변수와 표현식은 정적으로 타입을 검사하지만, 타입 선언이 생략되면 동적으로 검사를 수행한다. 


7. 자바스크립트 슈퍼셋으로서의 타입스크립트
: 타입스크립트는 자바스크립트의 상위 집합이다.

8. 값 vs 타입
- 값: 프로그램이 처리하기 위해 메모리에 저장하는 모든 데이터( 문자열, 숫자, 변수, 매개변수, 객체, 함수)

값 공간과 타입 공간의 이름은 서로 충돌하지 않기 때문에, 타입과 변수를 같은 이름으로 정의할 수 있다.
(타입스크립트 문법은 자바스크립트 런타임에서 제거되기 때문에, 값 공간과 타입 공간은 충돌하지 않는다.)

```
type Developer = {...}
const Developer = {...}
```

- class , enum : 값과 타입 공간에 동시에 존재하는 심볼

** 자바스크립트 es6에서 등장한 클래스는 객체 인스턴스를 더욱 쉽게 생성하기 위한 문법 기능으로, 실제 동작은 함수와 같다. 동시에 클래스는 타입으로도 사용된다. (값과 타입 공간 모두에 포함됨)

```
class Developer {
  name: string;
  domain: string;

  constructor(name: string, domain: string) {
    this.name = name;
    this.domain = domain;
  }
}

const me: Developer = new Developer("zig", "frontend");

```

** class와 마찬가지로 타입스크립트 문법인 enum 역시 런타임에 객체로 변환되는 값이다. enum은 런타임에 실제 객체로 존재하며, 함수로 표현할 수도 있다.

```
enum Direction {
  Up,    // 0
  Down,  // 1
  Left,  // 2
  Right, // 3
}

```

```
"use strict";
var Direction;
(function (Direction) {
    Direction[Direction["Up"] = 0] = "Up";
    Direction[Direction["Down"] = 1] = "Down";
    Direction[Direction["Left"] = 2] = "Left";
    Direction[Direction["Right"] = 3] = "Right";
})(Direction || (Direction = {}));

```

// 타입으로도 사용됨 
```
enum WeekDays {
  MON = "Mon",
  TUES = "Tues",
  WEDNES = "Wednes",
  THURS = "Thurs",
  FRI = "Fri",
}

// 'MON' | 'TUES' | 'WEDNES' | 'THURS' | 'FRI'
type WeekDaysKey = keyof typeof WeekDays;

function printDay(key: WeekDaysKey, message: string) {
  const day = WeekDays[key];
  if (day <= WeekDays.WEDNES) {
    console.log(`It's still ${day}, ${message}`);
  }
}

printDay("TUES", "wanna go home");

```

// 값으로도 사용됨
```
enum MyColors {
  BLUE = "#0000FF",
  YELLOW = "#FFFF00",
  MINT = "#2AC1BC",
}

function whatMintColor(palette: { MINT: string }) {
  return palette.MINT;
}

whatMintColor(MyColors); // ✅

```

=> 타입스크립트에서 어떠한 심볼이 값으로 사용된다는 것은 컴파일러를 이용해서 타입스크립트 파일을 자바스크립트 파일로 변환해도 여전히 자바스크립트에 해당 정보가 남아있음을 의미한다.

9. 타입을 확인하는 방법
: typeof, instanceof , 타입 단언을 사용하여 확인할 수 있다.
typeof 연산자가 반환하는 값은 자바스크립트의 7가지 기본 데이터 타입과 함수, 호스트 객체, Object 객체가 될 수 있다.

typeof는 값에서 쓰일 때와 타입에서 쓰일 때의 역할이 다르다.
- 값에서 사용된 typeof는 자바스크립트 런타임의 typeof 연산자가 된다.

```
interface Person {
    first: string;
    last: string;
}

const person: Person = {first: "zig", last: "song" }

const v1 = typeof person // 값은 object
```
*** c.f. TypeScript의 타입(Person)은 컴파일 타임에만 존재합니다.
JavaScript에서 typeof는 런타임 값의 타입을 반환하며, 객체는 항상 "object"로 출력됩니다.


- 타입에서 사용된 typeof는 값을 읽고 타입스크립트 타입을 반환한다.

```
type T1 = typeof person // 타입은 Person
```

```
class Developer {
  name: string;
  domain: string;

  constructor(name: string, domain: string) {
    this.name = name;
    this.domain = domain;
  }
}

const d = typeof Developer // 값이 function
type T = typeof Developer // 타입이 typeof Developer
```

### 2.3 원시 타입

- 자바스크립트에서 값은 타입을 가지나, 변수는 별도의 타입을 가지지 않는다. 따라서 자바스크립트의 변수에는 어떤 타입의 값이라도 자유롭게 할당할 수 있다. 
- 타입스크립트에서는 변수에 타입을 지정할 수 있다. 자바스크립트의 7가지 원시 값은 타입스크립트에서 원시 타입으로 존재한다.

 | boolean, undefined, null, number(NaN, Infinity), bigint(ES2020 에서 도입), string, symbol
 

### 2.4 객체 타입
- 앞서 언급한 7가지 원시 타입에 속하지 않는 값
- Object : 가급적 사용 금지, any 타입과 유사하게 객체에 해당하는 모든 타입 값 유동적으로 할당할 수 있기 때문
- {} : 빈 객체 타입. 안에 속성 할당 불가. 해당 타입 사용보다는 유틸리티 타입으로 REcord<string, never> 가 바람직
- Array: 한 배열 안에 타입 제한 없이 다양한 값을 다루는 자바스크립트와는 다르게, 타입스크립트는 배열을 Array 라는 별도 타입으로 다룬다. 하나의 타입 값만 가질 수 있다.
  - c.f. Tuple: 고정된 요소 수 만큼의 타입을 미리 선언하고, 배열을 표현하는 타입. 배열의 요소 수와 타입이 고정되어 있어, 요소 수가 미리 정해진 배열을 표현할 때 사용한다.
- type과 interface 키워드 
  - 타입스크립트에서만 독자적으로 사용할 수 있는 키워드로 객체를 타이핑하기 위해 쓰임.
- function
  - 자바스크립트에서 함수도 일종의 객체로 간주하지만 typeof를 쓸 경우 별도의 function 타입이 존재