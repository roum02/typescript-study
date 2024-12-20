# 4. 타입 확장하기 · 좁히기

<br/>

## 4.1 타입 확장하기

타입 확장은 기존 타입을 사용해서 새로운 타입을 정의하는 것을 말한다. 기본적으로 타입스크립트에서는 extends, 교차 타입, 유니온 타입을 사용하여 타입을 확장한다.

<br/>

### 4.1.1 타입 확장의 장점

타입 확장의 가장 큰 장점은 코드 중복을 줄일 수 있다는 것이다. 타입스크립트 코드 작성시, 중복되는 타입 선언이 생길 수 있는데, 이 때 중복되는 타입을 반복적으로 선언하는 것보다 기존에 작성한 타입을 바탕으로 타입 확장을 함으로써 불필요한 코드 중복을 줄일 수 있다.

<br/>

```tsx
/**
 * 메뉴 요소 타입
 * 메뉴 이름, 이미지, 할인율, 재고 정보를 담고 있다
 * */
interface BaseMenuItem {
  itemName: string | null;
  itemImageUrl: string | null;
  itemDiscountAmount: number;
  stock: number | null;
}

/**
 * 장바구니 요소 타입
 * 메뉴 타입에 수량 정보가 추가되었다
 */
interface BaseCartItem extends BaseMenuItem {
  // BaseMenuItem의 모든 타입을 가지면서, quantity 타입을 추가함.
  quantity: number;
}
```

장바구니 요소는 메뉴 요소가 가지는 모든 타입이 필요하지만, BaseMenuItem에 있는 속성을 중복해서 작성하지 않고 확장을 활용하여 타입을 정의했다. 이처럼 타입 확장은 중복된 코드를 줄일 수 있게 해준다.

<br/>

타입 확장은 중복 제거, 명시적인 코드 작성 외에도 확장성이란 장점을 가지고 있다. 앞서 정의한 `BaseCartItem`을 활용하면 요구 사항이 늘어날 때마다 새로운 CartItem 타입을 확장하여 정의할 수 있다.

<br/>

```tsx
/**
 * 수정할 수 있는 장바구니 요소 타입
 * 품절 여부, 수정할 수 있는 옵션 배열 정보가 추가되었다
 */
interface EditableCartItem extends BaseCartItem {
  isSoldOut: boolean;
  optionGroups: SelectableOptionGroup[];
}

/**
 * 이벤트 장바구니 요소 타입
 * 주문 가능 여부에 대한 정보가 추가되었다
 */
interface EventCartItem extends BaseCartItem {
  orderable: boolean;
}
```

이 코드에서 BaseCartItem을 확장하여 만든 EditableCartItem, EventCartItem 타입을 볼 수 있다. 이렇게 타입 확장을 활용하면 장바구니와 관련된 요구 사항이 생길 때마다 필요한 타입을 손쉽게 만들 수 있다. 더욱이, 기존 장바구니 요소에 대한 요구 사항이 변경되어도 BaseCartItem 타입만 수정하고 EditableCartItem이나 EventCartItem은 수정하지 않아도 되기 때문에 효율적이다.

<br/><br/>

### 4.1.2 유니온 타입

유니온 타입은 2개 이상의 타입을 조합하여 사용하는 방법이다. 집합 관점으로 보면 합집합으로 해석할 수 있다.

<br/>

```tsx
type MyUnion = A | B;
```

A와 B의 유니온 타입인 MyUnion은 타입 A와 B의 합집합이다. 집합 A의 모든 원소는 집합 MyUnion의 원소이며, 집합 B의 모든 원소 역시 집합 MyUnion의 원소라는 뜻이다. 즉, A 타입과 B 타입의 모든 값이 MyUnion 타입의 값이 된다.

<br/><br/>

주의해야 할 점은, 유니온 타입으로 선언된 값은 **유니온 타입에 포함된 모든 타입이 공통으로 갖고 있는 속성에만 접근**할 수 있다.

<br/>

```tsx
interface CookingStep {
  orderId: string;
  price: number;
}

interface DeliveryStep {
  orderId: string;
  time: number;
  distance: string;
}

function getDeliveryDistance(step: CookingStep | DeliveryStep) {
  return step.distance;
  // Property ‘distance’ does not exist on type ‘CookingStep | DeliveryStep’
  // Property ‘distance’ does not exist on type ‘CookingStep’
}
```

함수 본문에서 step.distance를 호출하고 있는데 distance는 DeliveryStep에만 존재하는 속성이다. 인자로 받는 step 타입이 CookingStep이라면 distance 속성을 찾을 수 없기 때문에 에러가 발생한다. 즉, step이라는 유니온 타입은 CookingStep 또는 DeliveryStep 타입에 해당할 뿐이지 CookingStep이면서 DeliveryStep인 것은 아니다.

<br/><br/>

> 💡 타입스크립트의 타입을 속성의 집합이 아니라 값의 집합이라고 생각해야 유니온 타입이 합집합이라는 개념을 이해할 수 있다.

<br/><br/>

### 4.1.3 교차 타입

교차 타입도 기존 타입을 합쳐 필요한 모든 기능을 가진 하나의 타입으로 만드는 것으로 이해할 수 있지만 유니온 타입과 다른 점이 있다.

<br/>

```tsx
interface CookingStep {
  orderId: string;
  time: number;
  price: number;
}

interface DeliveryStep {
  orderId: string;
  time: number;
  distance: string;
}

type BaedalProgress = CookingStep & DeliveryStep;

function logBaedalInfo(progress: BaedalProgress) {
  console.log(`주문 금액: ${progress.price}`);
  console.log(`배달 거리: ${progress.distance}`);
}
```

BaedalProgress는 CookingStep과 DeliveryStep 타입을 합쳐 모든 속성을 가진 단일 타입이 된다. 따라서 BaedalProgress 타입의 progress 값은 CookingStep이 갖고 있는 price 속성과 DeliveryStep이 갖고 있는 distance 속성을 포함하고 있다.

<br/>

앞서 유니온 타입은 합집합의 개념이라고 설명했다. 교차 타입은 교집합의 개념과 비슷하다.

<br/>

```tsx
type MyIntersection = A & B;
```

MyIntersection 타입의 모든 값은 A 타입의 값이며, MyIntersection 타입의 모든 값은 B 타입의 값이다. 집합의 관점에서 해석해보면 집합 MyIntersection의 모든 원소는 집합 A의 원소이자 집합 B의 원소임을 알 수 있다.

<br/><br/>

> 💡 다시 말하지만 타입스크립트의 타입을 속성의 집합이 아니라 값의 집합으로 이해해야 한다. BaedalProgress 교차 타입은 CookingStep이 가진 속성(orderId, time, price)과 DeliveryStep이 가진 속성(orderId, time, distance)을 모두 만족(교집합)하는 값의 타입(집합)이라고 해석할 수 있다.

<br/><br/>

```tsx
/* 배달 팁 */
interface DeliveryTip {
  tip: string;
  }
/* 별점 */
interface StarRating {
  rate: number;
}
/* 주문 필터 */
type Filter = DeliveryTip & StarRating;

const filter: Filter = {
  tip: “1000원 이하”,
  rate: 4,
};
```

교차 타입은 두 타입의 교집합을 의미한다고 했다. 그런데 DeliveryTip과 StarRating은 공통된 속성이 없는데도 Filter의 타입은 공집합(never 타입)이 아닌 DeliveryTip과 StarRating의 속성을 모두 포함한 타입이 된다. 왜냐하면 **타입이 속성이 아닌 값의 집합으로 해석되기 때문**이다. 즉, 교차 타입 Filter는 DeliveryTip의 tip 속성과 StarRating의 rate 속성을 모두 만족하는 값이 된다.

<br/><br/>

교차 타입을 사용할 때 타입이 서로 호환되지 않는 경우도 있다.
<br/>

```tsx
type IdType = string | number;
type Numeric = number | boolean;

type Universal = IdType & Numeric;
```

Universal은 IdType과 Numeric의 교차 타입이므로 두 타입을 모두 만족하는 경우에만 유지된다. 따라서 “number이면서 number인 경우”만 유효하여 Universal의 타입은 number가 된다.

<br/><br/>

### 4.1.4 extends와 교차 타입

<br/>

```tsx
interface BaseMenuItem {
  itemName: string | null;
  itemImageUrl: string | null;
  itemDiscountAmount: number;
  stock: number | null;
}

interface BaseCartItem extends BaseMenuItem {
  quantity: number;
}
```

BaseCartItem은 BaseMenuItem을 확장함으로써 BaseMenuItem의 속성을 모두 포함하고 있다. 즉, BaseCartItem은 BaseMenuItem의 속성을 모두 포함하는 상위 집합이 되고 BaseMenuItem은 BaseCartItem의 부분집합이 된다. 이를 교차 타입의 관점에서 작성하면 다음과 같다.

<br/>

```tsx
type BaseMenuItem = {
  itemName: string | null;
  itemImageUrl: string | null;
  itemDiscountAmount: number;
  stock: number | null;
};

type BaseCartItem = {
  quantity: number;
} & BaseMenuItem;

const baseCartItem: BaseCartItem = {
  itemName: “지은이네 떡볶이”,
  itemImageUrl: “https://www.woowahan.com/images/jieun-tteokbokkio.png”,
  itemDiscountAmount: 2000,
  stock: 100,
  quantity: 2,
};
```

BaseCartItem은 quantity라는 새로운 속성과 BaseMenuItem의 모든 속성을 가진 단일 타입이다. 교차 타입을 사용한 코드에서는 BaseMenuItem과 BaseCartItem을 interface가 아닌 type으로 선언했다. 왜냐하면 **유니온 타입과 교차 타입을 사용한 새로운 타입은 오직 type 키워드로만 선언할 수 있기 때문**이다.

<br/><br/>

주의할 점은 extends 키워드를 사용한 타입이 교차 타입과 100% 상응하지 않는다는 것이다.

<br/>

**extends 키워드를 사용한 예시**

```tsx
interface DeliveryTip {
  tip: number;
}

interface Filter extends DeliveryTip {
  tip: string;
  // Interface ‘Filter’ incorrectly extends interface ‘DeliveryTip’
  // Types of property ‘tip’ are incompatible
  // Type ‘string’ is not assignable to type ‘number’
}
```

🚨 DeliveryTip을 extends로 확장한 Filter 타입에 string 타입의 속성 tip을 선언하면 tip의 타입이 호환되지 않는다는 에러가 발생한다.

<br/><br/>

**교차 타입으로 작성한 예시**

```tsx
type DeliveryTip = {
  tip: number;
};

type Filter = DeliveryTip & {
  tip: string;
};
```

extends를 &로 바꿨을 뿐인데 에러가 발생하지 않는다. 이때 tip 속성의 타입은 “never”이다.

<br/>

type 키워드는 교차 타입으로 선언되었을 때 새롭게 추가되는 속성에 대해 미리 알 수 없기 때문에 선언 시 에러가 발생하지 않는다. **하지만 tip이라는 같은 속성에 대해 서로 호환되지 않는 타입이 선언되어 결국 never 타입이 된 것이다.**

<br/><br/>

### 4.1.5 배달의 민족 메뉴 시스템에 타입 확장 적용하기

배달 서비스 메뉴 예제로 간단한 타입 확장에 대해 알아봤다. 결론적으로, 주어진 타입에 무분별하게 속성을 추가하여 사용하는 것보다 타입을 확장해서 사용하는 것을 권장한다. 그렇게 하면 적절한 네이밍을 사용해서 타입의 의도를 명확히 표현할 수도 있고, 코드 작성 단계에서 예기치 못한 버그도 예방할 수 있기 때문이다.

<br/><br/>

## 4.2 타입 좁히기 - 타입 가드

타입스크립트에서 타입 좁히기는 변수 또는 표현식의 타입 범위를 더 작은 범위로 좁혀나가는 과정을 말한다. 타입 좁히기를 통해 더 정확하고 명시적인 타입 추론을 할 수 있게 되고, 복잡한 타입을 작은 범위로 축소하여 타입 안정성을 높일 수 있다.

<br/>

### 4.2.1 타입 가드에 따라 분기 처리하기

타입스크립트에서의 분기 처리는 조건문과 타입 가드를 활용하여 변수나 표현식의 타입 범위를 좁혀 다양한 상황에 따라 다른 동작으로 수행하는 것을 말한다. (\*타입 가드 : 런타임에 조건문을 사용하여 타입을 검사하고 타입 범위를 좁혀주는 기능) 다음 대화를 살펴보자.

<br/>

> 👶🏻 : 어떤 함수가 A | B 타입의 매개변수를 받을 때, 인자 타입이 A 또는 B일 때를 구분해서 로직 처리하고 싶을 땐 어떻게 해야 할까? <br/>
> 👨🏻‍🦳 : if문을 사용해서 처리하면 될 것 같은데! <br/>
> 👶🏻 : if문을 사용하면 컴파일 시 타입 정보는 모두 제거되어 런타임에 존재하지 않기 때문에 타입을 사용하여 조건을 만들 수는 없어. 컴파일해도 타입 정보가 사라지지 않는 방법을 사용해야 해. (👨🏻‍🦳: 떼잉..)

<br/>

타입에 따라 분기 처리를 하기 위해서는 특정 문맥 안에서 1) 타입스크립트가 해당 변수를 타입 A로 추론하도록 유도하면서 2) 런타임에서도 유효한 방법이 필요한데, 이때 **타입 가드**를 사용하면 된다. 타입 가드는 크게 **자바스크립트 연산자를 사용한 타입 가드**와 **사용자 정의 타입 카드**로 구분할 수 있다.

<br/><br/>

**자바스크립트 연산자를 활용한 타입 가드**

- typeof, instanceof, in과 같은 연산자를 사용해서 제어문으로 특정 타입 값을 가질 수밖에 없는 상황을 유도하여 자연스럽게 타입을 좁히는 방식
- 자바스크립트 연산자를 사용하는 이유 : 런타임에 유효한 타입 가드를 만들기 위해 (런타임에 유효하다 == TS뿐만 아니라 JS에서도 사용할 수 있는 문법이어야 한다.)
- 활용 예시 : 원시 타입을 추론할 때(typeof 연산자), 인스턴스화된 객체 타입을 판별할 때(instanceof 연산자), 객체의 속성이 있는지 없는지에 따른 구분(in 연산자)

<br/>

**사용자 정의 타입 가드**

- 사용자가 직접 어떤 타입으로 값을 좁힐지를 직접 지정하는 방식
- 활용 예시 : 타입 명제인 함수를 정의하여 사용(is 연산자)

<br/>
<br/>

### 4.2.2 원시 타입을 추론할 때: typeof 연산자 활용

typeof A === B를 조건으로 분기 처리하면, 해당 분기 내에서는 A의 타입이 B로 추론된다. 다만 typeof는 자바스크립트 타입 시스템만 대응할 수 있다. 자바스크립트의 동작 방식으로 인해 null과 배열 타입 등이 object 타입으로 판별되는 등 복잡한 타입을 검증하기에는 한계가 있기 때문에 주로 원시 타입을 좁히는 용도로만 사용할 것을 권장한다.

<br/><br/>

**typeof 연산자를 사용하여 검사할 수 있는 타입 목록**

- string
- number
- boolean
- undefined
- object
- function
- bigint
- symbol

<br/>

```tsx
const replaceHyphen: (date: string | Date) => string | Date = (date) => {
  if (typeof date === “string”) {
  // 이 분기에서는 date의 타입이 string으로 추론된다
  return date.replace(/-/g, “/”);
  }

  return date;
};
```

<br/><br/>

### 4.2.3 인스턴스화된 객체 타입을 판별할 때: instanceof 연산자 활용하기

instanceof 연산자는 인스턴스화된 객체 타입을 판별하는 타입 가드로 사용할 수 있다. A instanceof B 형태로 사용하며 A에는 타입을 검사할 대상 변수, B에는 특정 객체의 생성자가 들어간다. instanceof는 A의 프로토타입 체인에 생성자 B가 존재하는지를 검사해서 존재한다면 true, 그렇지 않다면 false를 반환한다. 이러한 동작 방식으로 인해 A의 프로토타입 속성 변화에 따라 instanceof 연산자의 결과가 달라질 수 있다는 점은 유의해야 한다.

<br/>

```tsx
const onKeyDown = (event: React.KeyboardEvent) => {
  if (event.target instanceof HTMLInputElement && event.key === “Enter”) {
  // 이 분기에서는 event.target의 타입이 HTMLInputElement이며
  // event.key가 ‘Enter’이다
  event.target.blur();
  onCTAButtonClick(event);
  }
};
```

HTMLInputElement에 존재하는 blur 메서드를 사용하기 위해, event.target이 HTMLInputElement의 인스턴스인지를 검사한 후 분기 처리하는 로직이다.

<br/><br/>

### 4.2.4 객체의 속성이 있는지 없는지에 따른 구분: in 연산자 활용하기

in 연산자는 객체에 속성이 있는지 확인한 다음에 true 또는 false를 반환한다. in 연산자를 사용하면 속성이 있는지 없는지에 따라 객체 타입을 구분할 수 있다. in 연산자는 A in B의 형태로 사용하는데 이름 그대로 A라는 속성이 B 객체에 존재하는지를 검사한다. 프로토타입 체인으로 접근할 수 있는 속성이면 전부 true를 반환한다. in 연산자는 B 객체 내부에 A 속성이 있는지 없는지를 검사하는 것이기 때문에 B 객체에 존재하는 A 속성에 undefined를 할당한다고 해서 false를 반환하는 것은 아니다. delete 연산자를 사용하여 객체 내부에서 해당 속성을 제거해야만 false를 반환한다.

<br/>

```tsx
interface BasicNoticeDialogProps {
  noticeTitle: string;
  noticeBody: string;
}

interface NoticeDialogWithCookieProps extends BasicNoticeDialogProps {
  cookieKey: string;
  noForADay?: boolean;
  neverAgain?: boolean;
}

export type NoticeDialogProps =
| BasicNoticeDialogProps
| NoticeDialogWithCookieProps;

const NoticeDialog: React.FC<NoticeDialogProps> = (props) => {
  if (“cookieKey” in props) return <NoticeDialogWithCookie {...props} />; // NoticeDialogWithCookieProps 타입
  return <NoticeDialogBase {...props} />; // BasicNoticeDialogProps 타입
};
```

NoticeDialogWithCookieProps는 BasicNoticeDialogProps를 상속받고 cookieKey 속성을 가진다. 따라서 두 객체 타입을 cookieKey 속성을 가졌는지 아닌지에 따라 in 연산자로 조건을 만들 수 있다.

<br/>

자바스크립트의 in 연산자는 런타임의 값만을 검사하지만 타입스크립트에서는 객체 타입에 속성이 존재하는지를 검사한다. 위의 코드처럼 여러 객체 타입을 유니온 타입으로 가지고 있을 때 in 연산자를 사용해서 속성의 유무에 따라 조건 분기를 할 수 있다.

<br/><br/>

### 4.2.5 is 연산자로 사용자 정의 타입 가드를 만들어 활용하기

사용자 정의 타입 가드는 반환 타입이 타입 명제(type predicates)인 함수를 정의하여 사용할 수 있다. 타입 명제는 A is B 형식으로 작성하면 되는데 여기서 A는 매개변수 이름이고 B는 타입이다. 참/거짓의 진릿값을 반환하면서 반환타입을 타입 명제로 지정하게 되면 반환 값이 참일 때 A 매개변수의 타입을 B 타입으로 취급하게 된다.

<br/>

\*타입 명제(type predicates) : 함수의 반환 타입에 대한 타입 가드를 수행하기 위해 사용되는 특별한 형태의 함수이다.

<br/>

```tsx
const isDestinationCode = (x: string): x is DestinationCode =>
  destinationCodeList.includes(x);
```

isDestinationCode는 string 타입의 매개변수가 destinationCodeList 배열의 원소 중 하나인지를 검사하여 boolean을 반환하는 함수이다. 함수의 반환값을 booelan이 아닌 x is DestinationCode로 타이핑하여 타입스크립트에게 이 함수가 사용되는 곳의 타입을 추론할 때 해당 조건을 타입 가드로 사용하도록 알려준다.

<br/><br/>

**isDestinationCode의 반환 값 타이핑을 x is DestinationCode가 아닌 boolean으로 했을 때**

```tsx
const isDestinationCode = (x: string): boolean =>
  destinationCodeList.includes(x);
```

```tsx

const getAvailableDestinationNameList = async (): Promise<DestinationName[]> => {
  const data = await AxiosRequest<string[]>(“get”, “.../destinations”);
  const destinationNames: DestinationName[] = [];
  data?.forEach((str) => {
  if (isDestinationCode(str)) { // str이 destinationCodeList의 원소가 맞는지 체크.
    destinationNames.push(DestinationNameSet[str]); // 맞다면 DestinationNames 배열에 push.
    /*
    isDestinationCode의 반환 값에 is를 사용하지 않고 boolean이라고 한다면 다음 에러가
    발생한다
    - Element implicitly has an ‘any’ type because expression of type ‘string’
    can’t be used to index type ‘Record<”MESSAGE_PLATFORM” | “COUPON_PLATFORM” | “BRAZE”,  // string[] 타입인 str을 DestinationName[]에 push할 수 없다는 에러
    “통합메시지플랫폼” | “쿠폰대장간” | “braze”>’
    */
    }
  });
  return destinationNames;
};
```

<br/>

이 경우 타입스크립트는 isDestinationCode 함수 내부에 있는 includes 함수를 해석해 타입 추론을 할수 없다. 타입스크립트는 if문 스코프의 str 타입을 좁히지 못하고 string으로만 추론한다. destinationNames의 타입은 DestinationName[]이기 때문에 string 타입의 str을 push할 수 없다는 에러가 발생한다.

<br/>

➡️ 이처럼 타입스크립트에게 반환 값에 대한 타입 정보를 알려주고 싶을 때 is를 사용할 수 있다. 반환 값의 타입을 x is DestinationCode로 알려줌으로써 타입스크립트는 if문 스코프의 str 타입을 DestinationCode로 추론할 수 있다.

<br/><br/>

## 4.3 타입 좁히기 - 식별할 수 있는 유니온(Discriminated Unions)

태그된 유니온으로도 불리는 식별할 수 있는 유니온은 타입 좁히기에 널리 사용되는 방식이다.

<br/>

### 4.3.1 에러 정의하기

배달의민족 선물하기 서비스에서는 유효성 에러가 발생하면 다양한 방식으로 에러를 보여준다. 이 에러를 크게 텍스트 에러, 토스트 에러, 얼럿 에러로 구분한다. 이들 모두 유효성 에러로 errorCode와 errorMessage를 가지고 있으며, 에러 노출 방식에 따라 추가로 필요한 정보가 있을 수 있다.

<br/>

```tsx
type TextError = {
  errorCode: string;
  errorMessage: string;
};
type ToastError = {
  errorCode: string;
  errorMessage: string;
  toastShowDuration: number; // 토스트를 띄워줄 시간
};
type AlertError = {
  errorCode: string;
  errorMessage: string;
  onConfirm: () => void; // 얼럿 창의 확인 버튼을 누른 뒤 액션
};
```

<br/>

각 에러 타입의 유니온 타입을 원소로 하는 배열을 정의해보면 다음과 같다.

```tsx
type ErrorFeedbackType = TextError | ToastError | AlertError;
const errorArr: ErrorFeedbackType[] = [
  { errorCode: “100”, errorMessage: “텍스트 에러” },
  { errorCode: “200”, errorMessage: “토스트 에러”, toastShowDuration: 3000 },
  { errorCode: “300”, errorMessage: “얼럿 에러”, onConfirm: () => {} },
];
```

ErrorFeedbackType의 원소를 갖는 배열 errorArr은 다양한 에러 객체를 관리한다. 다만, 여기서 ToastError의 특수 필드와 AlerError의 특수 필드를 모두 가지는 객체가 있다면, 타입 에러를 뱉어야할 것이다.

<br/>

```tsx
const errorArr: ErrorFeedbackType[] = [
  // ...
  {
  errorCode: “999”,
  errorMessage: “잘못된 에러”,
  toastShowDuration: 3000,
  onConfirm: () => {},
  }, // expected error
];
```

<br/>

하지만 자바스크립트는 “덕 타이핑 언어”이기 때문에 별도의 타입 에러를 뱉지 않는 것을 확인할 수 있다. 이런 상황에서 타입 에러가 발생하지 않는다면 앞으로의 개발에서 의미를 알 수 없는 에러 객체가 생겨날 위험성이 커진다.

<br/><br/>

### 4.3.2 식별할 수 있는 유니온

따라서 에러 타입을 구분할 방법이 필요하다. 각 타입이 비슷한 구조를 가지지만 서로 호환되지 않도록 만들어주기 위해서는 타입들이 서로 포함 관계를 가지지 않도록 정의해야 한다. 이때 바로 **식별할 수 있는 유니온**을 활용할 수 있다.

<br/>

💡 **식별할 수 있는 유니온
:** 타입 간의 구조 호환을 막기 위해 타입마다 구분할 수 있는 판별자를 달아주어 포함 관계를 제거 하는 것.

<br/>

판별자의 개념으로 각 에러 타입마다 errorType이라는 필드를 새로 정의해주었다.

```tsx
type TextError = {
  errorType: “TEXT”;
  errorCode: string;
  errorMessage: string;
};
type ToastError = {
  errorType: “TOAST”;
  errorCode: string;
  errorMessage: string;
  toastShowDuration: number;
}
type AlertError = {
  errorType: “ALERT”;
  errorCode: string;
  errorMessage: string;
  onConfirm: () = > void;
};
```

<br/>

각 에러 타입마다 이 필드에 대한 다른 값을 가지도록 하여 판별자를 달아주면 이들은 포함 관계를 벗어나게 된다. 새롭게 정의한 상태에서 errorArr을 새로 정의해보자.

<br/>

```tsx
type ErrorFeedbackType = TextError | ToastError | AlertError;

const errorArr: ErrorFeedbackType[] = [
  { errorType: “TEXT”, errorCode: “100”, errorMessage: “텍스트 에러” },
  {
    errorType: “TOAST”,
    errorCode: “200”,
    errorMessage: “토스트 에러”,
    toastShowDuration: 3000,
  },
  {
    errorType: “ALERT”,
    errorCode: “300”,
    errorMessage: “얼럿 에러”,
    onConfirm: () => {},
  },
  {
    errorType: “TEXT”,
    errorCode: “999”,
    errorMessage: “잘못된 에러”,
    toastShowDuration: 3000, // Object literal may only specify known properties, and ‘toastShowDuration’ does not exist in type ‘TextError’
    onConfirm: () => {},
  },
  {
    errorType: “TOAST”,
    errorCode: “210”,
    errorMessage: “토스트 에러”,
    onConfirm: () => {}, // Object literal may only specify known properties, and ‘onConfirm’ does not exist in type ‘ToastError’
  },
  {
    errorType: “ALERT”,
    errorCode: “310”,
    errorMessage: “얼럿 에러”,
    toastShowDuration: 5000, // Object literal may only specify known properties, and ‘toastShowDuration’ does not exist in type ‘AlertError’
  },
];
```

<br/>

우리가 처음 기대했던 대로 정확하지 않은 에러 객체에 대한 타입 에러가 발생하는 것을 확인할 수 있다.

<br/><br/>

### 4.2.3 식별할 수 있는 유니온의 판별자 선정

식별할 수 있는 유니온을 사용할 때 주의할 점이 있다. 식별할 수 있는 유니온의 판별자는 **유닛 타입**으로 선언되어야 정상적으로 동작한다. 유닛 타입은 다른 타입으로 쪼개지지 않고 오직 하나의 정확한 값을 가지는 타입을 말한다. null, undefined, 리터럴 타입을 비롯해 true, 1 등 정확한 값을 나타내는 타입이 유닛 타입에 해당한다. 반면 다양한 타입을 할당할 수 있는 void, string, number와 같은 타입은 유닛 타입으로 적용되지 않는다.

<br/><br/>

공식 깃허브의 [이슈 탭](https://github.com/microsoft/TypeScript/issues/30506#issuecomment-474802840)을 살펴보면 식별할 수 있는 유니온의 판별자로 사용할 수 있는 타입을 다음과 같이 정의한다.

<br/>

- 리터럴 타입이어야 한다.
- 판별자로 선정한 값에 적어도 하나 이상의 유닛 타입이 포함되어야 하며, 인스턴스화할 수 있는 타입(instantiable type)은 포함되지 않아야 한다.

1. The property is a literal type as outlined here [Discriminated union types #9163](https://github.com/microsoft/TypeScript/pull/9163)
2. The a property of a union type to be a discriminant property if it has a union type containing at least one unit type and no instantiable types as outlined here [Allow non-unit types in union discriminants #27695](https://github.com/microsoft/TypeScript/pull/27695)

<br/><br/>

## 4.4 Exhaustiveness Checking으로 정확한 타입 분기 유지하기

<br/>

Exhaustiveness Checking은 모든 케이스에 대해 철저하게 타입을 검사하는 것을 말하며 타입 좁히기에 사용되는 패러다임 중 하나이다. 모든 케이스에 대해 분기 처리해야만 유지보수 측면에서 안전하다고 생각되는 경우 Exhaustiveness Checking을 통해 모든 케이스에 대한 타입 검사를 강제할 수 있다.

<br/><br/>

### 4.4.1 상품권

선물하기 서비스에는 다양한 상품권이 있다. 상품권 가격에 따라 상품권 이름을 반환해주는 함수를 작성하면 다음과 같다.

<br/>

```tsx
type ProductPrice = “10000” | “20000”;

const getProductName = (productPrice: ProductPrice): string => {
  if (productPrice === “10000”) return “배민상품권 1만 원”;
  if (productPrice === “20000”) return “배민상품권 2만 원”;
  else {
    return “배민상품권”;
  }
};
```

<br/>

이때 새로운 상품권이 생겨 ProductPrice 타입이 업데이트되어야 한다고 해보자.

```tsx
type ProductPrice = “10000” | “20000” | “5000”;

const getProductName = (productPrice: ProductPrice): string => {
  if (productPrice === “10000”) return “배민상품권 1만 원”;
  if (productPrice === “20000”) return “배민상품권 2만 원”;
  if (productPrice === “5000”) return “배민상품권 5천 원”; // 조건 추가 필요
  else {
    return “배민상품권”;
  }
};
```

ProductPrice 타입이 업데이트되었을 때 getProductName 함수도 함께 업데이트 되었다. 그러나 getProductName 함수를 수정하지 않아도 별도 에러가 발생하는 것이 아니기 때문에 실수할 여지가 있다. 이처럼 모든 타입에 대한 타입 검사를 강제하고 시다면 다음과 같이 코드를 작성하면 된다.

<br/>

```tsx
type ProductPrice = “10000” | “20000” | “5000”;

const getProductName = (productPrice: ProductPrice): string => {
  if (productPrice === “10000”) return “배민상품권 1만 원”;
  if (productPrice === “20000”) return “배민상품권 2만 원”;
  // if (productPrice === “5000”) return “배민상품권 5천 원”;
  else {
    exhaustiveCheck(productPrice); // Error: Argument of type ‘string’ is not assignable to parameter of type ‘never’
    return “배민상품권”;
  }
};

const exhaustiveCheck = (param: never) => {
  throw new Error(“type error!”);
};
```

앞에서 추가한 productPrice가 “5000”일 때의 분기 처리가 주석된 상태이다. exhaustiveCheck(productPrice);에서 에러를 뱉고 있는데 5000이라는 값에 대한 분기 처리를 하지 않아서 (철저하게 검사하지 않았기 때문에) 발생한 것이다. 이렇게 모든 케이스에 대한 타입 분기 처리를 해주지 않았을 때, 컴파일타임 에러가 발생하게 하는 것을 Exhaustiveness Checking이라고 한다.

<br/>

exhaustiveCheck 함수를 자세히 보면, 이 함수는 매개변수를 never 타입으로 선언하고 있다. 즉, 매개변수로 그 어떤 값도 받을 수 없으며 만일 값이 들어온다면 에러를 내뱉는다. 이 함수를 타입 처리 조건문의 마지막 else 문에 사용하면 앞의 조건문에서 모든 타입에 대한 분기 처리를 강제할 수 있다.

<br/>

이렇게 Exhaustiveness Checking을 활용하면 예상치 못한 런타임 에러를 방지하거나 요구사항이 변경되었을 때 생길 수 있는 위험성을 줄일 수 있다. 타입에 대한 철저한 분기 처리가 필요하다면 Exhaustiveness Checking 패턴을 활용해보길 바란다.

<br/><br/>