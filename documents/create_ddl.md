# create impala table from json

1. spark shell 실행
```bash
spark2-shell
```
2. json import
```scala
import org.apache.spark.sql.types._
import org.apache.spark.sql.functions._
import spark.implicits._

val path = "/user/training/yelp2/business/business.json"
val df1 = spark.read.json(path)
df1.show()
```
3. 스키마에 포함된 '-'를 '_'로 변경하고 parquet 포맷으로 저장
```scala
def renameAllCols(schema: StructType, rename: String => String): StructType = {
  def recurRename(schema: StructType): Seq[StructField] = schema.fields.map{
      case StructField(name, dtype: StructType, nullable, meta) =>
        StructField(rename(name), StructType(recurRename(dtype)), nullable, meta)
      case StructField(name, dtype, nullable, meta) =>
        StructField(rename(name), dtype, nullable, meta)
    }
  StructType(recurRename(schema))
}

val renameFcn = (s: String) => s.replace("-", "_")

val df2 = spark.createDataFrame(df1.rdd, renameAllCols(df1.schema, renameFcn))
df2.printSchema
df2.coalesce(1).write.format("parquet").save("tmpbusiness")
```
4. Hue에서 Impala 테이블 생성
```sql
CREATE EXTERNAL TABLE tmpbusiness4 LIKE PARQUET '/user/training/tmpbusiness/part-00000-f3ef6f15-f6f6-4d57-96cd-426569c732af-c000.snappy.parquet'
STORED AS PARQUET
LOCATION '/user/training/tmpbusiness';
```
