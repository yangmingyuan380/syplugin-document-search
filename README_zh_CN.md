# 基于文档搜索（es版）

因为自己思源笔记文档较多，使用sql进行查询有时速度很慢，因此就考虑将数据同步到elasticsearch 进行查询，修改了Misuzu2027的基于文档搜索的插件。可以将数据同步到es。并且默认从es进行搜索，个人感觉速度更快些。
插件里增加了将sqlite中blocks表部分数据同步到es的功能，但是需要手动同步，而且初始时，最好运行下面python脚本同步。后续可以使用同步按钮同步。
```python
from sqlite3 import connect
from elasticsearch import Elasticsearch
from elasticsearch.helpers import bulk
from datetime import datetime  
import logging

# 连接到自己的 SQLite 数据库
conn = connect(r'***\siyuan.db')
cursor = conn.cursor()
# 执行查询
# 获取当前时间  
now = datetime.now()
# 转换为yyyymmddhhmmss格式  
formatted_time = now.strftime("%Y%m%d%H%M%S")  
cursor.execute("SELECT id, root_id,box,hash,hpath,ial, content,fcontent, type, created, updated FROM blocks WHERE created < ?", (formatted_time,))
rows = cursor.fetchall()
es = Elasticsearch('http://192.168.31.2:9200')
# 定义索引的映射
index_mappings = {
    "settings": {
    "analysis": {
      "analyzer": {
        "default": {
          "tokenizer": "ik_max_word"
        }
      }
    }
  },
  "mappings": {
    "_doc": {  # 注意：在ES 6.x中需要这样指定类型
      "properties": {
        "id": {"type": "keyword"},
        "root_id": {"type": "keyword"},
        "box": {"type": "keyword"},
        "hash": {"type": "keyword"},
        "hpath": {"type": "keyword"},
        "ial": {"type": "text"},
        "content": {"type": "text","analyzer": "ik_max_word"},
        "fcontent": {"type": "text"},
        "type": {"type": "keyword"},
        "created": {"type": "keyword"},
        "updated": {
          "type": "keyword",
          "null_value": "NULL"  
        }
      }
    }
  }
}

# 创建索引
index_name = "siyuan_blocks"
response = es.indices.create(index=index_name, body=index_mappings)

def generate_data(rows):
    for row in rows:
        yield {
            "_index": "siyuan_blocks",
            "_type": "_doc",  # 对于Elasticsearch 6.x版本，你需要提供_type; 通常使用_doc作为类型
            "_id": row[0],  # 如果你想指定文档ID的话
            "_source": {
                "id": row[0],
                "root_id": row[1],
                "box": row[2],
                "hash": row[3],
                "hpath": row[4],
                "ial": row[5],
                "content": row[6],
                "fcontent": row[7],
                "type": row[8],
                "created": row[9],
                "updated": row[10],
            },
        }
def bulk_insert(es, rows, batch_size=10000):
    start = 0
    success_count = 0
    try:
        while start < len(rows):
            batch_rows = rows[start:start+batch_size]
            actions = generate_data(batch_rows)
            success, failed = bulk(es, actions)
            success_count += success
            if failed:
                logging.warning(f"有 {len(failed)} 条文档插入失败")
            start += batch_size
            print(f"已处理 {start} 条文档...")
    except Exception as e:
        logging.error(f"批量插入过程中发生错误：{e}")
    return success_count

success_count = bulk_insert(es, rows)
print(f"成功插入 {success_count} 条文档")
```
此外因为思源删除block是硬删除，因此需要使用下面脚本在sqlite里创建删除的触发器
```sql
--- 创建删除日志表
CREATE TABLE IF NOT EXISTS delete_logs (
  id INTEGER PRIMARY KEY AUTOINCREMENT,
  deleted_at DATETIME DEFAULT CURRENT_TIMESTAMP,
  deleted_record_id TEXT
);
--- 创建删除触发器
  CREATE TRIGGER IF NOT EXISTS after_delete_blocks
  AFTER DELETE
  ON blocks
  FOR EACH ROW
  BEGIN
    INSERT INTO delete_logs (deleted_record_id)
    VALUES (OLD.id);
  END;
```

# 更新日志

* 0.6.0
  * 支持预览区高亮，支持表格定位首个匹配关键字位置
* 0.5.2
  * 优化设置“显示文档块”的逻辑，优化上下键选择搜索结果的样式。
* 0.5.1
  * 修复搜索预览超出页面，优化搜索页的结构。
* 0.5.0
  * 支持设置过滤块类型、笔记本、快属性（命名、别名等）；支持配置每页文档数量、默认展开数量
* 0.4.0
  * 支持查询块的命名、别名、备注并显示
* 0.3.0
  * 支持分页查询
* 0.2.0
  * 支持手机端
* 0.1.0
  * 支持Dock和自定义页签查询
