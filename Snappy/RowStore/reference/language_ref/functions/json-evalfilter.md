## JSON_EVALPATH
将JSON路径表达式应用于JSON列，从JSON列返回属性值。

您可以SelectItem在查询中使用此函数来从JSON列中项目属性值。根据JSON路径是否返回属性值，将函数包含在WHERE子句中，其中包含一个IS NOT NULL或IS NULL表达式来过滤查询结果。



### 语法

<code>
JSON_EVALPATH (json-column, json-path-expression )
</code>

json-column 
JSON数据类型的表列的名称。请参阅数据类型。

json-path-expression应用于引用列
的字符串格式的JSON路径（类似于XPath for XML数据）。路径可以使用点符号或括号符号以及JSONPath中描述的其他语法元素--Xpath for JSON。

RowStore支持使用&&或|| json-path-expression中的逻辑运算符。但是，您不能在同一表达式中同时使用两个运算符。

函数的结果是字符串格式的属性值。要使用WHERE子句中的函数来过滤结果，请使用IS NOT NULL或IS NULL表达式来测试JSON路径是否匹配列中的任何值。

### 示例

<code>
注意：在查询JSON列之前，请使用`snappy -shell` MaximumDisplayWidth命令增加显示宽度。这有助于确保查询结果不被截断。
</code>

使用示例JSON数据，提取一个或多个属性值：

<code>
-- Create the sample table with JSON column
CREATE TABLE t1 (col1 INT, col2 JSON) PERSISTENT; 

–- Insert sample JSON data
INSERT INTO t1 VALUES ( 1, 
'{ "store": {
    "book": [
      { "category": "reference",
        "author": "Nigel Rees",
        "title": "Sayings of the Century",
        "price": 8.95
      },
      { "category": "fiction",
        "author": "Evelyn Waugh",
        "title": "Sword of Honour",
        "price": 12.99
      },
      { "category": "fiction",
        "author": "Herman Melville",
        "title": "Moby Dick",
        "isbn": "0-553-21311-3",
        "price": 8.99
      },
      { "category": "fiction",
        "author": "J. R. R. Tolkien",
        "title": "The Lord of the Rings",
        "isbn": "0-395-19395-8",
        "price": 22.99
      }
    ],
    "bicycle": {
      "color": "red",
      "price": 19.95
    }
  }
}' );

-- Return the authors of all books in the store
SELECT json_evalPath(col2, '$.store.book[*].author') FROM T1 WHERE COL1 = 1;
1                                                                                               ----------------------------------------------------------------------------------------------------
Nigel Rees,Evelyn Waugh,Herman Melville,J. R. R. Tolkien

-- Return all authors
SELECT json_evalPath(col2, '$..author') FROM T1 WHERE COL1 = 1;
1                                                                                               ----------------------------------------------------------------------------------------------------
Nigel Rees,Evelyn Waugh,Herman Melville,J. R. R. Tolkien

-- Return all items in the store (books, as well as the bicycle)
SELECT json_evalPath(col2, '$.store.*') FROM T1 WHERE COL1 = 1;
1                                                                                               ----------------------------------------------------------------------------------------------------
{
  "book" : [ {
    "category" : "reference",
    "author" : "Nigel Rees",
    "title" : "Sayings of the Century",
    "price" : 8.95
  }, {
    "category" : "fiction",
    "author" : "Evelyn Waugh",
    "title" : "Sword of Honour",
    "price" : 12.99
  }, {
    "category" : "fiction",
    "author" : "Herman Melville",
    "title" : "Moby Dick",
    "isbn" : "0-553-21311-3",
    "price" : 8.99
  }, {
    "category" : "fiction",
    "author" : "J. R. R. Tolkien",
    "title" : "The Lord of the Rings",
    "isbn" : "0-395-19395-8",
    "price" : 22.99
  } ],
  "bicycle" : {
    "color" : "red",
    "price" : 19.95
  }
}


-- Return the price of all items in the store
SELECT json_evalPath(col2, '$.store..price') FROM T1 WHERE COL1 = 1;
1                                                                                               ----------------------------------------------------------------------------------------------------
8.95,12.99,8.99,22.99,19.95

-- Return all attributes for the third book
SELECT json_evalPath(col2, '$..book[2]') FROM T1 WHERE COL1 = 1;
1                                                                                               ----------------------------------------------------------------------------------------------------
{ "category" : "fiction", "author" : "Herman Melville", "title" : "Moby Dick", "isbn" : "0-553-21311-3", "price" : 8.99 }

-- Return attributes for the last book in the order
SELECT json_evalPath(col2, '$..book[(@.length-1)]') FROM T1 WHERE COL1 = 1;
1                                                                                               ----------------------------------------------------------------------------------------------------
{
  "category" : "fiction",
  "author" : "J. R. R. Tolkien",
  "title" : "The Lord of the Rings",
  "isbn" : "0-395-19395-8",
  "price" : 22.99
}

-- Return attributes for the first two books
SELECT json_evalPath(col2, '$..book[0,1]') FROM T1 WHERE COL1 = 1;
-- or
SELECT json_evalPath(col2, '$..book[:2]') FROM T1 WHERE COL1 = 1;
1                                                                                               ----------------------------------------------------------------------------------------------------
{
  "category" : "reference",
  "author" : "Nigel Rees",
  "title" : "Sayings of the Century",
  "price" : 8.95
},
{
  "category" : "fiction",
  "author" : "Evelyn Waugh",
  "title" : "Sword of Honour",
  "price" : 12.99
}

-- Return attributes for all books with an ISBN number
SELECT json_evalPath(col2, '$..book[?(@.isbn)]') FROM T1 WHERE COL1 = 1;
1                                                                                               ----------------------------------------------------------------------------------------------------
{
  "category" : "fiction",
  "author" : "Herman Melville",
  "title" : "Moby Dick",
  "isbn" : "0-553-21311-3",
  "price" : 8.99
},
{
  "category" : "fiction",
  "author" : "J. R. R. Tolkien",
  "title" : "The Lord of the Rings",
  "isbn" : "0-395-19395-8",
  "price" : 22.99
}

-- Return all books costing less than 10
SELECT json_evalPath(col2, '$..book[?(@.price<10)]') FROM T1 WHERE COL1 = 1;
1                                                                                               ----------------------------------------------------------------------------------------------------
{
  "category" : "reference",
  "author" : "Nigel Rees",
  "title" : "Sayings of the Century",
  "price" : 8.95
},
{
  "category" : "fiction",
  "author" : "Herman Melville",
  "title" : "Moby Dick",
  "isbn" : "0-553-21311-3",
  "price" : 8.99
}

</code>

在JSON路径中使用逻辑运算符：

<code>
注意：请记住，您不能同时使用&&和|| 运算符在同一个查询中
</code>

<code>
-- Use the && operator to return fiction books costing greater than 10
1                                                                                               ----------------------------------------------------------------------------------------------------
{
  "category" : "fiction",
  "author" : "Evelyn Waugh",
  "title" : "Sword of Honour",
  "price" : 12.99
},
{
  "category" : "fiction",
  "author" : "J. R. R. Tolkien",
  "title" : "The Lord of the Rings",
  "isbn" : "0-395-19395-8",
  "price" : 22.99
}

-- Use the || operator to return reference books or books costing greater than 10
1                                                                                               ----------------------------------------------------------------------------------------------------
{
  "category" : "reference",
  "author" : "Nigel Rees",
  "title" : "Sayings of the Century",
  "price" : 8.95
},
{
  "category" : "fiction",
  "author" : "Evelyn Waugh",
  "title" : "Sword of Honour",
  "price" : 12.99
},
{
  "category" : "fiction",
  "author" : "J. R. R. Tolkien",
  "title" : "The Lord of the Rings",
  "isbn" : "0-395-19395-8",
  "price" : 22.99
} 
</code>

在WHERE子句中使用该函数：

<code>
-- Return all authors from the JSON row where a book costs more than 20.
SELECT json_evalPath(col2, '$..author') from T1 WHERE json_evalPath(col2, '$..book[?(@.price>20)]' ) IS NOT NULL;
1                                                                                               ----------------------------------------------------------------------------------------------------
Nigel Rees,Evelyn Waugh,Herman Melville,J. R. R. Tolkien

</code>

请注意，上述声明不仅返回费用超过20的书籍的作者，尽管在首次读取SELECT语句时可能会出现这种情况。WHERE子句中的JSON路径仅对应用第一个JSON路径的表行进行过滤。这里使用的示例表有一行，所以WHERE子句的例子是Limited。但是，如果该表包含许多有JSON存储数据的行，则上述WHERE中的函数将确保在查询中仅选择具有20或更高价格的图书的行。然后，第一个函数中的JSON路径将应用于每个所选行中的JSON存储数据。


























































