------

# es

> 作者：hou
------

## 1.1 es是什么
- **速度快**：是近实时的搜索平台。
- **分布式**：具备分布式的本质特质，可扩展性强。
- **功能丰富**：包含数据汇总、索引生命周期管理等一系列功能。

## 1.2 es与其他搜索引擎对比
### 1.2.1 Solr
- ES 适合​​实时性高、动态扩展​​的场景（如日志分析）。
    - 架构去中心化
    - 查询灵活
- Solr 适合​​稳定、企业级​​搜索（如电商商品搜索）。
    - 依赖zookeeper
    - 延迟高
文档中心的搜索需求往往比较复杂，用户可能需要根据不同的条件进行搜索。
ES 提供了丰富的查询语句，能够满足各种复杂的搜索需求。
相比之下，Solr 虽然适合稳定、企业级搜索，
但依赖 Zookeeper 进行管理，架构相对复杂，且存在延迟高的问题，不太能满足文档中心对实时性和灵活性的要求。

### 1.2.2 ClickHouse
主要用于 ​​高速分析查询，日志分析，强项是大规模数据扫描

## 1.3 es常用语句
布尔查询 (bool query) 核心子句
must​​	必须匹配的条件（相当于 AND），参与相关性评分，无缓存
​​filter​​	必须匹配的条件，但不影响评分，性能更高（利用缓存）	
​​should​​	应该匹配的条件（相当于 OR），满足越多得分越高（常与 minimum_should_match 配合）
​​must_not​​	必须不匹配的条件（相当于 NOT），不参与评分

常用查询子句
match 分词查询
term 精确查询
terms
range
exists
prefix

复合查询
constant_score  给filter查询固定分数
dis_max 取多个查询中最高分（不累加）
function_score 自定义评分函数
