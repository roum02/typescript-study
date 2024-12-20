# 8. JSX에서 TSX로

타입스크립트와 리액트를 결합하면 더 손쉽게 유지보수할 수 있는 애플리케이션을 개발할 수 있다. 이 장에서는 리액트에서 사용하는 JSX 문법을 타입스크립트에 어떻게 적용하는지를 소개한다.

<br/>

## 8.1 리액트 컴포넌트의 타입

이 챕터에서는 헷갈릴 수 있는 리액트 컴포넌트 타입을 살펴보면서 상황에 따라 어떤 것을 사용하면 좋을지 그리고 사용할 때의 유의점은 무엇인지 알아볼 예정이다.

<br/>

### 8.1.1 클래스 컴포넌트

```tsx
interface Component<P = {}, S = {}, SS = any>
  extends ComponentLifecycle<P, S, SS> {} // P는 Props, S는 State를 의미함.

class Component<P, S> {
  /* ... 생략 */
}

class PureComponent<P = {}, S = {}, SS = any> extends Component<P, S, SS> {}
```

<br/>

클래스 컴포넌트가 상속받는 React.Component와 React.PureComponent의 타입 정의는 위와 같으며 P와 S는 각각 props와 상태(state)를 의미한다. 이 예시를 보면 props와 상태 타입을 제네릭으로 받고 있다는 것을 알 수 있다. Welcome 컴포넌트의 props 타입을 지정해보면 아래와 같다. 상태가 있는 컴포넌트 일 때는 제네릭의 두번째 인자로 타입을 넘겨주면 상태에 대한 타입을 지정할 수 있다.

<br/>

```tsx
interface WelcomeProps {
  name: string;
}

class Welcome extends React.Component<WelcomeProps> {
  /* ... 생략 */
}
```

<br/><br/>

### 8.1.2 함수 컴포넌트 타입

함수 표현식을 사용하여 함수 컴포넌트를 선언할 때 가장 많이 볼 수 있는 형태는 React.FC 혹은 React.VFC로 타입을 지정하는 것이다. FC는 FunctionComponent의 약자로 React.FC와 React.VFC는 리액트에서 함수 컴포넌트의 타입 지정을 위해 제공되는 타입이다.

<br/>

```tsx
// 함수 선언을 사용한 방식
function Welcome(props: WelcomeProps): JSX.Element {}

// 함수 표현식을 사용한 방식 - React.FC 사용
const Welcome: React.FC<WelcomeProps> = ({ name }) => {};

// 함수 표현식을 사용한 방식 - React.VFC 사용 ➡️ React v18 이후 삭제됨.
const Welcome: React.VFC<WelcomeProps> = ({ name }) => {};
```

<br/>

먼저 React.FC가 등장하고 이후 @types/react 16.9.4버전에서 React.VFC 타입이 추가되었다. 둘 사이에는 children이라는 타입을 허용하는지 아닌지에 따른 차이를 보인다. React.FC는 암묵적으로 children을 포함하고 있기 때문에 해당 컴포넌트에서 children을 사용하지 않더라도 children props를 허용한다. 따라서 children props가 필요하지 않는 컴포넌트에서는 더 정확한 타입 지정을 하기 위해 React.FC보다 React.VFC를 많이 사용한다.

<br/>

```tsx
type FC<P = {}> = FunctionComponent<P>;

interface FunctionComponent<P = {}> {
  // props에 children을 추가
  (props: PropsWithChildren<P>, context?: any): ReactElement<any, any> | null;
  propTypes?: WeakValidationMap<P> | undefined;
  contextTypes?: ValidationMap<any> | undefined;
  defaultProps?: Partial<P> | undefined;
  displayName?: string | undefined;
}

type VFC<P = {}> = VoidFunctionComponent<P>;

interface VoidFunctionComponent<P = {}> {
  // children 없음
  (props: P, context?: any): ReactElement<any, any> | null;
  propTypes?: WeakValidationMap<P> | undefined;
  contextTypes?: ValidationMap<any> | undefined;
  defaultProps?: Partial<P> | undefined;
  displayName?: string | undefined;
}
```

<br/>

하지만 리액트 v18로 넘어오면서 React.VFC가 삭제되고 React.FC에서 children이 사라졌다. 그래서 **앞으로는 React.VFC 대신 React.FC 또는 props 타입 · 반환 타입을 직접 지정하는 형태로 타이핑해줘야 한다.**

<br/><br/>

### 8.1.3 Children props 타입 지정

```tsx
type PropsWithChildren<P> = P & { children?: ReactNode | undefined };
```

<br/>

가장 보편적인 children 타입은 ReactNode | undefined가 된다. ReactNode는 ReactElement 외에도 boolean, number 등 여러 타입을 포함하고 있는 타입으로, 구체적인 타이핑의 용도를 적합하지않다. 예를 들어, 특정 문자열만 허용하고 싶을 때는 children에 대해 추가로 타이핑해줘야 한다. children에 대한 타입 지정은 다른 prop 타입 지정과 동일한 방식으로 지정할 수 있다.

<br/>

```tsx
// example 1
type WelcomeProps = {
  children: "천생연분" | "더 귀한 분" | "귀한 분" | "고마운 분"; // 특정 문자열만 허용
};

// example 2
type WelcomeProps = { children: string }; // 문자열만

// example 3
type WelcomeProps = { children: ReactElement }; // JSX로 작성된 컴포넌트만
```

<br/><br/>

### 8.1.4 render 메서드와 함수 컴포넌트의 반환 타입 - React.ReactElement vs JSX.Element vs React.ReactNode

React.ReactElement, JSX.Element, React.ReactNode 타입은 쉽게 헷갈릴 수 있다.

먼저 함수 컴포넌트의 반환타입인 ReactElement의 예시이다.
<br/>

```tsx
interface ReactElement<
  P = any,
  T extends string | JSXElementConstructor<any> =
    | string
    | JSXElementConstructor<any>
> {
  type: T;
  props: P;
  key: Key | null;
}
```

<br/>

React.createElement를 호출하는 형태의 구문으로 변환하면 React.createElement의 반환 타입은 ReactElement이다. 리액트는 실제 DOM이 아니라 가상의 DOM을 기반으로 렌더링하는데 가상 DOM의 엘리먼트는 ReactElement 형태로 저장된다. 즉 ReactElement 타입은 리액트 컴포넌트를 객체 형태로 저장하기 위한 포맷이다.

<br/>

```tsx
declare global {
  namespace JSX {
    interface Element extends React.ReactElement<any, any> {}
  }
}
```

<br/>

함수 컴포넌트 예시에서 JSX.Element 타입을 사용한 것을 보고 의아했을 수도 있다. JSX.Element 타입은 앞의 코드를 보면 알 수 있다시피 리액트의 ReactElement를 확장하고 있는 타입이며, 글로벌 네임스페이스에 정의되어 있어 외부 라이브러리에서 컴포넌트 타입을 재정의할 수 있는 유연성을 제공한다. 이러한 특성으로 인해 컴포넌트 타입을 재정의하거나 변경하는 것이 용이해진다. React.Node는 다음과 같이 타입이 정의되어 있다.

<br/><br/>

> 💡 **글로벌 네임스페이스(Global Namespace)** <br/>
> 프로그래밍에서 식별자(변수, 함수, 타입 등)가 정의되는 전역적인 범위를 말한다. 자바스크립트나 타입스크립트에서는 기본적으로 전역(글로벌) 스코프에서 선언된 변수나 함수 등은 글로벌 네임스페이스에 속한다. 즉, 어떤 파일이든지 해당 스코프에서 선언된 식별자는 모든 곳에서 접근할 수 있다.

<br/>

React.Node

```tsx
type ReactText = string | number;
type ReactChild = ReactElement | ReactText;
type ReactFragment = {} | Iterable<ReactNode>;

type ReactNode =
  | ReactChild
  | ReactFragment
  | ReactPortal
  | boolean
  | null
  | undefined;
```

<br/>

![image](https://github.com/woowa-typescript/woowahan-typescript-with-react-example-code/assets/102462534/ab9b7a8e-f583-4e70-8e1c-52e36b7ed161)

_출처 - [yoonvelog](https://velog.io/@yoonvelog/ReactNode-ReactElement-JSXElement)_

<br/>

ReactNode는 React 컴포넌트의 렌더링 결과물로 나오는 모든 종류의 Node를 나타내는 타입이다. 이는 문자열, 숫자, 또는 ReactElement와 같은 JSX Element 등 다양한 종류의 값을 포함한다.

<br/>

<details>
<summary>
🚩 ReactElement vs ReactNode vs JSX.Element [ChatGPT]</summary>

JSX의 반환 타입인 `ReactElement`, `ReactNode`, `JSX.Element`에 대해 자세히 설명드리겠습니다. 이 타입들은 React와 TypeScript에서 JSX를 다룰 때 중요한 역할을 합니다.

  <br/>

### **1. `ReactElement`**

- **정의**: `ReactElement`는 React 요소를 나타내는 인터페이스입니다. React 요소는 React 컴포넌트의 가장 작은 단위로, 화면에 렌더링 될 수 있는 것을 의미합니다.
- **사용**: `ReactElement`는 주로 React 컴포넌트의 반환 타입으로 사용됩니다. 예를 들어, 함수형 컴포넌트나 클래스 컴포넌트에서 `render` 메서드는 `ReactElement`를 반환합니다.
- **예시**:

  ```tsx
  tsxCopy code
  const MyComponent: React.FC = (): React.ReactElement => {
    return <div>Hello World</div>;
  };

  ```

  <br/>

### **2. `ReactNode`**

- **정의**: `ReactNode`는 `ReactElement`보다 더 넓은 범위의 타입입니다. `ReactNode`는 `ReactElement`, 문자열, 숫자, 배열, fragment, boolean 등을 포함할 수 있습니다. 즉, React 컴포넌트가 반환할 수 있는 거의 모든 것을 포함합니다.
- **사용**: `ReactNode`는 컴포넌트가 다양한 타입의 자식을 반환할 수 있을 때 사용됩니다.
- **예시**:

  ```tsx
  tsxCopy code
  const MyComponent: React.FC = (): React.ReactNode => {
    return <div>{someCondition ? "Text" : <OtherComponent />}</div>;
  };

  ```

  <br/>

### **3. `JSX.Element`**

- **정의**: `JSX.Element`는 TypeScript가 JSX를 사용할 때 정의하는 타입입니다. 이는 기본적으로 `ReactElement`와 동일하지만, JSX 문법을 사용하는 경우에 한해 적용됩니다.
- **사용**: 주로 JSX 문법을 사용하는 함수나 컴포넌트의 반환 타입으로 사용됩니다.
- **예시**:

  ```tsx
  tsxCopy code
  const MyComponent = (): JSX.Element => {
    return <div>Hello World</div>;
  };

  ```

  <br/>

### **요약**

- `ReactElement`는 React 컴포넌트가 반환할 수 있는 가장 기본적인 타입입니다.
- `ReactNode`는 `ReactElement`보다 더 넓은 범위의 타입을 포함합니다.
- `JSX.Element`는 JSX 문법을 사용하는 경우에 한정된 `ReactElement`와 같은 타입입니다.

  <br/>

</details>

<br/><br/>

### 8.1.5 ReactElement, ReactNode, JSX.Element 활용하기

ReactElement, ReactNode, JSX.Element는 모두 리액트의 요소를 나타내는 타입이다. 이 절에서는 이 3가지 타입의 차이점과 어떤 상황에서 어떤 타입을 사용해야 더 좋은 코드를 작성할 수 있는지를 소개한다. 먼저 @types/react 패키지에 정의된 타입을 살펴보면 아래와 같다.

<br/>

<details>
<summary>@types/react 패키지에 정의된 타입</summary>

```tsx
declare namespace React {
  // ReactElement
  interface ReactElement<
    P = any,
    T extends string | JSXElementConstructor<any> =
      | string
      | JSXElementConstructor<any>
  > {
    type: T;
    props: P;
    key: Key | null;
  }

  // ReactNode
  type ReactText = string | number;
  type ReactChild = ReactElement | ReactText;
  type ReactFragment = {} | Iterable<ReactNode>;

  type ReactNode =
    | ReactChild
    | ReactFragment
    | ReactPortal
    | boolean
    | null
    | undefined;
  type ComponentType<P = {}> = ComponentClass<P> | FunctionComponent<P>;
}

// JSX.Element
declare global {
  namespace JSX {
    interface Element extends React.ReactElement<any, any> {
      // ...
    }
    // ...
  }
}
```

</details>

<br/>

### **ReactElement**

createElement 메서드는 리액트 엘리먼트를 생성하는 메서드이다. 리액트를 사용하면서 JSX라는 자바스크립트를 확장한 문법을 자주 접했을 텐데 JSX는 createElement 메서드를 호출하기 위한 문법이다.

<br/>

> 💡 **JSX** <br/>
> JSX는 자바스크립트의 확장 문법으로 리액트에서 UI를 표현하는 데 사용된다. XML과 비슷한 구조로 되어 있으며 리액트 컴포넌트를 선언하고 사용할 때 더욱 간결하고 가독성 있게 코드를 작성할 수 있도록 도와준다. 또한 HTML과 유사한 문법을 제공하여 리액트 사용자에게 렌더링 로직(마크업)을 쉽게 만들 수 있게 해주고, 컴포넌트 구조와 계층 구조를 편리하게 표현할 수 있도록 해준다.

<br/>

즉, JSX는 리액트 엘리먼트를 생성하기 위한 문법이며 **트랜스파일러는 JSX 문법을 createElement 메서드 호출문으로 변환하여 아래와 같이 리액트 엘리먼트를 생성한다.**

<br/>

```tsx
const element = React.createElement(
  // ReactElement 생성하는 메서드
  "h1", // 생성할 DOM 태그의 타입 == <h1> 태그
  { className: "greeting" }, // 해당 요소의 props > greeting이란 클래스
  "Hello, world!" // 자식 요소  > 해당 텍스트를 자식으로 가짐.
);

// 주의: 다음 구조는 단순화되었다
const element = {
  // ReactElement의 구조를 단순화함
  type: "h1", // element의 타입. 여기서는 h1
  props: {
    className: "greeting",
    children: "Hello, world!",
  },
};

declare global {
  // 전역적으로 타입을 선언함
  namespace JSX {
    // JSX와 관련된 타입을 정의하는 네임스페이스
    interface Element extends React.ReactElement<any, any> {
      // ...     // JSX.Element는 React.ReactElement를 확장함.
      // JSX 문법을 사용할 때 반환되는 요소의 타입을 정의한 것.
    }
    // ...
  }
}
```

<br/>

리액트는 이런 식으로 만들어진 리액트 엘리먼트 객체를 만들어서 DOM을 구성한다. 리액트에는 여러 개의 createElement 오버라이딩 메서드가 존재하는데, 이 메서드들이 반환하는 타입은 ReactElement 타입을 기반으로 한다.

정리하면 ReactElement 타입은 JSX의 createElement 메서드 호출로 생성된 리액트 엘리먼트를 나타내는 타입이라고 볼 수 있다.

<br/><br/>

### **ReactNode**

ReactNode 타입에 알아보기 전 먼저 ReactChild 타입을 살펴보자.

<br/>

```tsx
type ReactText = string | number;
type ReactChild = ReactElement | ReactText;
```

<br/>

ReactChild 타입은 ReactElement | string | number로 정의되어 ReactElement 보다는 좀 더 넓은 범위를 갖고 있다. 그럼 ReactNode 타입을 살펴보자.

<br/>

```tsx
type ReactFragment = {} | Iterable<ReactNode>; // ReactNode의 배열 형태
type ReactNode =
  | ReactChild
  | ReactFragment
  | ReactPortal
  | boolean
  | null
  | undefined;
```

<br/>

ReactNode는 앞에서 설명한 ReactChild 외에도 boolean, null, undefined 등 훨씬 넓은 범주의 타입을 포함한다. 즉, ReactNode는 리액트의 render 함수가 반환할 수 있는 모든 형태를 담고 있다고 볼 수 있다.

<br/><br/>

### **JSX.Element**

```tsx
declare global {
  namespace JSX {
    interface Element extends React.ReactElement<any, any> {
      // ...
    }
    // ...
  }
}
```

<br/>

JSX.Element는 ReactElement의 제네릭으로 props와 타입 필드에 대해 any 타입을 가지도록 확장하고 있다. 즉, JSX.Element는 ReactElement의 특정 타입으로 props와 타입 필드를 any로 가지는 타입이라는 것을 알 수 있다.

<br/>

### 8.1.6 사용 예시

ReactElement, ReactNode, JSX.Element 3가지 타입의 공통점은 모두 리액트에서 제공하는 컴포넌트를 나타낸다는 것이다. 그럼 어떤 상황에 어떤 타입을 사용하는 게 좋을까?

<br/>

### ReactNode

ReactNode 타입은 앞서 언급한 대로 리액트의 render 함수가 반환할 수 있는 모든 형태를 담고 있기 때문에 리액트 컴포넌트가 가질 수 있는 모든 타입을 의미한다. 리액트의 Composition(합성) 모델을 활용하기 위해 prop으로 children을 많이 사용해봤을 것이다. children을 포함하는 props 타입을 선언하면 다음과 같다.

<br/>

```tsx
interface MyComponentProps {
  children?: React.ReactNode;
  // ...
}
```

<br/>

JSX 형태의 문법을 때로는 string, number, null, undefined 같이 어떤 타입이든 children prop으로 지정할 수 있게 하고 싶다면 ReactNode 타입으로 children을 선언하면 된다.

<br/>

❕ 리액트 내장 타입인 PropsWithChildren 타입도 ReactNode 타입으로 children을 선언하고 있다.

<br/>

```tsx
type PropsWithChildren<P = unknown> = P & {
  // P의 타입이 지정되어있지 않다면 unknown 타입
  children?: ReactNode | undefined;
};

interface MyProps {
  // ...
}

type MyComponentProps = PropsWithChildren<MyProps>; // MyProps와 children 속성을 결합한 새로운 타입
// MyProps 인터페이스에 정의된 모든 속성과 children 속성포함
```

<br/>

예제 코드를 보면, MyComponent는 다양한 타입의 children을 받을 수 있으며, MyProps 인터페이스에 정의된 추가 속성들도 함께 사용할 수 있다. 이런 식으로 ReactNode는 prop으로 리액트 컴포넌트가 다양한 형태를 가질 수 있게 하고싶을 때 유용하게 사용된다.

<br/><br/>

### JSX.Element

JSX.Element는 앞서 언급한 대로 props와 타입 필드가 any 타입인 리액트 엘리먼트를 나타낸다. 이러한 특성 때문에 리액트 엘리먼트를 prop으로 전달 받아 render props 패턴으로 컴포넌트를 구현할 때 유용하게 활용할 수 있다.

<br/>

```tsx
// JSX.Element 타입을 사용하여 React 컴포넌트에서 자식 컴포넌트를 props로 전달하고,
// 그 자식 컴포넌트의 props에 접근하는 방법을 보여주는 예제.

interface Props {
  // icon의 타입은 JSX.Element
  icon: JSX.Element;
}

const Item = ({ icon }: Props) => {
  // icon을 prop으로 받음.
  // prop으로 받은 컴포넌트의 props에 접근할 수 있다
  const iconSize = icon.props.size; // icon의 props에 접근하여 size 속성을 가져옴.
  // icon이 JSX 요소이기 때문에 가능함.

  return <li>{icon}</li>;
};

// icon prop에는 JSX.Element 타입을 가진 요소만 할당할 수 있다
const App = () => {
  return <Item icon={<Icon size={14} />} />; // Icon컴포넌트는 JSX.Element 타입의 요소
};
```

<br/>

icon prop을 JSX.Element 타입으로 선언함으로써 해당 prop에는 JSX 문법만 삽입할 수 있다. 또한 icon.props에 접근하여 prop으로 넘겨받은 컴포넌트의 상세한 데이터를 가져올 수 있다. 이 예제에서 중요한 점은 JSX.Element 타입의 요소가 전달되며, 이 요소의 props에 직접 접근하여 특정 속성을 사용할 수 있다는 것이다.

<br/><br/>

### ReactElement

JSX.Element 예시를 확장하여 추론 관점에서 더 유용하게 활용할 수 있는 방법은 JSX.Element 대신에 ReactElement를 사용하는 것이다. 이때 원하는 컴포넌트의 props를 ReactElement의 제네릭으로 지정해줄 수 있다. 만약 JSX.Element가 ReactElement의 props 타입으로 any가 지정되었다면, ReactElement 타입을 활용하여 제네릭에 직접 해당 컴포넌트의 props 타입을 명시해준다.

<br/>

```tsx
interface IconProps {
  size: number;
}

interface Props {
  // Item 컴포넌트의 props를 정의.
  // ReactElement의 props 타입으로 IconProps 타입 지정
  icon: React.ReactElement<IconProps>; // icon의 props의 타입은 IconProps
  // icon은 IconProps 타입의 props를 갖는 ReactElement임을 명시.
}

const Item = ({ icon }: Props) => {
  //icon을 props로 받음
  // icon prop으로 받은 컴포넌트의 props에 접근하면, props의 목록이 추론된다
  const iconSize = icon.props.size; // icon의 props에 접근.

  return <li>{icon}</li>;
};
```

<br/>

이처럼 코드를 작성하면 icon.props에 접근할 때 어떤 prop가 있는지가 추론되어 IDE에 표시되는 것을 확인할 수 있다.

<br/><br/>

### 8.1.7 리액트에서 기본 HTML 요소 타입 활용하기

리액트를 사용하면서 HTML button 태그를 확장한 Button 컴포넌트를 만들어보자.
<br/>

```tsx
const SquareButton = () => <button>정사각형 버튼</button>;
```

<br/>

이 SquareButton은 기존 HTML button과 같은 역할을 하면서도 새로운 기능이나 UI가 추가된 형태이다. 기존의 button 태그가 클릭 이벤트를 등록하기 위한 onClick 이벤트 핸들러를 지원하는 것처럼, 새롭게 만든 Button 컴포넌트도 onClick 이벤트 핸들러를 지원해야만 일관성과 편의성을 모두 챙길 수 있다. 이 절에서는 기존 HTML 태그의 속성 타입을 활용하여 타입을 지정하는 방법에 대해 알아보자.

<br/>

### DetailedHTMLProps와 ComponentPropsWithoutRef

HTML 태그의 속성 타입을 활용하는 대표적인 2가지 방법은 리액트의 DetailedHTMLProps와 ComponentPropsWithoutRef 타입을 활용하는 것이다.

<br/>

- `DetailedHTMLProps`는 특정 HTML 요소의 모든 속성(기본 HTML 속성 및 React 추가 속성)을 포함하는 타입을 정의하는 데 사용된다. 이를 통해 특정 HTML 요소의 타입 안전성을 보장할 수 있다.
- `ComponentPropsWithoutRef`는 `ref` 속성을 포함하지 않는 컴포넌트의 타입을 정의할 때 사용된다. 이는 주로 `forwardRef`를 사용하지 않는 컴포넌트에 적합하다.

먼저 React.DetailedHTMLProps를 활용하는 경우에는 아래와 같이 쉽게 HTML 태그 속성과 호환되는 타입을 선언할 수 있다.

<br/>

```tsx
type NativeButtonProps = React.DetailedHTMLProps<
  // <button> HTML 요소의 모든 속성 포함.
  React.ButtonHTMLAttributes<HTMLButtonElement>, //<button> 요소에 특정된 HTML 속성
  HTMLButtonElement // 버튼 요소 자체의 특성
>;

type ButtonProps = {
  // NatvieButtonProps에서 onClick 속성만 추출함.
  onClick?: NativeButtonProps["onClick"];
};
```

<br/>

❕ ButtomProps의 onClick 타입은 실제 HTML button 태그의 onClick 이벤트 핸들러 타입과 동일하게 할당되는 것을 확인할 수 있다.

<br/>

그리고 React.ComponentPropsWithoutRef 타입은 아래와 같이 활용할 수 있다.

<br/>

```tsx
type NativeButtonType = React.ComponentPropsWithoutRef<"button">;
// HTML <button> 요소의 모든 props를 가져오지만, ref 속성은 제외함.
type ButtonProps = {
  onClick?: NativeButtonType["onClick"]; // onClick 속성만 추출하여 정의.
};
```

<br/>

마찬가지로 리액트의 button onClick 이벤트 핸들러에 대한 타입이 할당된 것을 볼 수 있다.

<br/><br/>

### 언제 ComponentPropsWithoutRef를 사용하면 좋을까

이외에도 HTMLProps, ComponentPropsWithRef 등 HTML 태그의 속성을 지원하기 위한 다양한 타입이 있다. 컴포넌트의 props로서 HTML 태그 속성을 확장하고 싶을 때의 상황이라고 가정해보자.

<br/>

HTML button 태그와 동일한 역할을 하지만 커스텀 UI를 적용하여 재사용성을 높이기 위한 Button 컴포넌트를 만들어보자.

<br/>

```tsx
const Button = () => {
  return <button>버튼</button>;
};
```

<br/>

먼저 HTML button 태그를 대체하는 역할이므로 아래와 같이 기존 button 태그의 HTML 속성을 props로 받을 수 있게 지원해야 할 것이다.

<br/>

```tsx
type NativeButtonProps = React.DetailedHTMLProps<
  React.ButtonHTMLAttributes<HTMLButtonElement>,
  HTMLButtonElement
>;

const Button = (props: NativeButtonProps) => {
  return <button {...props}>버튼</button>;
};
```

<br/>

여기까지 보면 HTMLButtonElement의 속성을 모두 props로 받아 button 태그에 전달했으므로 문제 없어 보인다. 그러나 ref를 props로 받을 경우 고려해야 할 사항이 있다.

<br/>

> 💡 **ref** <br/>
> 생성된 DOM 노드나 리액트 엘리먼트에 접근하는 방법으로 아래와 같이 사용된다.

```tsx
// 클래스 컴포넌트
class Button extends React.Component {
  constructor(props) {
    super(props);
    this.buttonRef = React.createRef();
  }

  render() {
    return <button ref={this.buttonRef}>버튼</button>;
  }
}

// 함수 컴포넌트
function Button(props) {
  const buttonRef = useRef(null);

  return <button ref={buttonRef}>버튼</button>;
}
```

<br/>

컴포넌트 내에서 ref를 활용해 생성된 DOM 노드에 접근하는 것과 마찬가지로, 재사용할 수 있는 Button 컴포넌트 역시 props로 전달된 ref를 통해 button 태그에 접근하여 DOM 노드를 조작할 수 있을 것으로 예상된다.

<br/>

```tsx
type NativeButtonProps = React.DetailedHTMLProps<
  React.ButtonHTMLAttributes<HTMLButtonElement>,
  HTMLButtonElement
>;

// 클래스 컴포넌트
class Button extends React.Component {
  constructor(ref: NativeButtonProps["ref"]) {
    this.buttonRef = ref;
  }

  render() {
    return <button ref={this.buttonRef}>버튼</button>;
  }
}

// 함수 컴포넌트
function Button(ref: NativeButtonProps["ref"]) {
  const buttonRef = useRef(null);

  return <button ref={buttonRef}>버튼</button>;
}
```

<br/>

여기서 주목해야 할 점은 클래스 컴포넌트와 함수 컴포넌트에서 ref를 props로 받아 전달하는 방식에 차이가 있다는 것이다.

<br/>

```tsx
// 클래스 컴포넌트로 만들어진 Button 컴포넌트를 사용할 때
class WrappedButton extends React.Component {
  constructor() {
    this.buttonRef = React.createRef();
  }

  render() {
    return (
      <div>
        <Button ref={this.buttonRef} />
      </div>
    );
  }
}
```

<br/>

클래스 컴포넌트로 마들어진 버튼은 컴포넌트 props로 전달된 ref가 Button 컴포넌트의 button 태그를 그대로 바라보게 된다.

<br/>

```tsx
// 함수 컴포넌트로 만들어진 Button 컴포넌트를 사용할 때
const WrappedButton = () => {
  const buttonRef = useRef();

  return (
    <div>
      <Button ref={buttonRef} />{" "}
    </div>
  );
};
```

<br/>

하지만 함수 컴포넌트의 경우 기대와 달리 전달받은 ref가 Button 컴포넌트의 button 태그를 바라보지 않는다. 클래스 컴포넌트에서 ref 객체는 마운트된 컴포넌트의 인스턴스를 current 속성값으로 가지지만, 함수 컴포넌트에서는 생성된 인스턴스가 없기 때문에 ref에 기대한 값이 할당되지 않는 것이다.

이러한 제약을 극복하고 함수 컴포넌트에서도 ref를 전달받을 수 있도록 도와주는 것이 React.forwardRef 메서드이다.

<br/>

```tsx
// forwardRef를 사용해 ref를 전달받을 수 있도록 구현
const Button = forwardRef((props, ref) => {
  return (
    <button ref={ref} {...props}>
      버튼
    </button>
  );
});

// buttonRef가 Button 컴포넌트의 button 태그를 바라볼 수 있다
const WrappedButton = () => {
  const buttonRef = useRef();

  return (
    <div>
      <Button ref={buttonRef} />
    </div>
  );
```

<br/>

forwardRef는 2개의 제네릭 인자를 받을 수 있는데, 첫 번째는 ref에 대한 타입 정보이며 두번째는 props에 대한 타입 정보이다. 그렇다면 앞 코드에서 Button 컴포넌트에 대한 forwardRef의 타입 선언은 어떻게 할까?

<br/>

```tsx
type NativeButtonType = React.ComponentPropsWithoutRef<"button">;

// forwardRef의 제네릭 인자를 통해 ref에 대한 타입으로 HTMLButtonElement를, props에 대한 타입으로 NativeButtonType을 정의했다
const Button = forwardRef<HTMLButtonElement, NativeButtonType>((props, ref) => {
  return (
    <button ref={ref} {...props}>
      버튼
    </button>
  );
});
```

<br/>

앞의 코드를 보면 Button 컴포넌트의 props에 대한 타입인 NativeButtonType을 정의할 때 ComponentPropsWithoutRef 타입을 사용한 것을 알 수 있다. 이렇게 타입을 React.ComponentPropsWithoutRef<"button">로 작성하면 button 태그에 대한 HTML 속성을 모두 포함하지만, ref 속성은 제외된다. 이러한 특징 때문에 DetailedHTMLProps, HTMLProps, ComponentPropsWithRef와 같이 ref 속성을 포함하는 타입과는 다르다.

함수 컴포넌트의 props로 DetailedHTMLProps와 같이 ref를 포함하는 타입을 사용하게 되면, **실제로는 동작하지 않는 ref를 받도록 타입이 지정되어 예기치 않은 에러가 발생할 수 있다.** 따라서 HTML 속성을 확장하는 props를 설계할 때는 ComponentPropsWithoutRef 타입을 사용하여 ref가 실제로 forwardRef와 함께 사용될 때만 props로 전달되도록 타입을 정의하는 것이 안전하다.

<br/><br/>

## 8.2 타입스크립트로 리액트 컴포넌트 만들기

타입스크립트는 리액트 프로젝트에서 공통 컴포넌트에 어떤 타입의 속성(프로퍼티)이 제공되어야 하는지 알려준다. 또한 필수로 전달되어야 하는 속성이 전달되지 않았을 때는 에러를 표시해준다.

<br/>

### 8.2.1 JSX로 구현된 Select 컴포넌트

<br/>

```tsx
const Select = ({ onChange, options, selectedOption }) => {
  const handleChange = (e) => {
    const selected = Object.entries(options).find(
      ([_, value]) => value === e.target.value
    )?.[0];
    onChange?.(selected);
  };

  return (
    <select
      onChange={handleChange}
      value={selectedOption && options[selectedOption]}
    >
      {Object.entries(options).map(([key, value]) => (
        <option key={key} value={value}>
          {value}
        </option>
      ))}
    </select>
  );
};
```

<br/>

이 컴포넌트는 각 Option의 키-값 쌍을 객체로 받고 있으며, 선택된 값이 변경될 때 호출되는 onChange 이벤트 핸들러를 받도록 구현되어 있다. 현재 코드는 각 속성에 어떤 타입의 값을 전달해야 할지 알기 어렵기 때문에 컴포넌트를 사용하는 개발자가 각 속성에 어떤 타입의 값을 전달해야 할지 명확하게 알 수 있도록 추가적인 설명이 필요하다.

<br/><br/>

### 8.2.2 JSDocs로 일부 타입 지정하기

컴포넌트의 속성 타입을 명시하기 위해 JSDocs를 사용할 수 있다. JSDocs를 활용하면 컴포넌트에 대한 설명과 각 속성이 어떤 역할을 하는지 간단하게 알려줄 수 있다.

<br/>

```tsx
/**
* Select 컴포넌트
* @param {Object} props - Select 컴포넌트로 넘겨주는 속성
* @param {Object} props.options - { [key: string]: string } 형식으로 이루어진 option 객체
* @param {string | undefined} props.selectedOption - 현재 선택된 option의 key값 (optional)
* @param {function} props.onChange - select 값이 변경되었을 때 불리는 callBack 함수 (optional)
* @returns {JSX.Element}
*/
const Select = //...
```

<br/><br/>

### 8.2.3 props 인터페이스 적용하기

JSDocs를 활용하면 각 속성의 대략적인 타입과 어떤 역할을 하는지 파악할 수 있지만 options가 어떤 형식의 객체를 나타내는지 onChange의 매개변수 및 반환 값에 대한 구체적인 정보를 알기는 쉽지 않다. 타입스크립트를 사용하면 좀 더 구체적인 타입 지정이 가능하다.

<br/>

```tsx
type Option = Record<string, string>; // {[key: string]: string}
// 키와 값 모두 문자열

interface SelectProps {
  options: Option;
  selectedOption?: string;
  onChange?: (selected?: string) => void;
  //string 혹은 undefined를 매개변수로 받음. 어떤 값도 반환하지 않음.
  // 이 핸들러는 옵셔널 프로퍼티이므로, 부모 컴포넌트에서 제공하지 않아도 됨.
}

const Select = ({
  options,
  selectedOption,
  onChange,
}: SelectProps): JSX.Element => {
  //...
};
```

<br/>

❕ [key: string]은 사실상 모든 키값을 가질 수 있음을 나타낸다. 하지만 넓은 범위의 타입은 해당 타입을 사용하는 함수에 잘못된 타입이 전달되기 때문에, 가능한 한 타입을 좁게 제한하여 사용하는 것을 권장한다.
( 모든 문자열 키값을 허용하는 것이 아니라, 구체적인 키를 명시하거나 키의 값을 제한하는 것을 권장한다.)

<br/>

```tsx
interface Fruit {
  count: number;
}

interface Param {
  [key: string]: Fruit; // type Param = Record<string, Fruit>과 동일
}

// Param 타입의 객체를 매개변수로 받음.
// 객체분해할당을 사용하여 Param 객체 내의 apple 속성에 접근함.
const func: (fruits: Param) => void = ({ apple }: Param) => {
  console.log(apple.count);
};

// OK.
func({ apple: { count: 0 } });

// Runtime Error (Cannot read properties of undefined (reading 'count'))
func({ mango: { count: 0 } });
// func 함수가 apple에 접근을 시도하지만 전달된 객체에 apple은 없음.
// undefined의 count 속성에 접근하려고 하여 오류가 발생함.
```

<br/><br/>

### 8.2.4 리액트 이벤트

리액트는 가상 DOM을 다루면서 이벤트도 별도로 관리한다. onclick, onchange같이 DOM 엘리먼트에 등록되어 처리하는 이벤트 리스너와 달리, 리액트 컴포넌트(노드)에 등록되는 이벤트 리스너는 onClick, onChange처럼 카멜 케이스로 표기한다. 따라서 리액트 이벤트는 브라우저의 고유한 이벤트와 완전히 동일하게 동작하지 않는다.

<br/>

**리액트 이벤트 처리의 특징**

1. 카멜 케이스 표기 : 이벤트 핸들러 이름을 카멜 케이스로 작성한다.
2. 이벤트 버블링 : 기본적으로 이벤트 버블링 단계에서 이벤트 핸들러가 호출된다. 캡처 단계에서 이벤트 핸들러를 호출하려면, `Capture`를 추가한다. (ex. onClickCapture)
3. 합성 이벤트(SyntheticEvent) : 리액트는 브라우저의 기본 이벤트를 감싸는 합성 이벤트를 제공한다. 이는 브라우저 간 일관된 이벤트 객체를 제공하기 위함이다.

<br/><br/>

**이벤트 핸들러 타입**

```tsx
type EventHandler<Event extends React.SyntheticEvent> = (
  e: Event
) => void | null;

type ChangeEventHandler = EventHandler<ChangeEvent<HTMLSelectElement>>;
```

<br/>

이 코드에서 EventHandler는 리액트의 합성 이벤트를 매개변수로 받는 함수 타입이다. ChangeEventHandler는 HTMLSelectElement의 ChangeEvent에 대한 이벤트 핸들러 타입을 정의한다.

<br/><br/>

**일반 이벤트 vs 리액트 합성이벤트**

```tsx
const eventHandler1: GlobalEventHandlers["onchange"] = (e) => {
  e.target; // 일반 Event는 target이 없음
};

const eventHandler2: ChangeEventHandler = (e) => {
  e.target; // 리액트 이벤트(합성 이벤트)는 target이 있음
};
```

<br/>

eventHandler1은 일반적인 Javascript 이벤트 핸들러로, e.target은 정의되지 않을 수도 있다. 반면, eventHandler2는 리액트의 합성 이벤트 핸들러로 e.target을 안전하게 사용할 수 있다.

<br/><br/>

**리액트 Select 컴포넌트 예시**

```tsx
const Select = ({ onChange, options, selectedOption }: SelectProps) => {
  const handleChange: React.ChangeEventHandler<HTMLSelectElement> = (e) => {
    const selected = Object.entries(options).find(
      ([_, value]) => value === e.target.value
    )?.[0];
    onChange?.(selected); // selected는 string | undefined
  };

  return <select onChange={handleChange}>{/* ... */}</select>;
};
```

<br/>

이 Select 컴포넌트는 options 객체에서 사용자가 선택한 옵션을 찾아 onChange 핸들러에 전달한다. handleChange는 HTMLSelectElement에 대한 리액트 합성 이벤트 핸들러로, e.target.value를 사용하여 선택된 값을 찾는다.

<br/><br/>

### 8.2.5 훅에 타입 추가하기

```tsx
const fruits = { apple: "사과", banana: "바나나", blueberry: "블루베리" };

const FruitSelect: VFC = () => {
  const [fruit, changeFruit] = useState<string | undefined>();

  return (
    <Select onChange={changeFruit} options={fruits} selectedOption={fruit} />
  );
};
```

<br/>

Select 컴포넌트를 사용하여 과일을 선택할 수 있는 컴포넌트이다. useState 같은 함수 역시 타입 매개변수를 지정해줌으로써 반환되는 state 타입을 지정해줄 수 있다. 만약 제네릭 타입을 명시하지 않으면 타입 스크립트 컴파일러는 초깃값(default value)의 타입을 기반으로 state 타입을 추론한다.

만약 타입 매개변수가 없다면 fruit의 타입이 undefined로 추론되면서 onChange의 타입과 일치하지 않아 오류가 발생한다.

<br/>

```tsx
// fruit: undefined;
// changeFruit: (v: React.SetStateAction<undefined>) => void;
const [fruit, changeFruit] = useState();
// 초기값이 제공되지 않았으므로, fruit의 타입은 undefined로 추론됨.
// 즉, changeFruit에는 undefined만 매개변수로 넘길 수 있게 됨

return (
  <Select
    // Error - SetStateAction<undefined>와 맞지 않음
    // (changeFruit에는 undefined만 매개변수로 넘길 수 있음)
    onChange={changeFruit} // string 값을 받는 함수를 원함.
    options={fruits}
    selectedOption={fruit}
  />
);
```

<br/>

```tsx
const fruits = { apple: "사과", banana: "바나나", blueberry: "블루베리" };
const [fruit, changeFruit] = useState("apple");

// error가 아님
const func = () => {
  changeFruit("orange");
};
```

<br/>

fruit가 반드시 apple, banana, blueberry 중 하나라고 기대하고 작성한 코드이다. 하지만 useState에 제네릭 타입을 지정해주지 않았기 때문에 타입스크립트 컴파일러는 fruit를 string으로 추론할 것이고, 다음에 다른 개발자가 changeFruit에 fruit 타입에 속하지 않는 orange를 넣을 수도 있다. 컴파일러는 이를 에러로 잡지 않아 예상치 못한 사이드 이펙트가 발생할 수도 있다.

 <br/>

> 💡 **사이드 이펙트 (Side Effect)** <br/>
> 프로그램의 실행 결과가 예상치 못한 상태로 변경되거나 예상치 못한 동작을 하게 되는 상황을 가리킨다. 즉, 코드의 실행이 예상과는 다르게 동작하여 예상치 못한 결과를 초래하는 것을 의미한다.

<br/>

```tsx
const fruits = { apple: "사과", banana: "바나나", blueberry: "블루베리" };
type Fruit = keyof typeof fruits; // 'apple' | 'banana' | 'blueberry';
const [fruit, changeFruit] = useState<Fruit | undefined>("apple");

// 에러 발생
const func = () => {
  changeFruit("orange");
};
```

<br/>

이때 타입 매개변수로 좀 더 명확한 타입을 지정함으로써, 다른 개발자가 해당 state나, changeState를 한정된 타입으로만 다룰 수 있게 강제할 수 있다. keyof typeof fruits를 사용하여 fruits 키값만 추출해서 Fruit라는 타입을 새로 만들었다. 그리고 이렇게 정의된 Fruit 타입을 useState의 제네릭으로 활용하여 changeFruit에 ‘apple, banana, blueberry, undefined’를 제외한 다른 값이 할당되면 에러가 발생하도록 설정되었다.

keyof typeof obj는 해당 객체의 키값을 유니온 타입으로 추출하는 패턴으로 자주 사용된다. 이처럼 훅이나 외부 라이브러리 또는 내부 모듈의 함수는 적절한 제네릭 타입을 설정하여 활용할 수 있다.

<br/>

❕ string, number, boolean 같은 원시 타입은 자동으로 추론되므로 생략할 수 있다.

<br/><br/>

### 8.2.6 제네릭 컴포넌트 만들기

```tsx
const FruitSelect = () => {
  const [fruit, changeFruit] = useState<Fruit | undefined>();

  return (
    <Select onChange={changeFruit} options={fruits} selectedOption="orange" />
  );
};
```

<br/>

selectdedOption은 options에 존재하지 않는 값을 받아도 아무런 오류가 발생하지 않는다. 8.2.3에 언급한 대로 Option의 타입에서 키가 string이기만 하면 prop으로 넘겨줄 수 있기 때문이다.

<br/>

```tsx
type Option = Record<string, string>; // {[key: string]: string}
// 키와 값 모두 문자열

interface SelectProps {
  options: Option;
  selectedOption?: string;
  onChange?: (selected?: string) => void;
  //string 혹은 undefined를 매개변수로 받음. 어떤 값도 반환하지 않음.
  // 이 핸들러는 옵셔널 프로퍼티이므로, 부모 컴포넌트에서 제공하지 않아도 됨.
}
```

<br/>

하지만 changeFruit의 매개변수 Fruit는 prop으로 전달돼야 하는 onChange의 매개변수 타입인 string보다 좁기 때문에 타입 에러가 발생한다.

Select를 사용하는 입장에서 제한된 키와 값만을 가지도록 하려면 어떻게 해야할까? 함수 컴포넌트 역시 함수이므로 제네릭을 사용한 컴포넌트를 만들 수 있다.
<br/>

```tsx
interface SelectProps<OptionType extends Record<string, string>> {
  options: OptionType;
  selectedOption?: keyof OptionType;
  onChange?: (selected?: keyof OptionType) => void;
}

const Select = <OptionType extends Record<string, string>>({
  options,
  selectedOption,
  onChange,
}: SelectProps<OptionType>) => {
  // Select component implementation
};
```

<br/>

이 예제는 객체 형식의 타입을 받아 해당 객체의 키값을 selectedOption, onChange의 매개변수에만 적용하도록 만든 코드이다. Select 컴포넌트에 전달되는 props의 타입 기반으로 타입이 추론되어 <Select<추론된\_타입>> 형태의 컴포넌트가 생성된다. 이제 FruitSelect에서 잘못된 selectedOption을 전달하면 타입 에러가 발생한다.

<br/>

```tsx
const fruits = { apple: "사과", banana: "바나나", blueberry: "블루베리" };

const FruitSelect: VFC = () => {
  // ...
  // <Select<Fruit> ... />으로 작성해도 되지만, 넘겨주는 props의 타입으로 타입 추론을 해줍니다
  // Type Error - Type "orange" is not assignable to type "apple" | "banana" | "blueberry" | undefined

  return (
    <Select options={fruits} onChange={changeFruit} selectedOption="orange" />
    // selectedOption의 prop으로 orange를 전달하지만, fruits 객체의 키중 하나가 아니므로
    // 타입 에러가 발생함.
    // Select 컴포넌트가 OptionType에 기반하여 selectedOption을 추론하기 때문임.
  );
};
```

<br/>
<br/>

### 8.2.7 HTMLAttributes, ReactProps 적용하기

className, id와 같은 리액트 컴포넌트의 기본 props를 추가하려면 SelectProps에 직접 className?: string; id?: string;을 넣어도 되지만 리액트에서 제공하는 타입을 사용하면 더 정확한 타입을 설정할 수 있다.
<br/>

```tsx
type ReactSelectProps = React.ComponentPropsWithoutRef<"select">;

interface SelectProps<OptionType extends Record<string, string>> {
  id?: ReactSelectProps["id"];
  className?: ReactSelectProps["className"];
  // ...
}
```

<br/>

ComponentPropsWithoutRef는 리액트 컴포넌트의 prop 타입을 반환해주는 타입이다. Type[’key’]를 활용하면 객체 형식의 타입 내부 속성값을 가져올 수 있다. ReactProps에서 여러 개의 타입을 가져와야 한다면 Pick 키워드를 활용하여 아래처럼 사용할 수있다.

<br/>

```tsx
interface SelectProps<OptionType extends Record<string, string>>
  extends Pick<ReactSelectProps, "id" | "key" | /* ... */> {
  // ...
}
```

<br/>

Pick<Type, ‘key1’ | ‘key2 …>는 객체 형식의 타입에서 key1, key2…의 속성만 추출하여 새로운 객체 형식의 타입을 반환한다.
<br/>
<br/>

### 8.2.8 styled-components를 활용한 스타일 정의

리액트 컴포넌트를 만들 때 CSS 파일 대신 자바스크립트 안에 직접 스타일을 정의하는 CSS-in-JS 기법을 사용할 수 있다. 그 중 대표적인 styled-components를 활용하여 리액트 컴포넌트에 스타일 관련 타입을 추가해보자.

그 전에 컴포넌트에 스타일을 적용하는 데 사용되는 값을 정의해야 한다. 앞선 예시의 Select 컴포넌트에 글꼴 크기(fontsize)와 현재 선택된 option의 글꼴 색상(font color)을 설정할 수 있게 만들어보자. 일단 theme 객체를 생성하고 프로젝트에 사용될 fontSize와 color, 해당 타입을 간단하게 구성한다.
<br/>

```tsx
const theme = {
  fontSize: {
    default: "16px",
    small: "14px",
    large: "18px",
  },
  color: {
    white: "#FFFFFF",
    black: "#000000",
  },
};

type Theme = typeof theme;
type FontSize = keyof Theme["fontSize"];
type Color = keyof Theme["color"];
```

<br/>

다음 스타일과 관련된 props를 작성하고, color와 font-size의 스타일 정의를 담은 StyledSelect를 작성한다.
<br/>

```tsx
interface SelectStyleProps {
  color: Color;
  fontSize: FontSize;
}

const StyledSelect = styled.select<SelectStyleProps>`
  color: ${({ color }) => theme.color[color]};
  font-size: ${({ fontSize }) => theme.fontSize[fontSize]};
`;

// Select 컴포넌트의 props에 SelectStyleProps 타입을 상속한다.
// Partial<Type>을 사용하면 객체 형식의 타입 내 모든 속성이 옵셔널로 설정된다.
interface SelectProps extends Partial<SelectStyleProps> {
  // ...
}

const Select = <OptionType extends Record<string, string>>({
  fontSize = "default",
  color = "black",
}: // ...
SelectProps<OptionType>) => {
  // ...

  return (
    <StyledSelect
      // ...
      fontSize={fontSize}
      color={color}
      // ...
    />
  );
};
```

<br/>
<br/>

### 8.2.9 공변성과 반공변성

```ts
interface User {
  id: string;
}

// Member가 User 인터페이스를 상속받으므로 Member는 User의 서브 타입이된다. (Member ⊂ User)
interface Member extends User {
  nickName: string;
}

let users: Array<User> = [];
let members: Array<Member> = [];

users = members; // ✅Ok
members = uses; // 🚨Error
```

```ts
type PrintUserInfo<U extends User> = (user: U) => void;

let printUser: PrintUserInfo<User> = (user) => console.log(user.id);
let printMember: PrintUserInfo<Member> = (user) => console.loog(user.id, user.nickName);

printUser = printMember; // 🚨Error - Property 'nickName' is missing in type 'User' but required in type 'Member'
printMember = printUser // ✅Ok
```

타입 A가 B의 서브타입일 때, T\<A>가 T\<B>의 서브타입이 된다면 공변성을 띠고 있다고 말한다. 
일반적인 타입들은 공변성을 지니고 있어서 좁은 타입에서 넓은 타입으로 할당이 가능하다. 
(**TypeScript는 구조적 타입 시스템을 사용하므로, 타입의 구조가 같으면 공명성 조건을 만족한다**)



- 공변성 (Covariance):
타입이 자식 타입에서 부모 타입으로 확장될 수 있는 관계
주로 반환값에 적용

- 반공변성 (Contravariance):
타입이 부모 타입에서 자식 타입으로 확장될 수 있는 관계
주로 함수 매개변수에 적용

- 무공변성 (Invariance):
타입이 변하지 않고, 동일한 타입만 허용


c.f. 
함수의 매개변수는 반공변성
즉, 매개변수 타입이 더 넓은 타입(부모 타입)일 때 함수가 더 안전하게 동작할 수 있기 때문
제네릭 타입의 매개변수도 함수의 매개변수처럼 데이터를 소비(consume)하는 역할을 하므로, **더 넓은 타입(부모)**을 허용하는 반공변성이 필요
```ts
type Animal = { name: string };
type Dog = { name: string; breed: string };

function processAnimal(animal: Animal) {
  console.log(animal.name);
}

function processDog(dog: Dog) {
  console.log(dog.breed);
}

let animalHandler: (arg: Animal) => void;
let dogHandler: (arg: Dog) => void;

// 반공변성 예시
animalHandler = processDog; // ❌ Error: 'Dog' 타입은 'Animal'을 받는 핸들러보다 더 구체적이므로 호환되지 않음.
dogHandler = processAnimal; // ✅ OK: 'Animal'을 받는 핸들러는 'Dog'를 안전하게 처리할 수 있음.

```




제네릭 타입의 매개변수는 함수 매개변수처럼 동작하기 때문에, 
제네릭 타입을 지닌 함수는 반공변성을 가진다. 
즉, T\<B>가 T\<A>의 서브타입이 되어, 좁은 타입 T\<A>의 함수를 넓은 타입 T\<B>의 함수에 적용할 수 없다는 것을 의미한다.


```typescript
interface Props<T extends string> {
  onChangeA?: (selected: T) => void;
  onChangeB?(selected: T): void;
}

const Component = () => {
  const changeToPineApple = (selectedApple: 'apple') => { 
    console.log('this is pine'+selectedApple);
  };
  
  return (
    <Select
      // 🚨Error
      // onChangeA={changeToPineapple}
      // ✅Ok
      onChangeB={changeToPineApple}
    />
  );
};

```

- onChangeA와 같이 함수 타입을 화살표 표기법으로 작성한다면 반공변성을 띤다.
- onChangeB와 같이 함수 타입을 지정하면 공변성과 반공변성을 모두 가지는 이변성을 띠게 된다.


함수 타입을 화살표 표기법으로 작성한다면 반공변성을 띠게 된다. 안전한 타입 가드를 위해서는 특수한 경우를 제외하고는 일반적으로 반공변적인 함수 타입을 설정하는 것이 권장된다.

<br/>
<br/>

## 8.3 정리

리액트 프로젝트에서 타입스크립트는 컴포넌트를 안전하게 조합하고 사용할 수 있도록 도와준다. 또한 다양한 훅을 활용하여 컴포넌트 내부 동작을 구현할 때도 타입을 명확하게 지정함으로써 많은 실수를 미리 방지할 수 있게 해준다. 이처럼 타입스크립트를 잘 활용하면 리액트 프로젝트를 더 안정적으로 운영할 수 있다.

<br/>
<br/>