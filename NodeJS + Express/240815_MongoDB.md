### MongoDB란?

- NoSQL을 위한 DB 솔루션 또는 엔진
    - Homongous : 거대한 양의 데이터를 저장, 데이터 쿼리, 상호작용 등을 빠르게 할 수 있음
- MongoDB의 DB 구조
    - DB → Collection → Documents
- 스키마가 없음 → Collection 내 Document마다 같은 구조를 가질 필요가 없음
- Binary JSON을 사용 → JSON 객체와 동일, 저장 방식이 다름
- Document의 배열은 다른 문서, 객체, 문자열, 숫자 등을 원소로 가짐

### NoSQL에서의 관계

- 전에 사용한 Order, User, Product 테이블을 사용
    - 다른 문서에 중복된 데이터를 저장(Order에 User의 정보 중복 저장)
    - Order에서 특정 사용자로 검색할 때 전체 Order를 검색할 필요가 없음 → 속도 개선
- 관계를 만들 수 있음
    - User 안에 중첩 Document인 address를 넣을 수 있음
    - Reference : Product 정보가 바뀔 경우 Order의 정보를 갱신하기 위해 Order 문서에 reference를 저장(Product를 id로 저장)해서 Product 테이블에서 정보를 조회
- 속도와 유연성이 뛰어남

### MongoDB 설정하기

- MongoDB 설치하기
    - https://mongodb.com/ → get MongoDB → 계정 등록
    - MongoDB 클라우드에 프로젝트(Cluster) 생성
    - 사용자 권한을 Read & Write로 설정
    - IP Whitelist에서 현재 IP 주소 등록하기 + 나중에 배포 주소 등록
    - Cluster → Connect → connection string 복사하기

### MongoDB 드라이버 설치하기

- mongodb 설치

```jsx
npm i mongodb --save
```

- app.js, database.js에서 sequelize 코드 삭제 후 설정하기

```jsx
const mongodb = require('mongodb');
const MongoClient = mongodb.MongoClient;

const mongoConnect = (callback) => {
    // connection string & pw
    MongoClient.connect('string&pw')
    .then(client => {
        console.log("connected!");
        callback(client)
        ;
    })
    .catch(err => console.log(err));
}

module.exports = mongoConnect;

//app.js
const mongoConnect = require("./util/database");
...
mongoConnect((client) => {
  console.log(client);
  app.listen(3000);
})
```

### DB 연결 생성하기

- sequelize를 사용했던 라우트 재연결하기
    - 라우트를 연결하는 컨트롤러가 의존하는 모델이 Sequelize를 사용하고 있기 때문
    - 모델부터 수정

```jsx
const mongoConnect = require('../util/database');

class Product {
  constructor(title, price, description, imageUrl) {
    this.title = title,
    this.price = price,
    this.description = description,
    this.imageUrl = imageUrl
  }

  save() {

  }
}
```

- 현재 database에서 mongodb에 연결하는 방식은 좋지 않음
    - 콜백 후 연결을 클라이언트에 반환중
    - 모든 작업마다 DB에 연결 + 작업 후 연결 해제가 불가능
    - 앱의 다양한 위치에서 접근이 필요함
    - DB의 한 연결을 처리, 클라이언트로 접근을 반환 후 작업이 필요한 곳에서 접근

### DB 연결 완료하기

- connection string에서 [mongodb.net/](http://mongodb.net/)<db명>?에서 db명을변경하면 접근할 db를 설정 가능하며 없는 경우 새로 생성
- 연결 생성과 DB 접근 메소드를 분리하기

```jsx
const mongoConnect = (callback) => {

    .then((client) => {
      //DB 연결 저장
      _db = client.db();
      callback(client);
    })
	...
};

const getDb = () => {
    if (_db) return _db;
    throw "No DB Found";
}

exports.mongoConnect = mongoConnect;
exports.getDB = getDb;
```

- Connection Pooling을 통해 여러 연결을 동시에 처리함

### DB 연결 사용하기

```jsx
  save() {
    const db = getDb();
    // 작업할 콜렉션 지정, 없을 경우 생성
    // insertMany -> 배열, insertOne -> 객체
    db.collection('products').insertOne(this)
    .then(result => {
      console.log(result);
    })
    .catch(err => {
      console.log(err);
    });
  }
```

### 제품 생성하기

```jsx
// 컨트롤러
const product = new Product(title, price, description, imageUrl);

  Product.save(product)
    .then((result) => {
      console.log(result);
      res.redirect("admin/product");
    })
```

- 콘솔 로그에서 문서의 자동 생성된 id와 입력한 데이터 확인 가능

### MongoDB Compass의 이해

- MongoDB에 넣은 데이터를 확인하기 위해 Compass 설치
    - DB 정보를 확인하고 연결 및 상호작용을 돕는 GUI 툴
    - Cluster → Connect to MongoDB Compass → URL을 Compass에 입력 후 연결
