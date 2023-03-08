## 1. 问题描述

```scala
//以覆盖的模式写入到Apache Phoenix
    dataFrame.write.format("jdbc")
      .mode("overwrite")
      .option("dbtable",phoenix_dbtable)
      .option("url",phoenix_jdbc_url)
      .option("user",phoenix_user)
      .option("password",phoex_password)
      .save()
```

Apache Phoenix 无论插入还是更新数据的语法都是Upsert table values(....)，Spark SQL jdbc 的save方法支持的是INSERT语法，因此当使用该方法写数据到Phoenix时会遇到语法不兼容的错误。

`spark 1.6.0`

## 2. 解决

主要设计三个类:
* SaveMode
* DataFrameWrite
* JdbcUtils

DataFrameWrite.jdbc -> JdbcUtils.saveTable -> JdbcUtils.savePartition -> JdbcUtils.insertStatement

#### 2.1 增加Upsert模式

`SaveMode`是一个枚举类，在这个类中增加`Upsert`
```java
package org.apache.spark.sql;  
  
/**  
 * SaveMode is used to specify the expected behavior of saving a DataFrame to a data source. * * @since 1.3.0 */
 
 public enum SaveMode {   
 Append,  
 Overwrite,   
 ErrorIfExists,  
 Ignore,  
 Upsert  
}
```

#### 2.2 JdbcUtils增加对Upsert模式的支持
JdbcUtils类增加saveMode全局变量
`var saveMode`

找到JdbcUtils类下面的saveTable方法，增加mode参数，并给全局变量saveMode赋值

```scala
/**  
 * Saves the RDD to the database in a single transaction. */def saveTable(  
    df: DataFrame,  
    url: String,  
    table: String,
    mode：SaveMode,
    properties: Properties = new Properties()) {  
  val dialect = JdbcDialects.get(url)  
  saveMode = mode
  val nullTypes: Array[Int] = df.schema.fields.map { field =>  
    dialect.getJDBCType(field.dataType).map(_.jdbcNullType).getOrElse(  
      field.dataType match {  
        case IntegerType => java.sql.Types.INTEGER  
        case LongType => java.sql.Types.BIGINT  
        case DoubleType => java.sql.Types.DOUBLE  
        case FloatType => java.sql.Types.REAL  
        case ShortType => java.sql.Types.INTEGER  
        case ByteType => java.sql.Types.INTEGER  
        case BooleanType => java.sql.Types.BIT  
        case StringType => java.sql.Types.CLOB  
        case BinaryType => java.sql.Types.BLOB  
        case TimestampType => java.sql.Types.TIMESTAMP  
        case DateType => java.sql.Types.DATE  
        case t: DecimalType => java.sql.Types.DECIMAL  
        case _ => throw new IllegalArgumentException(  
          s"Can't translate null value for field $field")  
      })  
  }  
  
  val rddSchema = df.schema  
  val driver: String = DriverRegistry.getDriverClassName(url)  
  val getConnection: () => Connection = JDBCRDD.getConnector(driver, url, properties)  
  df.foreachPartition { iterator =>  
    savePartition(getConnection, table, iterator, rddSchema, nullTypes)  
  }  
}
```

找到JdbcUtils类下面的insertStatement方法，增加Upsert模式的支持
```scala
/**  
 * Returns a PreparedStatement that inserts a row into table via conn. */def insertStatement(conn: Connection, table: String, rddSchema: StructType): PreparedStatement = {  
  var sql   
  if(saveMode == SaveMode.Upsert)  
    {  
      sql = new StringBuilder(s"UPSERT INTO $table VALUES (")  
    }else{  
      sql = new StringBuilder(s"INSERT INTO $table VALUES (")  
  }  
  var fieldsLeft = rddSchema.fields.length  
  while (fieldsLeft > 0) {  
    sql.append("?")  
    if (fieldsLeft > 1) sql.append(", ") else sql.append(")")  
    fieldsLeft = fieldsLeft - 1  
  }  
  conn.prepareStatement(sql.toString())  
}
```

#### 2.3  DataFrameWrite增加Upsert支持

找到DataFrameWrite的mode

```scala
def mode(saveMode: String): DataFrameWriter = {  
  this.mode = saveMode.toLowerCase match {  
    case "overwrite" => SaveMode.Overwrite  
    case "append" => SaveMode.Append  
    case "ignore" => SaveMode.Ignore  
    case "upsert" => SaveMode.Upsert  
    case "error" | "default" => SaveMode.ErrorIfExists  
    case _ => throw new IllegalArgumentException(s"Unknown save mode: $saveMode. " +  
      "Accepted modes are 'overwrite', 'append', 'ignore','upsert', 'error'.")  
  }  
  this  
}
```



找到DataFrameWrite的jdbc(url: String, table: String, connectionProperties: Properties)

JdbcUtils.saveTable(df, url, table, props) 改为 JdbcUtils.saveTable(df, url, table, mode, props)

```scala
def jdbc(url: String, table: String, connectionProperties: Properties): Unit = {  
  val props = new Properties()  
  extraOptions.foreach { case (key, value) =>  
    props.put(key, value)  
  }  
  // connectionProperties should override settings in extraOptions  
  props.putAll(connectionProperties)  
  val conn = JdbcUtils.createConnection(url, props)  
  
  try {  
    var tableExists = JdbcUtils.tableExists(conn, table)  
  
    if (mode == SaveMode.Ignore && tableExists) {  
      return  
    }  
  
    if (mode == SaveMode.ErrorIfExists && tableExists) {  
      sys.error(s"Table $table already exists.")  
    }  
  
    if (mode == SaveMode.Overwrite && tableExists) {  
      JdbcUtils.dropTable(conn, table)  
      tableExists = false  
    }  
  
    // Create the table if the table didn't exist.  
    if (!tableExists) {  
      val schema = JdbcUtils.schemaString(df, url)  
      val sql = s"CREATE TABLE $table ($schema)"  
      conn.prepareStatement(sql).executeUpdate()  
    }  
  } finally {  
    conn.close()  
  }  
  
  JdbcUtils.saveTable(df, url, table, mode, props)  
}

```


## 4. 以Upsert模式写入Phoenix

```scala
//以覆盖的模式写入到Apache Phoenix
    dataFrame.write.format("jdbc")
      .mode("overwrite")
      .option("dbtable",phoenix_dbtable)
      .option("url",phoenix_jdbc_url)
      .option("user",phoenix_user)
      .option("password",phoex_password)
      .save()
```