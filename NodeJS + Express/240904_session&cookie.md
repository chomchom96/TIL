### 쿠키란?

- 사용자가 브라우저에서 서버와 연결할 때 특정 데이터를 저장해야 함
    - 사용자가 로그인한 정보
- 요청에 대한 응답에 쿠키를 함께 전송
- 쿠키는 사용자가 인증을 마쳤음을 증명하는 데 사용

### 요청 기반 로그인 솔루션 설치하기

- 로그인 후 사용자가 인증되었다고 가정하고 화면을 렌더링

```jsx
// admin controller
exports.getLogin = (req, res, next) => 
  res.render('auth/login', {
    path: '/login',
    pageTitle: 'Login',
    isAuthenticated: req.isLoggedIn
  });
};

exports.postLogin = (req, res, next) => {
  res.isLoggedIn = true;
  res.redirect('/');
};

// shop controller

exports.getProducts = (req, res, next) => {
  Product.find()
    .then(products => {
      console.log(products);
      res.render('shop/product-list', {
        ...
        isAuthenticated: req.isLoggedIn
      });
    })
};

// view
 <% if (isAuthenticated) { %>
  <li class="main-header__item">
      <a class="<%= path === '/admin/add-product' ? 'active' : '' %>" href="/admin/add-product">Add Product
      </a>
  </li>
  <li class="main-header__item">
      <a class="<%= path === '/admin/products' ? 'active' : '' %>" href="/admin/products">Admin Products
      </a>
  </li>
  <% } %>
```

### 쿠키 설정하기

- 요청은 응답을 지난 뒤 무효해짐 → req.isLoggedIn 접근의 로직에 문제가 있음
    - 대안 1 → JS 글로벌 변수 사용하기
        - 모든 요청 간 공유가 되지 않음
    - 대안 2 → 쿠키
        - 브라우저에 해당 사용자의 데이터를 저장
        - 사용자 별로 정보를 저장할 수 있음
- 요청에 헤더 설정
    - Set-Cookie 헤더명으로 서버에서 쿠키로 저장을 요청
    - key=value 형태로 요청

```jsx
exports.getLogin = (req, res, next) => {
  const isLoggedIn = req
    .get('Cookie')
    .split(';')[1]
    .trim()
    .split('=')[1];
  res.render('auth/login', {
    path: '/login',
    pageTitle: 'Login',
    isAuthenticated: isLoggedIn
  });
};

exports.postLogin = (req, res, next) => {
  res.isLoggedIn = true;
  res.setHeader('Set-Cookie', 'loggedIn=true');
  res.redirect('/');
};

```

### 쿠키 조작하기

- 개발자 도구의 Application 탭에서 쿠키 데이터를 쉽게 접근할 수 있음
    - 클라이언트에서 조작 가능한 쿠키 데이터를 서버에서 신용할 수 없음
    - 민감한 정보 대신 간단한 정보를 저장하기
    - 세션을 통해 보안 보완

### 쿠키 저장하기

- 쿠키가 사용자 추적에 사용되는 이유
    - 쿠키는 현재 페이지 외에 다른 페이지에서도 전달됨
    - 트래킹 픽셀과 같이 사용자 추적에 사용
    - 구글 사이트에 있는 이미지 페이지에 존재하는 쿠키를 함께 전송해서 어느 페이지에 있는지 확인하고 구글이 이동 경로를 파악
    - 서버에 저장된 채 구글의 모든 요청에 전송
- key=value 외 다양한 옵션 존재
    - Expires=날짜 → 만료일
    - Max-Age=초 → 쿠키 수명을 초 단위로 설정
    - Secure : HTTPS 페이지가 제공될 경우만 전달
        - 클라이언트 사이드에서 쿠키에 접근이 불가능
        - XSS(Cross Site Scripting) 공격을 방어
        - 악성 코드가 포함될 수 있는 클라이언트 쿠키를 서버가 읽지 않음

### 세션이란?

- 사용자 인증 정보를 프론트엔드(브라우저)에 저장하는 것은 바람직하지 않음
    - 백엔드의 세션에 저장
- 사용자별로 존재하는 세션에 인증 데이터를 유지
- 클라이언트가 서버에 본인의 세션을 알리기 위해 세션의 ID를 저장하는 쿠키로 인증
    - 세션의 정보는 브라우저에서 접근 불가 → 당연함

### 세션 미들웨어 초기화

- 세션 관리 패키지 express-session 설치

```jsx
app.use(session({
  secret: "my-secret", // secret key
  resave: false, // 세션이 변경될 때만 저장
  saveUninitialized: false, // 변경사항이 없으면 저장하지 않음
  // 쿠키 설정 가능
}))
```

### MongoDB를 사용해 세션 저장하기

- 세션이 메모리에 저장되고 있음 → 사용자 수에 따라 메모리 제약이 걸림
- Express 깃허브에서 세션 DB로 지원하는 데이터베이스 목록 확인 가능
    - connect-mongodb-session 패키지 설치

```jsx
const MongoDBStore = require("connect-mongodb-session")(session);

// URI의 retryWrites=true 제거
const store = new MongoDBStore({
  url: MONGODB_URI,
  collection: 'sessions',
});

...
app.use(
  session({
    secret: "my-secret", // secret key
    resave: false, // 세션이 변경될 때만 저장
    saveUninitialized: false, // 변경사항이 없으면 저장하지 않음
    store: store
  })
);
```

- 장바구니 등 기능에도 활용 가능
- 매번 접속할 때마다 접근하지만 다른 사용자에게 보이지 않는 데이터에 세션을 활용

### 세션 및 쿠키 - 요약

- 세션 쿠키 설정은 experss 미들웨어에 의해 실행
    - 쿠키 저장, 분석, 설정을 모두 처리
- 세션 쿠키로 사용자 인증은 가장 흔한 인증 방법
- 로그인 정보 뿐만 아니라 다른 정보 저장 가능

### 쿠키 삭제

- 로그아웃 시 브라우저에서 세션 쿠키 단순 삭제는 바람직하지 않음
    - db에 사용한 이전 쿠키들의 데이터가 그대로 남아있음
- 세션 미들웨어가 지원하는 메소드를 사용

```jsx
exports.postLogout = (req, res, next) => {
  req.session.destroy(() => {
    res.redirect('/');
  })
}
```

- 해당하는 세션이 삭제됨

### 사소한 버그 수정하기

- 장바구니, shop 등의 모든 사용자 인증 메소드가 세션 정보 대신 user에 직접 접근하고 있음
    - 사용자 인증 메소드에 세션 추가하기
    - 장바구니는 로그인한 상태에만 렌더링

### 장바구니에 추가 만들기

- 사용자 객체에서 사용하던 장바구니 추가 기능이 작동하지 않음
    - 이전 로직은 요청에 사용자를 저장 + 미들웨어에서 저장한 요청의 사용자를 가져옴
    - 세션을 사용하므로 로그인 시에만 사용자 정보를 저장함
- req.user로 접근하는 사용자 정보를 req.session.user_id로 대체해서 Mongoose 메소드에 접근
    - 또는 변수를 초기화해서 req.user에 req.session.user_id를 할당하면 코드를 전부 고칠 필요가 없음
