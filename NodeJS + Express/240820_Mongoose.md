### Mongoose란?

- Object-Document Mapping Library
    - Sequelize는 Object Relational Mapping Library
    - Document 기반의 MongoDB에 맞춤형 매핑
- 객체와 데이터에 초점을 맞추며 쿼리문이 간결해짐
- 스키마와 모델을 통해 데이터가 보일 형태를 정함
- 인스턴스로 모델에 예시를 제시 → JS 객체 가이드라인

### Mongoose로 MongoDB 연결하기

www.mongoosejs.com

- 유틸리티 및 연결 관리를 mongoose가 하기 때문에 별도의 db 설정이 불필요

```jsx
//app.js
const mongoose = require('mongoose');

mongoose
  .connect(
	  'url'
  )
  .then(result => {
    app.listen(3000);
  })
  .catch(err => {
    console.log(err);
  });
```

- mongoose에 맞게 모델을 모두 수정할 예정

### 제품 스키마 생성하기

- mongoose의 schema(청사진) 만들기

```jsx
const mongoose = require('mongoose');

const Schema = mongoose.Schema;

const productSchema = new Schema({
  title: {
    type: String,
    required: true
  },
  price: {
    type: Number,
    required: true
  },
  description: {
    type: String,
    required: true
  },
  imageUrl: {
    type: String,
    required: true
  }
```

### Mongoose로 데이터 저장하기

- model 함수는 mongoose가 해당 이름의 스키마와 연결하는 함수

```jsx
module.exports = mongoose.model('Product', productSchema);
```

- 컨트롤러에서 모델에 접근하기

```jsx
exports.postAddProduct = (req, res, next) => {
  const title = req.body.title;
  const imageUrl = req.body.imageUrl;
  const price = req.body.price;
  const description = req.body.description;
  // JS 객체와 유사한 생성
  const product = new Product({
    title: title,
    price: price,
    description: description,
    imageUrl: imageUrl
  });
  // mongoose의 save 메소드를 사용함
  product
    .save()
    .then(result => {
      console.log('Created Product');
      res.redirect('/admin/products');
    })
    .catch(err => {
      console.log(err);
    });
};
```

### 모든 제품 가져오기

- mongoose의 find 메소드를 사용

```jsx
exports.getProducts = (req, res, next) => {
	// find 메소드로 전체 조회
	// cursor 대신 products 배열을 반환함 -> 별도의 매핑 필요없음
  Product.find()
    .then(products => {
      console.log(products);
      res.render('shop/product-list', {
        prods: products,
        pageTitle: 'All Products',
        path: '/products'
      });
    })
    .catch(err => {
      console.log(err);
    });
};
```

### 단일 제품 가져오기

- mongoose의 findById 메소드를 사용

```jsx
exports.getProduct = (req, res, next) => {
  const prodId = req.params.productId;
  // Product.findAll({ where: { id: prodId } })
  //   .then(products => {
  //     res.render('shop/product-detail', {
  //       product: products[0],
  //       pageTitle: products[0].title,
  //       path: '/products'
  //     });
  //   })
  //   .catch(err => console.log(err));
  Product.findById(prodId)
    .then(product => {
      res.render('shop/product-detail', {
        product: product,
        pageTitle: product.title,
        path: '/products'
      });
    })
    .catch(err => console.log(err));
};
```

### 제품 업데이트하기

- getEdit → 제품 가져오기 → post 요청
    - mongoose의 save 메소드로 add와 update 요청을 같이 처리할 수 있음

```jsx
exports.postEditProduct = (req, res, next) => {
  const prodId = req.body.productId;
  const updatedTitle = req.body.title;
  const updatedPrice = req.body.price;
  const updatedImageUrl = req.body.imageUrl;
  const updatedDesc = req.body.description;

  const product = new Product(
    updatedTitle,
    updatedPrice,
    updatedDesc,
    updatedImageUrl,
    prodId
  );
  product
    .save()
    .then(result => {
      res.redirect('/admin/products');
    })
    .catch(err => console.log(err));
};
```

### 상품 삭제하기

- mongoose의 findByIdAndRemove 메소드 활용

```jsx
exports.postDeleteProduct = (req, res, next) => {
  const prodId = req.body.productId;
  Product.findByIdAndRemove(prodId)
    .then(() => {
      res.redirect('/admin/products');
    })
    .catch(err => console.log(err));
};
```

### 사용자 모델 추가하기

- 이름과 이메일, 제품 배열을 가진 사용자 스키마 정의

```jsx
const mongoose = require("mongoose");

const Schema = mongoose.Schema;

const userSchema = new Schema({
  name: {
    type: String,
    required: true,
  },
  email: {
    type: String,
    required: true,
  },
  cart: {
    // items 정의
    items: [
      {
        productId: { type: Schema.Types.ObjectId },
        quantity: { type: Number, required: true },
      },
    ],
  },
});

module.exports = mongoose.model("User", userSchema);
```

- 앱 시작시 더미 유저 생성

```jsx
mongoose
  .connect(
    ''
  .then(result => {
    const User = new User({
      name: 'Test',
      email: 'example@test.com',
      cart: {
        item: [],
      }
    });
    user.save();
    ...
```

→ 생성된 유저 ID 확인 후 설정

```jsx
app.use((req, res, next) => {
  User.findById('')
    .then(user => {
      req.user = user;
      next();
    })
    .catch(err => console.log(err));
});
```

### Mongoose에서 관계 사용하기

- User - Product 관계 설정하기

```jsx
// Product Schema 수정
const productSchema = new Schema({
	...
  userId: {
    type: Schema.Types.ObjectId,
    // 참조 모델
    ref: 'User'
  }
// User Schema 수정
const userSchema = new Schema({
	...
	 productId: {
      type: Schema.Types.ObjectId,
      ref: "Product",
      required: true,
    },
// Controller -> postAdd에 userId 추가
exports.postAddProduct = (req, res, next) => {
...
 const product = new Product({
    title: title,
    price: price,
    description: description,
    imageUrl: imageUrl,
    // mongoose가 user 객체 전체를 저장함
    userId : req.user._id,
  });
```
