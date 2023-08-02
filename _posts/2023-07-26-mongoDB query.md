---
title: mongoDB query
author: heehong
date: 2023-07-26 19:00:00 +0800
categories: [mongoDB]
tags: [데이터베이스, noSql, mongoDB, 몽고디비 인 액션]
published: true
---

### find vs findOne
```javascript
db.category.find()
db.category.findOne()
```
findOne 은 도큐먼트를 반환하는 반면에 find는 커서 객체를 반환한다.
findOne 쿼리에 매칭되는 항목이 여러 개라면 컬렉션에 존재하는 도큐먼트 중 자연 정렬 상으로 가장 첫번째 항목을 반환할 것이다.

#### skip, limit 그리 쿼리 옵션
```javascript
db.category.find().skip(0).limit(12)
```
skip과 limit는 질의 뒤에 호출된 것처럼 보이지만, 정렬과 제한 매개변수는 질의와 함께 전달되고 mongoDB 서버에서 처리하게 된다.
구문 패턴은 메서드 체이닝 이라 부르며, 쿼리 작성을 쉽게 하기 위한 것이다.

### 부분 매칭 쿼리
```javascript
db.users.find ({'last_name': 'Banker'})
```
이름이 정확히 일치해야 한다는 제한이 있다.

```javascript
db.users.find ({'last_name': /^Ba/})
```
사용자의 성이 Ba 로 시작하는 것을 검색한다.


### 셀렉터 매칭
```javascript
db.users.find({'last_name': "Banker"})
db.users. find ({'first_name': "Smith", birth_year: 1975})
```
쿼리 조건 함수는 논리적 AND 이다. 모든 텍스트 문자열 일치는 대소문자를 구분한다. 대소문자를 구분하지 않는 일치를 수행해야 하는 경우 정규식 용어 사용을 고려해야 한다.

#### 범위 쿼리 연산자
$lt : ~ 보다 작은
$gt : ~ 보다 큰
$lte : ~ 보다 작거나 같은
$gte : ~ 보다 크거나 같은

```javascript
db.users.find('birth_year': {'$gte': 1985, '$lte': 2015})
```

#### 집합 연산자
연산자에 대한 predicate로 하나 혹은 그 이상의 값의 리스트를 받으며, 이를 연산자라 부른다. 

'$in' :  어떤 인수든 하나라도 참고 집합에 있는 경우 일치
'$all' : 모든 인수가 참고 집합에 있고 배열이 포함된 도큐먼트에서 사용되는 경우 일치
'$nin' : 그 어떤 인수도 참고 집합에 있지 않을 경우 일치

```javascript
db.products.find({
    'main_cat_id': { '$in': [
        ObjectId('6a5b1476238d3b4dd5000048'), 
        ObjectId('6a5b1476238d3b4dd5000051'), 
        ObjectId('6a5b1476238d3b4dd5000057')
    ]}
})
```
어느 하나 라도 속해 있는 모든 상품을 찾는다.
'$in' 연산자는 하나의 속성에 대해 일종의 논리합으로 볼 수 있다.
'$in' 연산자는 ID 리스트와 함께 자주 쓰인다.


```javascript
db.products.find({'details.color': {'snin': ["black", "blue" ]}})
```
'$nin' 은 일치되는 것이 없는 도큐먼트를 찾아준다. 색상이 검정색이나 파란색이 아닌 모든 상품을 찾아준다.

```javascript
db.products.find({'tags': '$all': ["gift", "garden" }})
```
'$all' 은 해당 조건이 검색 키와 일치하는 도큐먼트를 찾아준다. 함께 태그된 모든 상품을 찾기 원한다면 다음과 같이 사용한다.

#### 부울 연산자

'$ne' : 인수가 요소와 같지 않은 경우 일치
'$not' : 일치 결과를 반전시킴
'$or' : 제공된 검색어 집합 중 하나라도 true 인 경우 일치
'$nor' : 제공된 검색어 집합 중 그 어떤 것도 true 가 아닌 경우 일치
'$and' : 제공된 검색어 집합이 모두 true 인 경우 일치
'$exist' : 요소가 도큐먼트 안에 존재할 경우 일치

```javascript
db.products.find({'details.manufacturer': 'Acme', tags: {$ne: "gardening"}})
```
같지 않음을 의미하는 '$ne' 연산자는 최소한 하나 이상의 다른 연산자와 조합해서 사용하면 좋다. 그렇지 않으면 인덱스를 이용할 수 없으므로
비효율적이 될 수 있다.

```javascript
db.users.find({'age': {'$not': {'$lte': 30}}})
```
'$ne'는 지정한 값이 아닌 경우를 찾아주지만, '$not'연산자는 다른 연산자나 정규표현식 쿼리로부터 얻은 결과의 여집합을 반환한다.
> 대부분의 쿼리 연산자는 부정의 의미를 갖는 연산자가 있다.


```javascript
db.products.find({'details.color': {$in: ['blue', 'Green']}})
```
색상이 파란색이나 초록색인 상품을 찾는 쿼리이다.

```javascript
db.products.find({ '$or': [
        {'details.color': 'blue'}, 
        {'details.manufacturer': 'Acme'}
    ]
})
```
ACME 가 제조하거나 색상이 파란색인 상품을 모두 찾기 위해서는 '$or' 사용해야 한다.


```javascript
db.products.find({
    Sand: [ {
                tags: {sin: ['gift', 'holiday']}
            },
            {
                tags: {$in: ['gardening', 'landscaping']}
            }   
        ]
})
```
하나 이상의 키를 갖는 질의 셀렉터는 각 조건을 AND 로 묶는 것으로 해석한다.
gift 나 holiday 중 하나로 태그되고 gardening 이나 landscaping 중 하나로 태그된 모든 상품을 검색한다.


```javascript
db.products.find({'details.color': {$exists: false}})
db.products.find({'details.color': {$exists: true}})
```
도큐먼트가 특정 키를 가지고 있는지 질의할 때 사용한다.
필드가 도큐먼트에 존재하는지 여부를 확인한다.

#### 서브 도큐먼트 매칭

```plain
{
    _id: ObjectId("4c4b1476238d3b4dd5003981"),
    slug: "wheel-barrow-9092",
    sku: "9092",
    details: {
        model_num: 4039283402, 
        manufacturer: "Acme",
        manufacturer_id: 432, 
        color: "Green"
    }
}
```
```javascript
db.products.find({'details.manufacturer': "Acme"});
```
acme가 제조한 모든 상품을 찾으려면 다음과 같다.

```plain
{
    _id: ObjectId ("4c4b1476238d3b4dd5003981"), 
    slug: "wheel-barrow-9092",
    sku: "9092",
    details: {
        model_num: 4039283402, 
        manufacturer: {
            name: "Acme", 
            id: 432
        },
        color: "Green"
    }
}
```
```javascript
db.products.find({'details.manufacturer.id':432});
```

```plain
{
    _id: {
        sym: 'GOOG'
        date: 20101005
    J,
    open: 40.23, 
    high: 45.50,
    low: 38.81, 
    close: 41.22
}
```
```javascript
db.ticks.find({'_id': {'sym': 'GOOG', 'date': 20101005}})
```

```javascript
db.ticks.find({'_id' : {'date':20101005, 'sym' : 'GOOG'}})
```
다음 쿼리는 도큐먼트를 찾지 못한다.

#### 배열
'$elemMatch' : 제공된 모든 조건이 동일한 하위 도큐먼트에 있는 경우 일치
'$size' : 배열 하위 도큐먼트의 크기가 제공된 리터럴 값과 같으면 일치

```plain
{
    _id: ObjectId ("4c4b1476238d3b4dd5003981"),
    slug: "wheel-barrow-9092",
    sku: "9092",
    tags: ["tools", "equipment", "soil"] }
}
```

```javascript
db.products.find({tags: "soil"})
```

```javascript
db.products.ensureIndex({tags: 1}) 
db.products.find({tags: "soil"}).explain()
```
tags 필드에 대한 인덱스를 이용할 수 있다. 필요한 인덱스를 생성하고 explain()으로 쿼리를 실행하면 B-트리 커서가 사용된다.

```javascript
db.products.find({'tags.0': "soil"})
```
닷 표기법을 써서 배열 내의 특정 위치에 있는 값에 대해 질의할 수 있다.


```plain
{
    _id: ObjectId("4c4b1476238d3b4dd5000001"),
    username: "kbanker"
    addresses: [
        {
            name: "home",
            street: "588 5th Street",
            city : "Brooklyn",
            state: "NY",
            zip: 11215
        },
        {
            name: "work",
            street: "1 E. 23rd Street", 
            city: "New York"
            state: "NY", 
            zip: 10010     
        }
    ]
}
```

```javascript
db.users.find({'addresses.0.state': "NY"})
```
첫번째 주소가 뉴욕인 것을 검색한다.

```javascript
db.users.find ({'addresses.state': "NY"})
```
여러 주소 중에 어느 것이라도 주가 뉴욕이면 그 도큐먼트를 반환한다.

```javascript
db.users.find ({'addresses.name': 'home', 'addresses.state': 'NY'})
```
거주지 주소가 뉴욕이 모든 사용자를 찾으려고 할 때, 해당 쿼리는 필드 지시자가 같은 주소에 대해 국한되지 않는다는 점이다.

```javascript
db.users.find({ 
    'addresses': {
        'SelemMatch': {
            'name': 'home', 
            'state': 'NY'
        } 
    }
})
```
어떤 한 주소가 이름이 home이고 주가 NY 이려면 해당 쿼리를 사용해야 한다.
여러 개의 조건을 하나의 서브도큐먼트에 대해 제한하려면 '$elemMatch' 연산자를 쓰면 된다.
논리적으로 '$elemMatch' 는 서브도큐먼트에서 두 개 이상의 속성이 매치되는 것을 찾는 경우에만 사용된다.

#### 배열 크기별 질의
```javascript
db.users.find({'addresses': {$size: 3}})
```
'$size'연산자를 통해 배열의 크기에 대해 질의 할 수 있다.
인덱스를 사용할 수 없고 정확히 일치하는 쿼리에만 제한된다. 크기의 범위를 지정할 수 없다.


### 정규표현식
'$regex' : 요소를 제공된 정규 표현식과 맞춰본다.
정규 표현식 쿼리는 인덱스를 사용할 수 없고, 대부분의 선택자보다 실행시간이 오래 걸린다.

```javascript
db.reviews.find({
    'user_id' : ObjectId("4c4b1476238d3b4dd5000001"),
    'text': /best|worst/i 
})
```
i 플래그를 사용해서 대소문자를 구분하지 않는다.,
해당 사용자의 상품평 중에서 best 나 worst라는 단어가 들어간 것을 검색한다.

```javascript
db.reviews.find({
    'user_id': ObjectId ("4c4b1476238d3b4dd5000001"), 
    'text': {
        '$regex': "best|worst",
        'soptions': "i"
        }
})
```
i 플래그를 사용하면 인덱스를 사용할 수 없다는 것이다.

### 그 밖의 쿼리 연산자
'$mod[(몫),(결과)]' : 몫으로 나눈 결과가 요소와 일치할 경우 일치
'$type' : 요소의 타입이 명시된 BSON 타입과 일치할 경우 일치
'$text' : 텍스트 인덱스로 인덱싱된 필드의 내용에 대해 텍스트 검색을 수행할 수 있도록 해준다.

```javascript
db.orders.find ({subtotal: {$mod: [3, 0]}})
```
첫번째 값은 나누는 값이고, 두번째 값은 나눈 후의 나머지 값이다. 
해당 쿼리는 주문 액수를 3으로 나누면 나머지가 0이 되는 모든 주문을 검색하라는 뜻이다.

'$type' 연산자는 값이 어떤 BSON 타입인지 확인해준다.


### 쿼리 옵션

#### 프로젝션
프로젝션은 결과값 도큐먼트에 대해 반환할 필드를 지정하는 데 사용한다.
도큐먼트의 수가 많을 때 프로젝션을 사용하면 네트워크 지연과 deserialization 에 들어가는 비용을 줄일 수 있다.

'$slice' : 반환되는 도큐먼트의 부분집합을 선택한다.

```javascript
db.users.find({}, {'username': 13)
```
_id 필드는 디폴트로 항상 포함된다.

```javascript
db.users.find({}, {'addresses': 0, 'payment_methods': 0})
```
프로젝션 도큐먼트에서 값을 0으로 설정하여 포함시키지 말아야 할 필드를 지정할 수 있다.

```javascript
db.products.find({}, {'reviews': {$slice: 12}})
db.products.find({}, {'reviews': {$slice: -5}})
```
배열의 값을 어떤 범위 내에서 정할 수도 있다.
처음 12개의 리뷰 혹은 마지막 5개의 리뷰를 가져오려면 해당과 같다.

```javascript
db.products.find({}, {'reviews': {slice: [24, 12]}})
```
skip 과 limit를 나타낸다. 처음 24개의 리뷰를 제외하고 난 후 가져올 리뷰를 12개로 제한한다.

```javascript
db.products.find({}, {'reviews': {'$slice': [24, 12]}, 'reviews.rating': 1})
```
필드를 포함하거나 배제하는 기능과 함께 사용될 수 있다.

### 정렬
하나 혹은 그 이상의 필드에 대해 오름차순이나 내림차순으로 정렬할 수 있다.

```javascript
db.reviews.find({}).sort({'rating': -1})
```
리뷰를 평점 높은 것부터 낮은 것까지 내림차순으로 정렬하는 쿼리이다.

```javascript
db.reviews.find({}).sort({'helpful_votes':-1, 'rating':-1})
```
추천수와 평점순으로 정렬할 수 있다.
