## 18本地化 于欣雨

## 选题：基于MYSQL全文索引构建简单的中文“搜索引擎”

#### 前期调研：

  通过数值比较、范围过滤等就可以完成绝大多数我们需要的查询，但是，如果希望通过关键字的匹配来进行查询过滤，那么就需要基于相似度的查询，而不是原来的精确数值比较。全文索引就是为这种场景设计的。

  全文索引在大量的数据面前，能比 like + % 快 N 倍，速度不是一个数量级，但是全文索引可能存在精度问题。

  全文索引是基于关键词的，如何区分不同的关键词，就要用到分词。在本次作业中，打算使用百度的自然语言处理API的分词功能。

#### 实验：

##### 创建表的同时创建全文索引

  `CREATE TABLE articles (`
    `id INT UNSIGNED AUTO_INCREMENT NOT NULL PRIMARY KEY,`
    `title VARCHAR (200),`
    `body TEXT,`
    `FULLTEXT (title, body) WITH PARSER ngram`
`) ENGINE = INNODB DEFAULT CHARSET=utf8 COMMENT='文章表';`

##### 在已存在的表上创建全文索引

``create fulltext index content_tag_fulltext``

``on fulltext_test(content,tag);``

##### 或者通过 SQL 语句 ALTER TABLE 创建全文索引

`alter table fulltext_test`

`add fulltext index content_tag_fulltext(content,tag);`

##### 使用全文索引

``select * from fulltext_test`

`where match(content,tag) against('xxx xxx');`

##### 直接使用 DROP INDEX 删除全文索引

`drop index content_tag_fulltext`

`on fulltext_test;`

### 两种全文索引

#### 自然语言的全文索引

默认情况下，或者使用 in natural language mode 修饰符时，match() 函数对文本集合执行自然语言搜索，上面的例子都是自然语言的全文索引。

自然语言搜索引擎将计算每一个文档对象和查询的相关度。这里，相关度是基于匹配的关键词的个数，以及关键词在文档中出现的次数。在整个索引中出现次数越少的词语，匹配时的相关度就越高。相反，非常常见的单词将不会被搜索，如果一个词语的在超过 50% 的记录中都出现了，那么自然语言的搜索将不会搜索这类词语。上面提到的，测试表中必须有 4 条以上的记录，就是这个原因。

#### 布尔全文索引

在布尔搜索中，我们可以在查询中自定义某个被搜索的词语的相关性，当编写一个布尔搜索查询时，可以通过一些前缀修饰符来定制搜索。

MySQL 内置的修饰符，上面查询最小搜索长度时，搜索结果 ft_boolean_syntax 变量的值就是内置的修饰符，下面简单解释几个，更多修饰符的作用可以查手册

- **+** 必须包含该词
- **-** 必须不包含该词
- **>** 提高该词的相关性，查询的结果靠前
- **<** 降低该词的相关性，查询的结果靠后
- **(\*)星号** 通配符，只能接在词后面