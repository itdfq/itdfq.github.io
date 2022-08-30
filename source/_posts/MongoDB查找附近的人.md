---
title: MongoDB查找附近的人
top: false
cover: false
toc: true
mathjax: true
date: 2021-08-24 13:30:33
password:
summary: MongoDB查找附近的人
tags:
- 原创
- sql
categories:
- java
---
## 方式一
```java
db.getCollection("places").find()

//创建集合
 
db.createCollection("places")

//#插入数据
db.places.insert({
    name: "Central Park",
   location: { type: "Point", coordinates: [ -73.97, 40.77 ] },
   category: "Parks"
} );

db.places.insert({
   name: "Sara D. Roosevelt Park",
   location: { type: "Point", coordinates: [ -73.9928, 40.7193 ] },
   category: "Parks"
} );

db.places.insert({
   name: "Polo Grounds",
   location: { type: "Point", coordinates: [ -73.9375, 40.8303 ] },
   category: "Stadiums"
} );


//#在location字段上创建索引
db.places.createIndex( { location: "2dsphere" } )

//以下查询使用$near操作返回距离指定GeoJSON至少1000米且最远5000米的数据，并按从最近到最远的顺序排序：

db.places.find(
   {
     location:
       { $near:
          {
            $geometry: { type: "Point",  coordinates: [ -73.9667, 40.78 ] },
            $minDistance: 1000,
            $maxDistance: 5000
          }
       }
   }
)

//按照距离指定最近的
db.runCommand(
   {
     geoNear: "places",
     near: { type: "Point", coordinates: [ -73.9667, 40.78 ] },
     spherical: true,
     query: { category: "Parks" }
   }
)
```
## 方式二

```java
db.user.insertMany([
 {'name':'杨帅哥', 'address':'江西省南昌市青山湖区市场和质量监督管理局', 'gender':1, loc:[115.993121,28.676436]},
 {'name':'王美眉', 'address':'江西省南昌市青山湖区创新一路职位小厨', 'gender':0, loc:[116.000093,28.679402]},
 {'name':'张美眉', 'address':'江西省南昌市青山湖区紫阳大道1916号', 'gender':0, loc:[115.999967,28.679743]},
 {'name':'李美眉', 'address':'江西省南昌市青山湖区云中城', 'gender':0, loc:[115.995593,28.681632]},
 {'name':'彭美眉', 'address':'江西省南昌市青山湖区北京东路1666号', 'gender':0, loc:[115.975543,28.679509]},
 {'name':'赵美眉', 'address':'江西省南昌市青山湖区市场一路大润发', 'gender':0, loc:[115.968428,28.669368]},
 {'name':'廖美眉', 'address':'江西省南昌市南昌县奥林匹克中心', 'gender':0, loc:[116.035262,28.677037]},
 {'name':'余帅哥', 'address':'江西省南昌市南昌县科技学院瑶湖校区', 'gender':1, loc:[116.02477,28.68667]},
 {'name':'吴帅哥', 'address':'江西省南昌市青山湖区创新一路母婴店', 'gender':1, loc:[116.002384,28.683865]},
 {'name':'何帅哥', 'address':'江西省南昌市青山湖区紫阳大道2999号', 'gender':1, loc:[116.000821,28.68129]},
])

//设置索引
db.user.createIndex({'loc':"2d"})

//查询附近2000米的人
db.user.aggregate({
    $geoNear:{
        near: [115.999567,28.681813], // 当前坐标
        spherical: true, // 计算球面距离
        distanceMultiplier: 6378137, // 地球半径,单位是米,那么的除的记录也是米
        maxDistance: 1000/6378137, // 过滤条件2000米内，需要弧度
        distanceField: "distance" // 距离字段别名
    }
})
```
