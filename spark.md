## Apache Spark
- データ形式：RDDとDataFrameがある
  - RDDは主にScalaで操作する
  - DataFrameは主にSQLから操作する
    - こっちのほうが高速らしい
    - https://www.slideshare.net/databricks/dataframes-and-pipelines/11?src=clipshare
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

```
spark.sql("""
select tag1 as tag, recipeId from recipe
union all
select tag2 as tag, recipeId from recipe
union all
select tag3 as tag, recipeId from recipe
union all
select tag4 as tag, recipeId from recipe
""").createOrReplaceTempView("tagUnion")

spark.sql("""
/* tagはすべて等価なので全部unionしてみる */
select tag, cnt2 from (
    select tag, count(recipeId) as cnt2 from tagUnion
        where length(tag) > 0 group by tag
) where cnt2 > 500
""").createOrReplaceTempView("tagName");

spark.sql("""
select tu.recipeId, tu.tag
    from tagName
    join tagUnion as tu 
        on tu.tag = tagName.tag
""").createOrReplaceTempView("tagFiltered");
```

```
val rdd = spark.sql("SELECT * from tagFiltered").rdd
rdd.take(1)

val indexKeysRDD = rdd.map(x=>x(1)).distinct().zipWithIndex()
val indexKeys = indexKeysRDD.collectAsMap()

val broadcast_IndexKeySize = sc.broadcast(indexKeys.size)

def encode(x: String) =
{
    var encodeArray = Array.fill(indexKeys.size)(0)
    encodeArray(indexKeys.get(x).get.toInt)=1
    encodeArray
}

val encodedRDD = rdd.map{ x => (x(0).asInstanceOf[String],encode(x(1).asInstanceOf[String]))}.reduceByKey((a, b) => (a zip b).map(e => e._1 + e._2))
encodedRDD.take(1)
```

```
def filterByID(idFileURL: String) = 
{
    val filterIDRDD= sc.textFile(idFileURL).flatMap( line => line.split("\n") )
    val filterIDPairRDD = filterIDRDD.map{id => 
            (id, Array.fill(broadcast_IndexKeySize.value)(0))
    }
    val rdd3 = filterIDPairRDD.leftOuterJoin(encodedRDD)
    val rdd4 = rdd3.map(e => e._2._2 match {
        case Some(v) => (e._1, v)
        case None => (e._1, e._2._1)
    }) 
    rdd4
}
val encoded_train = filterByID("file:///home/ansible/recipe_id_train.tsv")
val encoded_test = filterByID("file:///home/ansible/recipe_id_test.tsv")
encoded_train.count()
encoded_test.count()
```

```
def saveIntVectorRDDToFile(rdd: org.apache.spark.rdd.RDD[(String, Array[Int])], fileURL: String) = {
    val save_target_rdd = rdd
    val encode_material_lines = save_target_rdd.map(v => v._1 +: v._2.flatMap(e => e.toString)).map(list => list.mkString("\t"))
    encode_material_lines.saveAsTextFile(fileURL)
    encode_material_lines.count()
}
def saveStringVectorRDDToFile(rdd: org.apache.spark.rdd.RDD[(String, Array[String])], fileURL: String) = {
    val save_target_rdd = rdd
    val encode_material_lines = save_target_rdd.map(v => v._1 + "\t" + v._2.mkString("\t"))
    encode_material_lines.saveAsTextFile(fileURL)
    encode_material_lines.count()
    encode_material_lines.take(3)
}
```

```
saveIntVectorRDDToFile(encoded_train, "file:///home/ansible/tag_vector_train_.tsv")
saveIntVectorRDDToFile(encoded_test, "file:///home/ansible/tag_vector_test_.tsv")
```

```
val headerRDD = sc.parallelize(Array(("recipeId", indexKeysRDD.map(e => e._1.asInstanceOf[String]).collect)))
saveStringVectorRDDToFile(headerRDD, "file:///home/ansible/tag_vector_header_.tsv")
```
