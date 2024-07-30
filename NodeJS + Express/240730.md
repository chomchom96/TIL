### Express.js란?

- Node 단독으로 사용 시 서버에서 요청 처리, 라우팅, 입출력 등 처리에 필요한 코드량이 많음
- 정의 : Node의 필수 작업 및 직접 처리하기 번거로운 작업을 간소화 + 유틸리티 코드를 지원하는 Framework
    - 비즈니스 로직에 집중
- 유연하고 높은 확장성

### Express.js 설치

- 이제 순수 node로 구성한 route handler를 더 이상 사용하지 않음

```jsx
const express = require('express');
const app = express();
const server = http.createServer(app);
server.listen(3000);
```

### 미들웨어 추가

- 미들웨어 : 들어오는 요청을 express.js에 의한 다양한 함수를 통해 자동으로 이동
    - 단일 요청 핸들러 대신 요청이 통과하는 다양한 함수들을 거침
- use method로 미들웨어 함수 추가
    - 요청 핸들러 배열 또는 콜백 함수를 인자로 받음
    - 콜백 함수에는 req, res, next 3가지 인자가 필수
        - next에 한 미들웨어에서 처리가 끝난 요청을 다음 미들웨어로 이동하기 전 실행하는 함수 전달

```jsx
app.use((req, res, next) => {
    console.log("First Middleware");
    next();
});

app.use((req, res, next) => {
    console.log("Second Middleware");
    // 
})

```

### 미들웨어 작동 방식

- expressJs는 기본 응답을 보내지 않음 → 직접 설정해야함
- res.setHeader, res.write가 여전히 가능하지만 res.send()로 본문을 간단히 보낼 수 있음
    - Header 기본값은 text/html, 컨텐츠에 따라 헤더를 설정
    - 파일 등 컨텐츠를 다루기 매우 쉬워짐
- 개념 : 요청이 통과하는 funnel에 기능을 연결, 다음 미들웨어로 연결 또는 응답 전송

### Express.js 백그라운드 확인

- express 깃허브
    - send() 함수 내부 확인
        - 어떤 데이터를 보내는지 확인해서 헤더를 설정하는 내부 로직을 확인할 수 있다

### 다른 라우트 확인법

- 다양한 url 처리하기
    - express use 메소드에 url 전달 가능
        - app.use([path,], callback, [callback])
        - url을 단일 또는 배열로 전달함

```jsx
app.use("/", (req, res, next) => {
    console.log("전역 필터");
    next();
})

app.use("/add-product", (req, res, next) => {
  res.send("<h1>Add Product</h1>");
});

app.use("/", (req, res, next) => {
  res.send("<h1>Hi From Express!</h1>");
});

```

- url 검사는 top-down으로
- 응답을 전송한 뒤 next를 호출하지 말 것

### 수신 요청 분석

```jsx
app.use("/add-product", (req, res, next) => {
  res.send("<form action='/product' method='post'><input type='text' name='title' /><button type='submit'>Add product</button></form>");
});

app.use("/product", (req, res, next) => {
    // redirect + console log
    console.log(req.body);
    res.redirect('/');
});
```

→ 로그에 undefined

- req는 들어오는 요청의 본문을 파싱하지 않음
- 경로 처리 미들웨어 전에 body 파싱 미들웨어를 설정
    - body-parser 패키지 설치

```jsx
const bodyParser = require("body-parser");

app.use(bodyParser.urlencoded({extended: false}));
```

- 모든 미들웨어 함수의 끝에서 next를 호출해서 요청이 미들웨어에 도달하도록 함
    - 전에 수동으로 했던 본문 파싱도 진행
    - 폼을 통해 전송한 데이터를 분석
- 이제 콘솔에 body가 출력됨
    - {title: ‘book’}인 key-value JS 객체

### POST 요청으로 미들웨어 제한

- 현재 미들웨어는 GET, POST 요청 모두에서 실행됨
- app.use 대신 app.get 사용 가능
    - 들어오는 GET 요청에만 사용
- app.post도 같은 원리

### Express 라우터 사용

- 규모가 커질수록 라우팅 로직이 복잡해질 수 있음
- Express.js에선 다른 파일에 라우팅을 분리함
- routes 폴더 → admin.js, shop.js 생성
- POST 요청은 admin.js에서, 홈페이지는 shop.js에서 처리

```jsx
// admin.js
const express = require("express");

const router = express.Router();

router.get("/add-product", (req, res, next) => {
  res.send(
    "<form action='/product' method='post'><input type='text' name='title' /><button type='submit'>Add product</button></form>"
  );
});

router.post("/product", (req, res, next) => {
  console.log(req.body);
  res.redirect("/");
});

module.exports = router;
// app.js
const adminRoutes = require('/routes/admin');

app.use(adminRoutes);
```

### 404 오류 페이지

- 현재는 모든 페이지를 GET/POST 처리하므로 아무 url을 입력하면 404 에러 발생
- 라우팅 후에 모든 페이지의 요청을 받을 수 있음

```jsx
app.use(shopRoutes);
app.use(adminRoutes);

app.use('/', (req, res, next) => {
    res.status(404).send('<h1>Page not found</h1>')
})
```

- Express는 res status를 한줄로 처리할 수 있다
