### 3.1 타입스크립트만의 독자적 타입 시스템

![alt text](https://velog.velcdn.com/images/bami/post/f3b873f5-f12d-4cec-b885-bd9c84c55ea1/image.png)


1) any 타입
any 타입은 자바스크립트의 사용 방식과 일치하여 자바스크립트에 존재하는 모든 값을 오류 없이 받을 수 있다. 즉, 자바스크립트에서의 기본적인 사용 방식과 같으므로 타입을 명시하지 않은 것과 동일한 효과를 나타낸다.

2) unknown 타입
: unknown 타입은 any 타입과 유사하게 모든 타입의 값이 할당될 수 있다. 그러나 any를 제외한 다른 타입으로 선언된 변수에 unknown 타입의 값을 할당할 수 없다.

```
let unknownValue: unknown;

unknownValue = 100; // any 타입과 유사하게 숫자이든
unknownValue = "hello mobi"; // 문자열이든
unknownValue = () =>  console.log("this is any type")
// 함수이든 상관없이 할당 가능하지만

let someValue1: any = unknownValue; // (o) any 타입으로 선언된 변수를 제외한 다른 변수는 모두 할당이 불가
let someValue2: number = unknownValue; // (x)
let someValue3: string = unknownValue; // (x)

```

3) void 타입
: 타입스크립트에서 어떤 값을 반환하지 않는 함수

4) never 타입
: never 타입은 모든 타입의 하위 타입이다. 즉, never 자신을 제외한 어떤 타입도 never 타입에 할당될 수 없다는 것을 의미한다. any 타입이라 할지라도 never 타입에 할당될 수 없다.

5) Array type
- 자바스크립트의 Object.prototype.toString.call() 연산자를 사용하여 확인 가능
- JS 에서는 배열을 객체에 속하는 타입으로 분류한다. 하나의 배열에 여러 타입의 원소가 들어갈 수 있는데, 이는 정적 언에와 맞지 않는다.

```
// 두 방식은 큰 차이가 없다
const array: number[] = [1,2,3];
const array2: Array<number> = [1,2,3];
```

- 튜플 : 기존 타입스크립트의 배열 + 길이제한 추가한 타입 시스템
```
let tuple: [number, string] = [1, 'string'];
```
- useState API가 튜플 타입을 반환한다. 첫 번째 원소는 상태값, 두 번째 원소는 setter

- 튜플과 배열의 성질을 혼합해서 사용할 수도 있다.
```
const httpStatusFromPaths: [number, string, ...string[]] = [
    400,
    "Bad Request",
    "/user/:id",
    "/user/:userId",
]
```
- 튜플에 옵셔널 프로퍼티 명시도 가능하다.

6) enum 타입 (열거형)
- ts에서 지원하는 특수한 타입
- 명명한 각 멤버의 값을 스스로 추론한다. (0부터 1씩 늘림)

```
enum ProgrammingLanguage {
  Typescript, // 0
  Javascript, // 1
  Java, // 2
  Python, // 3
  Kotlin, // 4
  Rust, // 5
  Go, // 6
}

// 각 멤버에게 접근하는 방식은 자바스크립트에서 객체의 속성에 접근하는 방식과 동일
ProgrammingLanguage.Typescript; // 0
ProgrammingLanguage.['Rust']; // 5

// 역방향 접근 또한 가능하다.
ProgrammingLanguage[2]; // 'Java'
```

- 각 멤버에 명시적으로 값을 할당할 수 있다. 직접 값을 할당하지 않으면 이전 멤버 값을 기준으로 1씩 늘려가며 자동으로 할당한다.

```
enum ProgrammingLanguage {
  Typescript = 'Typescript,
  Javascript = "JavaScript",
  Java = 300,
  Python = 400, 
  Kotlin, // 401
  Rust, // 402
  Go, // 403
}
```

- 주로 문자열 상수를 생성하는데 사용된다. 응집력있는 집합 구조체를 만들 수 있다.
```
enum ItemStatusType {
  DELIVERY_HOLD = 'DELIVERY_HOLD', // 배송 보류
  DELIVERY_READY = 'DELIVERY_READY', // 배송 준비 중
  DELIVERING = 'DELIVERING', // 배송 중
  DELIVERED = 'DELIVERED', // 배송 완료
}

const checkItemAvailable = (itemStatus: ItemStatusType) => {
  switch(itemStatus) {
    case ItemStatusType.DELIVERY_HOLD:
    case ItemStatusType.DELIVERY_READY:
    case ItemStatusType.DELIVERING:
      return false;
    case ItemStatusType.DELIVERED:
    default:
      return true;
  }
}
```

- 다만, 숫자로만 이루어지거나 타입스크립트가 자동으로 추론한 열거형은 안전하지 않다.
- const enum 은 이런 문제를 일부 방지할 수 있으나, 숫자 상수로 관리되는 열거형은 선언한 값 이외의 값을 할당하거나 접근할 때 이를 방지하지 못한다.
```
ProgrammingLanguage[200] // undefined 이나 에러를 뱉지 않는다.
const enum ProgrammingLanguage {
  // ...
}
```

```
const enum NUMBER {
  ONE: 1,
  TWO: 2,
}

const myNumber: NUMBER = 100; // NUMBER enum에서 100을 관리하고 있지 않지만 에러를 발생시키지 않는다

const enum STRING_NUMBER {
  ONE: 'ONE',
  TWO: 'TWO',
}

const myStringNumber: STRING_NUMBER = 'THREE'; // Error
```

- 또한, 열거형은 컴파일 시 즉시실행함수 형식으로 변환된다. 이 경우, 일부 번들러에서 즉시실행함수로 변환된 값을 사용하지 않는 코드로 인식하지 못하는 경우가 발생할 수 있다. -> const enum 또는 as const assertion 사용하기



### 3.2 타입 조합

#### 1) 교차타입 intersection => &을 사용
: 기존 존재하는 타입을 합쳐 모든 멤버를 가지는 새로운 타입 생성하기
```
// C는 A, B 모든 타입을 가지고 있음
type C = A & B
```

#### 2) 유니온 타입 Union

```
// C는 A B 중 하나가 될 수 있는 타입
type C = A | B
```

#### 3) 인덱스 시그니처 Index Signatures -> 인터페이스 내부에 [Key: T]
: 특정 타입의 속성 이름은 알 수 없으나, 속성값의 타입을 알고 있을 때 사용
```
interface IndexSignaturesEx {
  [key: string]: number;
}
```

#### 4) 인덱스드 엑세스 타입(Indexed Access Types)
: 다른 타입의 특정 속성이 가지는 타입을 조회하기 위해 사용

```
type Example = {
  a: number;
  b: string;
  c: boolean;
}

type IndexedAccess = Example["a"]; // number
type IndexedAccess2 = Example["a" | "b"]; // number | string
type IndexedAccess3 = Example[keyof Example]; // number | string | boolean

type ExAlias = "b" | "c";
type IndexedAccess4 = Example[ExAlias]; // string | boolean

```

- 배열의 요소 타입을 조회하기 위해 인덱스드 엑세스 타입을 사용하는 경우가 있다.
number로 인덱싱하여 배열 요소를 얻은 후 typeof 연산자를 부텨주면 해당 배열 요소의 타입을 가져올 수 있다.
```
const ProductionList = [
  { type: "product", name: "chicken" },
  { type: "product", name: "pizza" },
  { type: "card", name: "cheer-up" },
]

type ElementOf<T> = typeof T[number];
type PromotionItemType = ElementOf<ProductionList>;
```

#### 5) 맵드 타입(Mapped Types)
- map : 유사한 형태를 가진 여러 항목의 목록 A를 변환된 항목의 목록 B로 바꾸는 것
```
// 반복 타입 선언을 줄일 수 있다.
type Example = {
  a: number;
  b: string;
  c: boolean;
};

type Subset<T> = {
  [K in keyof T] = T[K];
};

const aExample: Subset<Example> = { a: 3 };
const bExample: Subset<Example> = { b: "hello" };
const cExample: Subset<Example> = { a: 4, c: true };
```

- readonly, ? 수식어 제외가 가능하다.
```
type ReadOnlyEx = {
  readonly a: number;
  readonly b: string;
};

type CreateMutable<Type> = {
  -readonly [Property in keyof Type]: Type[Property];
};

type ResultType = CreateMutable<ReadOnlyEx>; // { a: number; b: string }

type OptionalEx = {
  a?: number;
  b?: string;
  c?: boolean;
};

type Concrete<Type> = {
  [Property in keyof Type]-?: Type[Property];
};

type ResultType = Concrete<OptionalEx>; // { a: number; b: string; c: boolean }

```

- 실 예시
```
const BottomSheetMap = {
  RECENT_CONTACTS: RecentContactBottomSheet,
  CARD_SELECT: CardSelectBottomSheet,
  SOFT_FILTER: SoftFilterBottomSheet,
  PRODUCT_SELECT: ProductSelectBottomSheet,
  REPLAY_CARD_SELECT: ReplayCardSelectBottomSheet,
  RECEND: RecendBottomSheet,
  STICKER: StickerBottomSheet,
  BASE: null,
};

export type BOTTOM_SHEET_ID = keyof typeof BottomSheetMap;

// 불필요한 반복이 발생된다.
type BottomSheetStore = {
  RECENT_CONTACTS: {
    resolver?: (payload: any) => void;
    args?: any;
    isOpened: boolean;
  };
  CARD_SELECT: {
    resolver?: (payload: any) => void;
    args?: any;
    isOpened: boolean;
  };
  SOFT_FILTER: {
    resolver?: (payload: any) => void;
    args?: any;
    isOpened: boolean;
  };
  // ...
};

// Mapped Types를 통해 효율적으로 타입 선언을 할 수 있다.
type BottomSheetStore = {
  [index in BOTTOM_SHEET_ID]: {
    resolver?: (payload: any) => void;
    args?: any;
    isOpened: boolean;
  };
};
```

- as 키워드를 사용하여 키를 재지정할 수 있다.
```
type BottomSheetStore = {
  [index in BOTTOM_SHEET_ID as `${index}_BOTTOM_SHEET`]: {
    resolver?: (payload: any) => void;
    args?: any;
    isOpened: boolean;
  };
};
```

#### 6) 템플릿 리터럴 타입(Temeplate Literal Types)
: 자바스크립트의 템플릿 리터럴 문자를 사용하여 문자열 리터럴 타입을 선언할 수 있는 문법
```
type Stage = 
| "init"
| "select-image"
| "edit-image"
| "decorate-card"
| "capture-image";

type StageName = `${Stage}-stage`;
// 'init-stage' | 'select-image-stage' ...
```

#### 7) 제네릭(Generic)
: 제네릭은 C나 자바 같은 정적 언어에서 다양한 타입 간에 재사용성을 높이기 위해 사용하는 문법
- 함수, 타입 클래스 등에서 내부적으로 사용할 타입을 미리 정해두지 않고 타입 변수를 사용해서 해당 위치를 비워 둔 다음에, 실제로 그 값을 사용할 때 외부에서 타입 변수 자리에 타입을 지정하여 사용하는 방식

- 재사용성이 향상된다.

```
type ExampleArrayType<T> = T[];

const array1: ExampleArrayType<string> = ["치킨", "피자", "우동"];
```

- 제네릭 함수를 호출할 때 반드시 꺽쇠괄호 (<>) 안에 타입을 명시해야 하는 것은 아니다. 타입을 명시하는 부분을 생략하면 컴파일러가 인수를 보고 타입을 추론해준다.
```
function exampleFunc<T>(arg: T): T[] {
  return new Array(3).fill(arg);
};

exampleFunc('hello'); // T는 string으로 추론된다
```

- 특정 요소 타입을 알 수 없을 때는 제네릭 타입에 기본값을 추가할 수 있다.
```
interface SubmitEvent<T = HTMLElement> extends SyntheticEvent<T> { submitter: T; }
```

- 특정 속성만 가진 타입만 받게 제약을 걸 수도 있다.
```
interface TypeWithLength {
  length: number;
}

function exampleFunc3<T extends TypeWithLength>(arg: T): number {
  return arg.length;
}
```

- 파일 확장자가 tsx일 때 화살표 함수에 제네릭을 사용하면 에러가 발생한다. tsx는 타입스크립트 + JSX 이므로 제네릭의 꺽쇠괄호와 태그의 꺽쇠괄호를 혼동하여 문제가 생기는 것
- 이러한 상황을 피하기 위해서는 제네릭 부분에 extends 키워드를 사용하여 컴파일러에게 특정 타입의 하위 타입만 올 수 있음을 확실히 알려주면 된다. 보통 제네릭을 사용할 때는 function 키워드로 선언하는 경우가 많다.

```
// 에러 발생: JSX Element 'T' has no corresponding closing tag
const arrowExampleFunc = <T>(arg: T): T[] => {
  return new Array(3).fill(arg);
};

// 에러 발생 x
const arrowExampleFunc2 = <T extends {}>(arg: T): T[] => {
  return new Array(3).fill(arg);
};       
```


## 3.3 제네릭 사용법

### 1) 함수의 제네릭
: 어떤 함수의 매개변수나 반환 값에 다양한 타입을 넣고 싶을 때 제네릭을 사용할 수 있다.

```
function ReadOnlyRepository<T>(target: ObjectType<T> | EntitySchema<T> | string): Repository<T> {
  return getConnection("ro").getRepository(target);
};
```

### 2) 호출 시그니처의 제네릭
- 호출 시그니처는 타입스크립트의 함수 타입 문법으로 함수의 매개변수와 반환 타입을 미리 선언하는 것
- 제네릭을 사용하면 동적 타입 정의가 가능하다.
```
// 제네릭 호출 시그니처
type GetFirstElement<T> = (arr: T[]) => T;

const getFirstElement: GetFirstElement<number> = (arr) => arr[0];

console.log(getFirstElement([1, 2, 3])); // 출력: 1

// 다른 타입으로도 사용 가능
const getFirstString: GetFirstElement<string> = (arr) => arr[0];

console.log(getFirstString(["a", "b", "c"])); // 출력: "a"

```

### 3) 제네릭 클래스
: 외부에서 입력된 타입을 클래스 내부에 적용할 수 있는 클래스

### 4) 제한된 제네릭
- 제한된 제네릭은 타입 매개변수에 대한 제약 조건을 설정하는 기능을 말한다. 
-  string 타입으로 제약하려면 타입 매개변수는 특정 타입을 상속(extends)해야 한다.

```
type ErrorRecord<Key extends string> = Exclude<Key, ErrorCodeType> extends never
? Partial<Record<Key, boolean>>
| never;

// Exclude<T, U>: TypeScript 유틸리티 타입으로, T에서 U에 해당하는 타입들을 제외한 타입을 반환
// Exclude<Key, ErrorCodeType>가 never 타입이라면 조건식이 참(true)
// Exclude<Key, ErrorCodeType>이 never인 경우:
// Key가 모두 ErrorCodeType에 속한다는 뜻입니다.
// 즉, Key와 ErrorCodeType이 완전히 겹칠 때만 참이 됩니다.
// Partial<Record<Key, boolean>>는 Key를 키로 가지는 불린 값의 선택적 객체 타입입니다.
```

### 5) 확장된 제네릭
- 제네릭 타입은 여러 타입을 상속받을 수 있으며 타입 매개변수를 여러 개 둘 수도 있다.

