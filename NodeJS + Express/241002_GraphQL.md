### GraphQL이란?

- REST on steroids
- REST API는 stateless, 클라이언트와 독립되어 데이터를 교환하는 API임
- 뷰 렌더링, 세션 저장, 클라이언트에 따라 변화하지 않고 받은 요청을 분석해서 응답을 반환
- GraphQL API도 동일함, 차이점은 쿼리 유연성이 높음
- REST API의 제한점
    - REST의 endpoint ex: GET /post에 요청할 경우
        - 요청에 따라 데이터를 반환함
        - 클라이언트가 모든 데이터를 필요하지 않은 경우(id, title 외에 content, createdAt 등)가 있음
        - 하나의 endpoint를 사용하는 다양한 시나리오가 존재할 수 있음
    - 새로운 endpoint를 생성해서 다양한 경우 처리할 수도 있음
        - 클라이언트에 지나치게 많은 데이터를 발생하면 문제가 생길 수 있음, 특히 모바일 환경
        - endpoint가 많아지면서 REST API가 복잡해지고 유연성이 떨어짐
        - 새로운 유형의 데이터가 필요하면 백엔드 개발자가 생성할 REST API가 매번 발생
    - query param 사용
        - data=slim 등 상황에 대한 param 전달
        - 역시 번거롭게 매번 API를 갱신해야함
        - param에 따라 혼동이 발생할 수 있음
- GraphQL로 해결 가능
    - 쿼리 언어가 풍부해 요구사항에 맞는 데이터를 동적으로 검색
    - 쿼리 언어를 프론트엔드에서 접근할 수 있음
    - 백엔드 요청에 쿼리를 같이 보냄
- GraphQL의 유일한 요청은 /graphql POST 요청 → 하나의 endpoint
- POST 본문에 쿼리를 넣음

```jsx
{
	query { // 요청 타입
		user { // 요청의 endpoint
			name // 필요 필드명
			age
		}
	}
}
```

- 요청 타입
    - query →REST API의 GET
    - mutation → POST, PUT, PATCH, DELETE
    - subscription → websocket으로 실시간 접속 생성
- Type definition는 요청 타입을 정의하고 설정
- 정의된 요청은 서버 측 논리를 포함한 Resolver 함수에 연결
- Type definition은 라우터, Resolver는 컨트롤러와 유사

### 설정 이해 및 쿼리 작성

- 더 이상 라우트 기반으로 작동하지 않음, 라우트 제거
- GraphQL 설치 및 설정
    - graphql과 express버전의 express-graphql도 설치

```jsx
npm i graphql --save express-graphql --save
```

- 필요한 resolver, schema 설정

```jsx
// schema.js
const { buildSchema } = require('graphql');

module.exports = buildSchema(`
	// 느낌표는 필수 필드
	// 필드간 쉼표 없음
	type TestData {
		test: String!
		views: Int!
	}
	
	type RootQuery {
		// 만들 수 있는 유형의 쿼리 설정
		// 쿼리명과 반환 타입 설정
		hello: TestData!
	}
	schema {
			// 허용하는 쿼리 정의
			query: rootQuery
		}
`);
// resolver.js
module.exports = {
	 // 스키마에 정의된 각 쿼리에 대응하는 메소드, 이름이 일치해야함
	 // 루트 쿼리 말고 서브 쿼리에 resolver가 필요함
	 hello() {
		 return 'Hello World!',
		 views: 12121,
  }
}
// app.js
const graphqlHttp = require('express-graphql');

app.use(
  '/graphql',
  // 패키지 메소드에 schema, resolver 전달
  graphqlHttp({
    schema: graphqlSchema,
    rootValue: graphqlResolver,
  })
);
```

- Postman으로 graphql에 요청 보내기
    - POST 요청, endpoint는 통일

```jsx
{
//  "{}" 안에 요구 쿼리 작성
// 추가 괄호 안에 필요한 필드명 작성
	query: "{ hello { text views } }"
}
```

### 뮤테이션 스키마 정의하기

- 회원가입에서 받은 정보로 회원을 생성하기 위해  뮤테이션 사용

```jsx
// schema.js
 type Post {
 		// id가 unique함을 전달
    _id: ID!
    title: String!
    content: String!
    imageUrl: String!
    creator: User!
    createdAt: String!
    updatedAt: String!
}
// input 데이터 정의는 type 대신 input 키워드 사용
input UserInputData {
      email: String!
      name: String!
      password: String!
}

type User {
    _id: ID!
    name: String!
    email: String!
	  // pw는 항상 필요하지 않음
    password: String
    status: String!
    // post 배열
    posts: [Post!]!
}
    
type RootMutation {
    createUser(userInput: UserInputData): User!
}

schema {
    query: RootQuery
    mutation: RootMutation
}
```

### 뮤테이션 분석기 및 GraphQL 사용하기

- 사용자 생성 resolver 생성하기

```jsx
module.exports = {
	// async await 또는 promise로 비동기 처리
	// async await를 사용하지 않으면 직접 promise를 return 해야함
  createUser: async function({ userInput }, req) {
	  // 스키마에서 정의한 모든 typess를 args로 접근 가능
	  const email = userInput.email;
	  const existingUser = await User.findone({email: email});
	  if (existingUser) {
		  const error = new Error("기존유저");
		  throw error;
	  }
	  cosnt hashedPw = bcrypt.hash(userInput.password, 12);
    const user = new User({
      email: userInput.email,
      name: userInput.name,
      password: hashedPw
    });
    const createdUser = await user.save();
    // _doc으로 Mongoose가 추가한 메타데이터 제외
    // 독립된 _id를 추가해 id 덮어쓰기
    return { ...createdUser._doc, _id: createdUser._id.toString() };
  }
}
```

- app에서 graphiql: true 설정 → 실행된 서버에서 endpoint로 GET 요청을 받을 수 있음(localhost:8080/graphql) → 브라우저에서 graphql 작업을 할 수 있음

```jsx

app.use(
  '/graphql',
  graphqlHttp({
    schema: graphqlSchema,
    rootValue: graphqlResolver,
    graphiql: true,
    
  })
);
```

### 입력 검증 추가하기

- endpoint가 하나이므로 모든 요청에 대해 동일한 검증을 하지 않음
- 따라서  resolver에서 입력에 대한 검증을 함
- validator 패키지 설치 → express validater가 배후에서 실행하던 로직을 직접 실행

```jsx
const validator = require('validator');

module.exports = {
  createUser: async function({ userInput }, req) {
    //   const email = args.userInput.email;
    const errors = [];
    if (!validator.isEmail(userInput.email)) {
      errors.push({ message: 'E-Mail is invalid.' });
    }
    if (
      validator.isEmpty(userInput.password) ||
      !validator.isLength(userInput.password, { min: 5 })
    ) {
      errors.push({ message: 'Password too short!' });
    }
    if (errors.length > 0) {
      const error = new Error('Invalid input.');
      throw error;
    }
}
```

### 오류 처리

- 에러 발생시 코드 500으로 에러 처리 확인
- 반환되는 에러에 정보 추가하기
    - 직접 에러 포맷을 만들어서 설정할 수 있음

```jsx
// app.js
app.use(
  '/graphql',
  graphqlHttp({
    schema: graphqlSchema,
    rootValue: graphqlResolver,
    graphiql: true,
    formatError(err) {
	    // express-graphql이 사용자나 제3자 패키지의 오류를 감지할 경우 -> graphql이 분류할 수 없음
      if (!err.originalError) {
        return err;
      }
      // originalError 처리
      const data = err.originalError.data;
      const message = err.message || 'An error occurred.';
      const code = err.originalError.code || 500;
      return { message: message, status: code, data: data };
    }
  })
);
// resolver에서 originalError 사용
 if (errors.length > 0) {
    const error = new Error('Invalid input.');
    error.data = errors;
    error.code = 422;
    throw error;
  }
  const existingUser = await User.findOne({ email: userInput.email });
  if (existingUser) {
    const error = new Error('User exists already!');
    throw error;
  };
```
