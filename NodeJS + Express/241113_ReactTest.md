### Why & How

- 일반적으로 브라우저에서 직접 작동 + 테스트
    - 사용자가 사용하는 방법과 같으므로 중요함
    - 모든 케이스의 시나리오를 테스트하기 어려우므로 에러에 취약
    - 수정한 부분 외에 다른 기존 부분에 발생한 에러를 잡기 어려울 수 있음
- 자동 테스트로 테스트 케이스에 대한 테스트

### 테스트 기술 설정

- 테스트 실행 및 결과 확인
    - Jest
- 리액트 컴포넌트의 렌더링 테스트
    - React Testing Library
- create-react-app의 기본 설정에 포함

### 첫 번째 테스트 실행하기

```jsx
// App.test.js
// 테스트할 컴포넌트 파일과 파일명을 같게 하는 게 컨벤션
// .test 확장자로 명시

// test 함수의 인자는 테스트명(설명)과 익명함수
test('renders learn react link', () => {
  render(<App />);
  // 대소문자 구분없이 해당 글자를 화면에 있는지 확인
  const linkElement = screen.getByText(/learn react/i);
  expect(linkElement).toBeInTheDocument();
});
```

- 터미널에서 npm test로 test script 실행
    - 실패시 해당 소스코드와 발생위치를 터미널에 출력

### 첫 번째 테스트 작성하기

- 직접 작성한 컴포넌트로 테스트 작성하기

```jsx
// Greeting.js
const Greeting = () => {
  return (
    <>
      <h2>Hello World!</h2>
      <p>ㅎㅇ</p>
    </>
  );
};

export default Greeting;
// App.js
function App() {
  return (
    <Greeting />
  );
}
// Greeting.test.js
// test 메소드는 import 필요없음
test('renders hello in screen', () => {
  render(<Greeting />);
  // 정규식 또는 문자열로 찾을수있음
  const element = screen.getByText("Hello World", {exact: false});
  expect(element).toBeInTheDocument();
});
```

### 테스트 그룹화하기

- 컴포넌트나 기능 내에 속하는 테스트를 describe suite에 묶어서 저장

```jsx
describe('Greeting component', () => {
    test();
    test();
})
```

### 사용자 상호 작용 & State test

- 간단한 state에 대해 테스트 작성하기

```jsx
import { useState } from "react";

const Greeting = () => {
  const [state, setState] = useState(false);
  return (
    <>
      <h2>Hello</h2>
      {!state && <p>false</p>}
      {state && <p>true</p>}
      <button onClick={() => setState(!state)}>button</button>
    </>
  );
};

export default Greeting;

// test
describe("Greeting component", () => {
  test("rendering test", () => {
    render(<Greeting />);
    // 정규식 또는 문자열로 찾을수있음
    const element = screen.getByText("Hello World", {exact: false});
    expect(element).toBeInTheDocument();
  });

  test("default", () => {
    render(<Greeting />);
    const element = screen.getByText("false", {exact: false});
    expect(element).toBeInTheDocument();
  });
  // 버튼 클릭시 테스트
  // testing-library의 userEvent import
  test("state change when button clicked", () => {
    render(<Greeting />);
    const buttonElement = screen.getByRole("button");
    userEvent.click(buttonElement);

    const outputTrueElement = screen.getByText("true");
    const outputFalseElement = screen.getByText("false");
    // 표시됨
    expect(outputTrueElement).toBeInTheDocument();
    // 존재하지 않음
    expect(outputFalseElement).toBeNull();
  });
});

```

### 외부 컴포넌트 테스트

- props와 외부 컴포넌트 테스트

```jsx
// Output
export default function Output(props) {
    return <p>props.text</p>
};
// Greeting
const Greeting = () => {
  const [state, setState] = useState(false);
  return (
    <>
      <h2>Hello</h2>
      {/* {!state && <p><Output props={state} /></p>}
      {state && <p><Output props={state} /></p>} */}
      <p><Output props={state} /></p>
      <button onClick={() => setState(!state)}>button</button>
    </>
  );
};

export default Greeting;
```

- 정상 작동함 → render 함수이므로
    - 통합 테스트라 할 수 있다

### 비동기 테스트

- useEffect로 API를 비동기로 받아오는 테스트

```jsx
// Async.js
import { useEffect, useState } from 'react';

const Async = () => {
  const [posts, setPosts] = useState([]);

  useEffect(() => {
    fetch('https://jsonplaceholder.typicode.com/posts')
      .then((response) => response.json())
      .then((data) => {
        setPosts(data);
      });
  }, []);

  return (
    <div>
      <ul>
        {posts.map((post) => (
          <li key={post.id}>{post.title}</li>
        ))}
      </ul>
    </div>
  );
};

export default Async;
// test
describe("Greeting component", () => {
    test('non async rendering', () => {
        render(<Async />);
        // screen 메소드로 접근하면 비동기 작업을 기다리지 않고 즉시 테스트 
        const listItemElement = screen.getAllByRole("listitem");
        // 통과X(promise가 반환되지 않음)
        expect(listItemElement).not.toHaveLength(0);
    });

    test('async rendering', async () => {
        render(<Async />);
        // find 또는 find 시작 메소드는 성공할 떄까지 여러 번 테스트 실행 -> async를 wait
        const listItemElement = await screen.findAllByRole("listitem");
        // 통과X(promise가 반환되지 않음)
        expect(listItemElement).not.toHaveLength(0);
    });
});
```

### 모의 작업

- 여전히 최선의 테스트가 아님
    - HTTP 요청을 보내고 있는데 보통 테스트 시 서버에 HTTP 요청을 보내진 않음
        - DB가 변경되거나 서버 과부화 등
    - 실제 요청을 전송하지 않거나 테스트 서버를 운영할 수 있음
    - fetch 함수는 테스트하지 않음 → 브라우저 내장 함수
- 실제 요청 대신 mock 요청을 사용하기
    - Jest의 fn 메소드

```jsx
test('async rendering', async () => {
    window.fetch = jest.fn();
    // fetch 호출 후 반환되는 값을 객체로 설정
    window.fetch.mockResolvedValueOnce({
        // 포스트 배열
        json: async () => [{id: "1", title: "Post"}]
    });
    render(<Async />);
    const listItemElement = await screen.findAllByRole("listitem");
    expect(listItemElement).not.toHaveLength(0);
});
```
