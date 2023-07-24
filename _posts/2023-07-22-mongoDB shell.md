---
title: mongoDB document shell
author: heehong
date: 2023-07-22 19:00:00 +0800
categories: [mongoDB]
tags: [데이터베이스, noSql, mongoDB, document, 몽고디비 인 액션]
published: true
---

## mongoDB shell 사용하기

### 데이터베이스 변경

```javascript
use tutorial
```
데이터베이스를 생성하지도 않았는데 어떻게 그 데이터베이스로 변경할 수 있을까?
사실 mongoDB에서는 데이터베이스를 만드는 것이 필요 없다. 데이터베이스와 컬렉션은 도큐먼트가 처음 입력될 때 생성된다. 


### 데이터 저장

```javascript
db.users.insertOne({
    username: "smith"
})
```
이 때 tutorial 데이터베이스와 users 컬렉션이 만들어진다.

### 데이터 조회

```javascript
db.users.find()
```
``` json
[
    { "_id": "ObjectId(64ba0b08557e718ed108e563)", "username": "smith" } 
]
```
_id는 도큐먼트의 프라이머리 키이다.
도큐먼트가 생성될 때 이 필드가 없으면 mongoDB 객체 id 라는 특별한 값을 생성해서 도큐먼트에 자동으로 추가한다.
컬렉션 내에서 객체 id는 고유한 값을 가져야만 하는데, 이 필드에 대해 유일하게 요구되는 사항이다.

![scaling.png](/assets/img/post//2023-07-22-mongoDB shell/id_rule.png)
id들이 서버가 아닌 드라이버에서 생성된다는 것을 이해하는 것이 중요하다.
서버상에서 프라이머리 키를 증가시켜 결국 서버에서 병목현상을 일으키게 되는 기존의 RDBMS 와는 다른점이다.
id 를 통해 도큐먼트의 생성시간을 알 수 있다.

```javascript
db.users.insertOne({
    username: "jones"
})
```
``` json
{
  "acknowledged": true,
  "insertedId": "ObjectId(64ba13c2557e718ed108e564)"
}
```

```javascript
 db.users.countDocuments()
```
``` json
2
```

```javascript
db.users.find()
```
``` json
[
  { "_id": "ObjectId(64ba0b08557e718ed108e563)", "username": "smith" },
  { "_id": "ObjectId(64ba13c2557e718ed108e564)", "username": "jones" }
]
```

```javascript
db.users.find({username: "jones"})
```
존재하는 모든 도큐먼트에 대해 username 키의 값이 문자적으로 일치하는지 여부를 검사한다.
db.user.find() 와 db.users.find({}) 는 동일하다.

### 데이터 업데이트

```javascript
db.users.updateOne({username: "smith"}, {$set: {country: "canada"}})
```

##### 주의사항
```javascript
db.users.updateOne({username: "smith"}, {country: "canada"})
```
도큐먼트는 오직 country필드만을 포함하는 것으로 대체된다. username 필드는 제거된다.
_id는 같지만, 데이터는 업데이트 문에서 대체된다. 만약 전체 도큐먼트를 대체하는 것이 아니라 필드를 추가하거나 값을 설정하기를 원한다면 반드시 $set 연산자를 사용하자.

현재는 해당 에러로 불가능하다.
> MongoInvalidArgumentError: Update document requires atomic operators 


```javascript
db.users.updateOne({username: "smith"}, {$unset: {country: "canada"}})
```
$unset 을 이용해 해당 값을 지울 수 있다.


```javascript
db.users.updateOne({username: "smith"}, 
    {
        $set: {
            favorites: {
                cities : ["Chicago", "Cheyenne"],
                movies: ["Casablanca", "For a Few Dollars More", "The Sting"]
            }
        }
    })
```

```javascript
db.users.updateOne({username: "jones"}, 
    {
        $set: {
            favorites: {
                movies: ["Casablanca", "Rocky"]
            }
        }
    })
```

```javascript
db.users.find().pretty()
```
pretty() 는 실제로 cursor.pretty() 이고, 결과를 가독성 있는 형태로 보여 주기 위해 커서를 설정하는 방법이다.

```json
[
  {
    "_id": "ObjectId(64ba0b08557e718ed108e563)",
    "username": "smith",
    "favorites": {
      "cities": [ "Chicago", "Cheyenne" ],
      "movies": [ "Casablanca", "For a Few Dollars More", "The Sting" ]
    }
  },
  {
    "_id": "ObjectId(64ba13c2557e718ed108e564)",
    "username": "jones",
    "favorites": { "movies": [ "Casablanca", "Rocky" ] }
  }
]
```

```javascript
 db.users.find ({"favorites.movies": "Casablanca" })
```


```javascript
db.users.updateOne({ "favorites.movies": "Casablanca"}, 
    { $addToSet: {"favorites.movies" : "The Maltese Falcon"}},
    false,
    true
    )
```
$addToSet 연산자를 이용해서 리스트에 추가한다.
세번째 인자는 update & insert 가 허용되는지 아닌지를 조절한다. 해당되는 도큐먼트가 존재하지 않을 때 update 연산이 도큐먼트를 입력해야 하는지 아닌지를 말해주고,
업데이트가 연산자 업데이트냐 아니면 대체를 위한 업데이트냐에 따라 다른 행동을 취한다.
네번째 인자는 이 업데이트가 다중 업데이트, 즉 하나 이상의 도큐먼트에 대해 업데이트가 이루어져야 함을 뜻한다. mongoDB에서는 쿼리 셀렉터의 조건에 맞는 첫번째 도큐먼트에 대해서만
업데이트를 하는 것이 기본 설정으로 되어 있다. 조건에 일치하는 모든 도큐먼트에 대해 업데이트 연산을 수행하려면 이것을 명시적으로 지정해야 한다. 


### 데이터 삭제

```javascript
db.foo.remove({})
```
삭제 명령에서 매개변수가 주어지지 않으면 컬렉션의 모든 도큐먼트를 지운다.

```javascript
db.users.remove({"favorites.cities": "Cheyenne"})
```
조건에 맞는 일부의 도큐먼트를 지워야 할 때는 remove 메서드에 쿼리 셀렉터를 넘겨주면 된다.
remove() 연산은 컬렉션을 지우지 않는다.  SQL에서 delete 명령어와 비슷하다.

어느 한 컬렉션을 모든 인덱스와 함께 지우려고 한다면 drop() 명령을 사용하면 된다.

```javascript
db.users.drop()
```

> 셸은 mongoDB로 작업하는 것을 훨씬 쉽게 만들어 준다.
> 위아래 방향 키를 사용하여 이전 명령어를 다시 활용할 수 있고, 컬렉션 이름과 같은 것에 자동완성 기능을 사용할 수 있다.
> 자동완성 기능은 자동완성을 수행하거나 가능성 있는 결과물을 보여주기 위해 탭 키를 사용한다.


```javascript
help
```
많은 함수가 자신을 설명하는 help 메시지를 보기 좋은 형태로 출력한다.

```javascript
db.help()
```