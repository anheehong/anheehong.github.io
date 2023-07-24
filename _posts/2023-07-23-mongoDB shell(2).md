---
title: mongoDB document shell(2)
author: heehong
date: 2023-07-23 19:00:00 +0800
categories: [mongoDB]
tags: [데이터베이스, noSql, mongoDB, document, 몽고디비 인 액션]
published: true
---

## mongoDB index, query

### 대용량 컬렉션 생성
```javascript
for(i=0; i< 20000; i++){
  db.numbers.insertOne({num: i});
}
```

```javascript
db.numbers.count()
```

```javascript
db.numbers.find()
it
```

#### 범위 쿼리
```javascript
db.numbers.find({num:500})
```

```javascript
db.numbers.find({num: { "$gt": 19995 }})
```

```javascript
db.numbers.find({num: { "$gt": 20, "$lt": 25 }})
```
greater than or equal to = $gte
less than or equal to = $lte
not equal to = $ne

### 인덱싱 explain
데이터베이스가 쿼리를 받았을 때 이를 실행하는 방법에 대해 계획을 세우는 것은 query plan 이라 부른다.
explain 은 쿼리가 사용한 인덱스가 있을 경우에 어떤 인덱스를 사용했는지를 찾아서 쿼리 경로에 대한 정보를 제공해 줌으로써 
시간이 많이 소요되는 연산을 찾아내도록 도와준다.

```javascript
db.numbers.find({num: { "$gt": 19995 }}).explain("executionStats")
```

```plain
{
  explainVersion: '1',
  queryPlanner: {
    namespace: 'tutorial.numbers',
    indexFilterSet: false,
    parsedQuery: { num: { '$gt': 19995 } },
    queryHash: 'A1E79916',
    planCacheKey: 'A1E79916',
    maxIndexedOrSolutionsReached: false,
    maxIndexedAndSolutionsReached: false,
    maxScansToExplodeReached: false,
    winningPlan: {
      stage: 'COLLSCAN',
      filter: { num: { '$gt': 19995 } },
      direction: 'forward'
    },
    rejectedPlans: []
  },
  executionStats: {
    executionSuccess: true,
    nReturned: 4,
    executionTimeMillis: 15,
    totalKeysExamined: 0,
    totalDocsExamined: 20000,
    executionStages: {
      stage: 'COLLSCAN',
      filter: { num: { '$gt': 19995 } },
      nReturned: 4,
      executionTimeMillisEstimate: 3,
      works: 20002,
      advanced: 4,
      needTime: 19997,
      needYield: 0,
      saveState: 20,
      restoreState: 20,
      isEOF: 1,
      direction: 'forward',
      docsExamined: 20000
    }
  },
  command: {
    find: 'numbers',
    filter: { num: { '$gt': 19995 } },
    '$db': 'tutorial'
  },
  serverInfo: {
    host: 'heehongui-MacBookPro.local',
    port: 27017,
    version: '6.0.6',
    gitVersion: '26b4851a412cc8b9b4a18cdb6cd0f9f642e06aa7'
  },
  serverParameters: {
    internalQueryFacetBufferSizeBytes: 104857600,
    internalQueryFacetMaxOutputDocSizeBytes: 104857600,
    internalLookupStageIntermediateDocumentMaxSizeBytes: 104857600,
    internalDocumentSourceGroupMaxMemoryBytes: 104857600,
    internalQueryMaxBlockingSortMemoryUsageBytes: 104857600,
    internalQueryProhibitBlockingMergeOnMongoS: 0,
    internalQueryMaxAddToSetBytes: 104857600,
    internalDocumentSourceSetWindowFieldsMaxMemoryBytes: 104857600
  },
  ok: 1
}
```
totalDocsExamined 스캔한 도큐먼트 수이다.
totalKeysExamined 필드는 스캔한 인덱스 엔트리의 개수이다.

스캔한 도큐먼트 수와 결과값의 큰 차이는 이 쿼리가 비효율적으로 실행되었음을 보여준다.


```javascript
db.numbers.createIndex({num: 1})
```
createIndex 메서드에 도큐먼트를 매개변수를 넘겨줌으로써 인덱스 키를 정의한다. 
{num:1} 도큐먼트는 numbers 컬렉션에 있는 모든 도큐먼트의 num 키에 대해 오름차순 인덱스를 생성함을 뜻한다.


```javascript
db.numbers.getIndexes()
```

```plain
[
  { v: 2, key: { _id: 1 }, name: '_id_' },
  { v: 2, key: { num: 1 }, name: 'num_1' }
]
```

***다시 조회해보면***
```
{
  explainVersion: '1',
  queryPlanner: {
    namespace: 'tutorial.numbers',
    indexFilterSet: false,
    parsedQuery: { num: { '$gt': 19995 } },
    queryHash: 'A1E79916',
    planCacheKey: '40DCF34C',
    maxIndexedOrSolutionsReached: false,
    maxIndexedAndSolutionsReached: false,
    maxScansToExplodeReached: false,
    winningPlan: {
      stage: 'FETCH',
      inputStage: {
        stage: 'IXSCAN',
        keyPattern: { num: 1 },
        indexName: 'num_1',
        isMultiKey: false,
        multiKeyPaths: { num: [] },
        isUnique: false,
        isSparse: false,
        isPartial: false,
        indexVersion: 2,
        direction: 'forward',
        indexBounds: { num: [ '(19995, inf.0]' ] }
      }
    },
    rejectedPlans: []
  },
  executionStats: {
    executionSuccess: true,
    nReturned: 4,
    executionTimeMillis: 2,
    totalKeysExamined: 4,
    totalDocsExamined: 4,
    executionStages: {
      stage: 'FETCH',
      nReturned: 4,
      executionTimeMillisEstimate: 0,
      works: 5,
      advanced: 4,
      needTime: 0,
      needYield: 0,
      saveState: 0,
      restoreState: 0,
      isEOF: 1,
      docsExamined: 4,
      alreadyHasObj: 0,
      inputStage: {
        stage: 'IXSCAN',
        nReturned: 4,
        executionTimeMillisEstimate: 0,
        works: 5,
        advanced: 4,
        needTime: 0,
        needYield: 0,
        saveState: 0,
        restoreState: 0,
        isEOF: 1,
        keyPattern: { num: 1 },
        indexName: 'num_1',
        isMultiKey: false,
        multiKeyPaths: { num: [] },
        isUnique: false,
        isSparse: false,
        isPartial: false,
        indexVersion: 2,
        direction: 'forward',
        indexBounds: { num: [ '(19995, inf.0]' ] },
        keysExamined: 4,
        seeks: 1,
        dupsTested: 0,
        dupsDropped: 0
      }
    }
  },
  command: {
    find: 'numbers',
    filter: { num: { '$gt': 19995 } },
    '$db': 'tutorial'
  },
  serverInfo: {
    host: 'heehongui-MacBookPro.local',
    port: 27017,
    version: '6.0.6',
    gitVersion: '26b4851a412cc8b9b4a18cdb6cd0f9f642e06aa7'
  },
  serverParameters: {
    internalQueryFacetBufferSizeBytes: 104857600,
    internalQueryFacetMaxOutputDocSizeBytes: 104857600,
    internalLookupStageIntermediateDocumentMaxSizeBytes: 104857600,
    internalDocumentSourceGroupMaxMemoryBytes: 104857600,
    internalQueryMaxBlockingSortMemoryUsageBytes: 104857600,
    internalQueryProhibitBlockingMergeOnMongoS: 0,
    internalQueryMaxAddToSetBytes: 104857600,
    internalDocumentSourceSetWindowFieldsMaxMemoryBytes: 104857600
  },
  ok: 1
}
```