## 1.全文搜索
- 开源的 Elasticsearch(以下简称 Elastic)是目前全文搜索引擎的首选。
- 它可以快速地存储、搜索和分析海量数据

## 2. 安装
1. 安装[jdk](http://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html)
2. 安装[elasticsearch](https://www.elastic.co/downloads/elasticsearch)

## 3. 访问
[http://localhost:9002/](http://localhost:9200/)
```json
{
  "name" : "PC-201704292335", //
  "cluster_name" : "elasticsearch"，//集群名称
  "cluster_uuid" : "pb4rTAeoSxyLgJGtyz4fAg",//集群ID
  "version" : {
    "number" : "5.6.10",
    "build_hash" : "b727a60",
    "build_date" : "2018-06-06T15:48:34.860Z",
    "build_snapshot" : false,
    "lucene_version" : "6.6.1"
  },
  "tagline" : "You Know, for Search"
}
```
- C:\ProgramData\Elastic\Elasticsearch\config\elasticsearch.yml 配置文件

## 4.基本概念
### 4.1 节点和集群
- Elastic 本质上是一个分布式数据库，允许多台服务器协同工作，每台服务器可以运行多个Elastic实例
- 单个Elastic实例称为一个节点node。一组节点构成一个集群cluster
### 4.2 索引
- Elastic会索引所有字段，经过处理后写入一个反向索引（Inverted Index）。查找数据的时候，直接查找该索引
- Elastic 数据管理的顶层单位叫做Index(索引)，Index(即数据库)的名字必须是小写
### 4.3 文档
- Index里面单条的记录称为Document（文档）
- 许多条 Document 构成了一个Index
- Document使用JSON格式表示
### 4.4 类型（Type）
- 文档可以分组，比如学生这个Index里面，可以按性别分组（男生一组，女生一组），也可以按省份分组（北京和广东）
- 这种分组就叫做类型，它是虚拟的逻辑分组，用来过滤文档
- 不同的类型应该有相似的结构
- 每个 Index 包含了一个 Type, 7.x 版将会彻底移除 Type
## 5. 操作 Index
### 5.1 创建索引
```sh
curl -X PUT 'http://localhost:9200/student'
```
- 不能有请求体
### 5.2 删除索引
```sh
curl -X DELETE 'http://localhost:9200/student'
```
## 6.数据操作
### 6.1 新增文档
```sh
curl -X PUT 'http://localhost:9200/student/city/1' -d `
    {
        "name":"张三",
        "age": 5,
        "city": "北京"
    }
`
```
```sh
curl -X POST 'http://localhost:9200/student/city' -d `
    {
        "name": "赵六",
        "age": "7",
        "city":"江苏"
    }
`
```
### 6.2 查看文档
```sh
curl 'http://localhost:9200/student/city/1'
```
### 6.3 更新记录
```sh
curl -X PUT 'http://localhost:9200/student/city/1' -d`
    {
    "name":"张三2",
    "age":55,
    "city":"北京2"
}
`
```
### 6.4 删除文档
```sh
curl -X DELETE 'http://localhost:9200/student/city/1'
```
## 7.数据查询
### 7.1 查询全部
```sh
curl 'http://localhost:9200/student/city/_search'
```
### 7.2 全文搜索
```sh
curl 'http://localhost:9200/student/city/_search' -d `
{
  "query" : { "match" : { "name" : "李" }},
  "size":1,
  "from":1
}
`
```
### 7.3 OR
```sh
curl 'http://localhost:9200/student/city/_search' -d `
{
  "query" : { "match" : { "name" : "赵 李" }}
}
`
```
### 7.4 AND
```sh
curl 'http://localhost:9200/student/city/_search' -d `
{
  "query": {
    "bool": {
      "must": [
        { "match": { "name": "赵" } },
        { "match": { "name": "六" } }
      ]
    }
  }
}
`
```
## 8.node 中如何用
```js
var elasticsearch = require('elasticsearch');
var client = new elasticsearch.Client({
    host: 'localhost:9200',
    log: 'trace'
});
/**
client.search({
    index: 'student',
    type: 'city',
    body: {
        query: {
            match: {
                name: '赵'
            }
        }
    }
}).then(ret => {
    console.log(ret.hits.hits);
});
*/
(async function() {
    let name = Date.now();
    let id = Date.now();
    const created = await client.create({
        index: 'student',
        type: 'city',
        id,
        body: {
            name,
            age: 10
        }
    });
    console.log(created);
    const updated = await client.update({
        index: 'student',
        type: 'city',
        id,
        body: {
            doc: {
                name: name,
                age: 101
            }
        }
    });
    console.log(created);
    // shift+alt+a
    /*
    const deleted = await client.delete({
        index: 'student',
        type: 'city',
        id
    })
    */
    console.log(deleted);
}).then(ret => console.log(ret), err => console.log(err));
```
## 9.参考
- [elastic](https://www.elastic.co/cn/)
- [installation](https://www.elastic.co/downloads/elasticsearch)
- [elasticsearch](https://github.com/elastic/elasticsearch-js)
- [api](https://www.elastic.co/guide/en/elasticsearch/client/javascript-api/current/api-reference.html#api-delete)