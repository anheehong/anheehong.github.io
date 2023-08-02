---
title: mongoDB document shell(3)
author: heehong
date: 2023-07-23 19:00:00 +0800
categories: [mongoDB]
tags: [데이터베이스, noSql, mongoDB, 몽고디비 인 액션]
published: true
---


### 데이터베이스 정보 얻기

```javascript
show dbs
```
시스템상의 모든 데이터베이스를 보여준다.

```javascript
show collections
```
현재 사용 중인 데이터베이스에서 정의된 모든 컬렉션을 보여준다.

```javascript
db.stats()
```
데이터베이스와 컬렉션에 대해서 좀 더 하위 계층의 정보를 얻기 위해서는 stats()명령이 유용하다.

```plain
{
  db: 'tutorial',
  collections: 1,
  views: 0,
  objects: 20000,
  avgObjSize: 31,
  dataSize: 620000,
  storageSize: 237568,
  indexes: 2,
  indexSize: 413696,
  totalSize: 651264,
  scaleFactor: 1,
  fsUsedSize: 278787207168,
  fsTotalSize: 994662584320,
  ok: 1
}
```

```javascript
db.numbers.stats()
```

### 명령어가 동작하는 방식

```javascript
db.stats()
```
stats() 메서드는 커맨드 호출 메서드를 감싸고 있는 helper 메서드다.

```javascript
db.runCommand ( {dbstats: 1} )
```
stat() 명령과 동일하다.
> 일반적으로 runCommand 메서드에 매개변수로서 명령을 도큐먼트로 정의하여 넘겨주면 어떤 명령이라도 간단히 실행할 수 있다.


```javascript
db.runCommand
```
괄호가 없이 실행시키면 메서드의 내부를 볼 수 있다.

```plain
[Function: runCommand] AsyncFunction {
  apiVersions: [ 1, Infinity ],
  returnsPromise: true,
  serverVersions: [ '0.0.0', '999.999.999' ],
  topologies: [ 'ReplSet', 'Sharded', 'LoadBalanced', 'Standalone' ],
  returnType: { type: 'unknown', attributes: {} },
  deprecated: false,
  platforms: [ 'Compass', 'Browser', 'CLI' ],
  isDirectShellCommand: false,
  acceptsRawInput: false,
  shellCommandCompleter: undefined,
  help: [Function (anonymous)] Help
}
```