### 테스팅이란

- 현재는 코드를 프론트 페이지에서 직접 테스트
    - 페이지를 사용자 입장에서 인터페이스를 통해 테스트
    - 일부를 테스트하기 어려움
        - 작은 코드 수정 시 전체 어플리케이션을 다시 실행해야함
- 따라서 자동화 테스트가 필요함
    - 실행되는 과정과 테스트 시나리오를 정의
    - 배포 전에 테스트를 실행해 오류 표시
    - 변경 사항이 발생해도 앱이 재실행되지 않음
    - 정의해둔 커버러지만 테스트하므로 인터페이스 테스트가 어려움

### Why Testing?

- 정의한 모든 코드를 테스트
    - 항상 시나리오가 같다는 가정 하에 적은 오류도 잡아냄
    - 에러에 따른 breakpoint를 확인
- 실행 단계
1. 테스트 실행
    1. 테스트 코드를 정의
    2. Mocha 라이브러리 활용
2. 결과 확인
    1. 특정 테스트에 대한 성공 여부 확인을 위해 실행 결과를 미리 정의해야함
    2. Chai 라이브러리 활용
3. 부작용 & 외부 디펜던시 & 복잡한 환경을 위해 Sinon, Mock 사용

### 첫 번째 테스트 설정

- mocha & chai npm 설치
- package.json → test script 설정

```jsx
"scripts": {
  "test": "mocha --timeout 5000",
  "start": "nodemon app.js"
},
```

- mocha는 test 폴더에서 실행됨

```jsx
// start.js
// should와 expect는 메소드 차이 정도로 미미함
const expect = require('chai').expect;

// 테스트명 + 테스트 함수로 테스트 코드 정의
// 현재는 정적 값으로 작동 테스트만
it('should add numbers correctly', function() {
    const num1 = 2;
    const num2 = 3;
    // expect에 테스트 코드와 결과를 전달
    expect(num1 + num2).to.equal(5);
})

it('should not give a result of 6', function() {
    const num1 = 3;
    const num2 = 3;
    expect(num1 + num2).not.to.equal(6);
})
```

### 인증 미들웨어 테스트하기

- 현재 코드는 애플리케이션과 상관없이 100% 독립 실행
- 애플리케이션과 연계해서 입력값을 받고 검증을 위한 설정
- 

```jsx
const expect = require('chai').expect;

const authMiddleware = require('../middleware/is-auth');

// 보안 검증에 대한 unit test 코드
it('should throw an error if no authorization header is present', function() {
  const req = {
    // 권한을 호출하면(GET 요청) 반환값이 없음
    // 실제로
    get: function(headerName) {
      return null;
    }
  };
  // 에러 시 백엔드에서 반환하는 메시지 결과 예상
  // mocha와 chai가 실행하도록 직접 middleware를 호출하지 않고 참조를  bind
  expect(authMiddleware.bind(this, req, {}, () => {})).to.throw(
    'Not authenticated.'
  );
);
```

- npm test로 테스트

### 테스트 구성하기

- 보안 미들웨어는 Bearer와 토큰을 요청 헤더에서 예상함

```jsx
const token = authHeader.split(" ")[1];
```

- 헤더에 토큰이 없을 경우(split 불가) 에러 출력

```jsx
it('should throw an error if the authorization header is only one string', function() {
  const req = {
    get: function(headerName) {
      return 'xyz';
    }
  };
  expect(authMiddleware.bind(this, req, {}, () => {})).to.throw();
});
// 테스트 실행시 에러 확인
```

- describe 함수로 여러 테스트를 묶어서 호출을 중첩할 수 있음
    - authMiddleware를 사용하는 함수들에게 한번의 호출로 모든 테스트 케이스 전달

```jsx
describe('Auth middleware', function() {
	  it('should throw an error if no authorization header is present', function() {
		...
		
		it('should throw an error if the authorization header is only one string', function() {
	  ...
})
```
