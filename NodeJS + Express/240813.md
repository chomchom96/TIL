### Sequalize란?

- ORM 지원 라이브러리
    - SQL 쿼리를 직접 작성하지 않아도 됨
- Sequalize로 객체를 DB에 연결 → 관계를 자동으로 설정
- JS에서 객체를 호출하면 Sequalize가 SQL에서 필요한 쿼리를 작성
- 모델 정의 → 모델 기반 사용자 객체 생성 → 쿼리 생성 → 모델 간 연결

### DB에 연결

- sequelize 설치
    - mysql2가 설치 되어있어야 함
- mysql의 기존 데이터베이스 삭제

```
const Sequalize = require('sequelize');
// schema, username, pw
const sequelize = new Sequalize('node_complete', 'root', '1234', {dialect: 'mysql', host: 'localhost'});
```

### 모델 정의

```jsx
const Sequalize = require("sequelize");

const sequelize = require('../util/database');
// db connnection 생성 
// 이름은 lowercase
const Product = sequelize.define('product', {
  id: {
    type: Sequalize.INTEGER,
    autoIncrement: true,
    allowNull: false,
    primaryKey: true,
  },
  title: Sequalize.STRING,
  price: {
    type: Sequalize.NUMBER,
    allowNull: false,
  },
  imageUrl: {
    type: Sequalize.STRING,
    allowNull: false,
  },
  description: {
    type: Sequalize.STRING,
    allowNull: false,
  }
});

module.exports = Product;
```

### DB에 JS 정의를 동기화

- Sequalize에 테이블 생성 명령

```jsx
const sequelize = require('./util/database');
...
sequelize.sync().then(result => {
    app.listen(3000);
})
.catch(err => {
    console.log(err);
});
```

→ 이름의 복수형인 테이블명 생성(product → products)

### 데이터 삽입 및 제품 생성

- create → 모델 기반 객체 생성 후 즉시 db에 반영
- build → 모델 기반, 저장을 직접 함
    - 모든 Sequelize 메소드는 Promise, 반환 후 then, catch

```jsx
exports.postAddProduct = (req, res, next) => {
  const title = req.body.title;
  const imageUrl = req.body.imageUrl;
  const price = req.body.price;
  const description = req.body.description;
  Product.create({
    title: title,
    imageUrl: imageUrl,
    price: price,
    description: description,
  })
    .then((result) => {
      console.log(result);
    })
    .catch((err) => {
      console.log(err);
    });
  // product.save();
  res.redirect("/");
};
```

### 데이터 검색 및 제품 찾기

- 이전의 fetchAll 메소드 대신 Sequelize의 findAll 메소드로 전체 목록을 가져옴

```jsx
exports.getIndex = (req, res, next) => {
  Product.fetchAll().then(products => {
    res.render('shop/index', {
      prods: products,
      pageTitle: 'Shop',
      path: '/'
    });
  })
};
```

### Where 조건으로 단일 제품 찾기

- Sequelize 5버전 이상부터는 findById → findByPk

```jsx
exports.getProduct = (req, res, next) => {
  const prodId = req.params.productId;
  Product.findByPk(prodId)
    .then((product) => {
      res.render("shop/product-detail", {
        product: product,
        pageTitle: product.title,
        path: "/products",
      });
    })
    .catch((err) => console.log(err));
};

```

- findAll → where문으로 검색 param 조절 가능
    - 이 경우 배열을 반환하므로 유의

```jsx
  Product.findAll({ where: { id: prodId } })
    .then((product) => {
      res.render("shop/product-detail", {
        product: product[0],
        pageTitle: product[0].title,
        path: "/products",
      });
    })
    .catch((err) => console.log(err));
```

### 상품 업데이트하기

- 편집할 제품 정보 가져오기 → Update
    - 더이상 JS 객체를 넘겨서 업데이트하지 않음

```jsx
exports.postEditProduct = (req, res, next) => {
  const prodId = req.body.productId;
  const updatedTitle = req.body.title;
  const updatedPrice = req.body.price;
  const updatedImageUrl = req.body.imageUrl;
  const updatedDesc = req.body.description;
  Product.findByPk(prodId).then((product) => {
    product.title = updatedTitle;
    product.price = updatedPrice;
    product.imageUrl = updatedImageUrl;
    product.updatedDesc = updatedDesc;
    return product.save();
  })
  .then(result => {
    console.log("updated");
  })
  .catch(err => console.log(err));
  res.redirect("/admin/products");
};
```

- 업데이트시 DB는 반영되지만 페이지에는 반영되지 않음
    - Promise에 전달되지 않고 redirect하기 때문, then 구문에 redirect

### 상품 삭제하기

- destroy 메소드 사용
    - findByPk → destroy 또는 바로 destory 메소드 사용 가능

```jsx
exports.postDeleteProduct = (req, res, next) => {
  const prodId = req.body.productId;
  Product.findByPk(prodId)
    .then((product) => {
      return Product.destroy();
    })
    .then((result) => {
      res.redirect("/admin/products");
    })
    .catch((err) => console.log(err));
};
```

### 사용자 모델 생성하기

```jsx
const Sequelize = require("sequelize");

const sequelize = require("../util/database");

const User = sequelize.define("user", {
  id: {
    type: Sequelize.INTEGER,
    autoIncrement: true,
    allowNull: false,
    primaryKey: true,
  },
  name: Sequelize.STRING,
});

module.exports = User;
```

### 연관 관계 설정하기

- Product, User, Cart, Order 테이블이 존재함
    - Product → 많은 Cart에 속함
    - User- Cart는 일대일
    - User - Order는 일대다
    - Product - User는 다대다

```jsx
const Product = require('./models/product');
const User = require('./models/user');

Product.belongsTo(User, {
  constratins: true,
  // onDelete: 'CASCADE',
});

User.hasMany(Product);
...
// 테이블 변경사항 강제 적용하기
sequelize
  .sync({
    force: true,
  })
```

### 더미 사용자 데이터 생성 및 관리

```jsx
sequelize
  .sync({
    // force: true,
  })
  .then((result) => {
    return User.findByPk(1);
    // console.log(result);
  })
  .then((user) => {
    if (!user) {
      User.create({ name: "Max", email: "example@example.com" });
    }
    return user;
  })
  .then((user) => {
    console.log(user);
    app.listen(3000);
  })
  .catch((err) => {
    console.log(err);
  });
```

- 앱을 실행할 때마다 항상 더미 사용자가 생성됨
- 현재 사용자를 설정하는 미들웨어 추가

```jsx
//사용자 설정 미들웨어
app.use((req, res, next) => {
  User.findByPk(1).then(user => {
    // sequelize 객체를 반환
    req.user = user;
    next();
  })
  .catch(err => console.log(err));
});
```
