# Impala

## 1. impala 特点

1. Impala作为老牌的SQL解析引擎，其面对即席查询(Ad-Hoc Query)类请求的稳定性和速度在工业界得到过广泛的验证
2. 没有存储，负责解析sql
3. 定位类似hive，关注即席查询sql快速解析
4. 长sql还是hive更合适
5. group by sql 使用内存计算，建议内存128G以上  **Hive使用MR，效率低，稳定性好**