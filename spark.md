## Apache Spark
- データ形式：RDDとDataFrameがある
  - RDDは主にScalaで操作する
  - DataFrameは主にSQLから操作する
- 両者は相互に変換できる

```
case class Tukurepo(
    recipeId: String, userId: String, 
    category0: String, category1: String, category2: String, 
    title: String, motivation: String, explain: String, name: String,
    tag1: String, tag2: String, tag3: String, tag4: String,
    time: String, purpose: String, cost: String,
    data: String)

val tukurepoDF = sc.textFile("swebhdfs://somewhere/to/your/hdfs/data")
    .flatMap( line => line.split("\n") )
    .map(_.split("\t")).filter(_.length > 0)
    .map(e => new Tukurepo(
        e(0), e(1), 
        e(2), e(3), e(4), 
        e(5), e(6), e(7), e(9),
        e(10), e(11), e(12), e(13),
        e(15), e(16), e(17),
        e(19))).toDF
tukurepoDF.createOrReplaceTempView("recipe");
```
