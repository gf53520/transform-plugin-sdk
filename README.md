# transform-plugin-sdk

## 添加依赖
maven方式:

```
<dependency>
  <groupId>com.qiniu</groupId>
  <artifactId>transform-plugin-sdk</artifactId>
  <version>1.0.0</version>
</dependency>
```
或者sbt方式:

```
"com.qiniu" %% "transform-plugin-sdk" % 1.0.0
```

或者gradle方式：

```
compile 'com.qiniu:transform-plugin-sdk:1.0.0'
```

## plugin编写

### Java语言编写plugin

* 自定义 `plugin` 需要继承 `JavaParser` 抽象类,  [JavaParser example](https://github.com/qiniu/transform-plugin-sdk/blob/develop/src/main/scala/com/qiniu/pipeline/sdk/examples/AppendDateTimeJavaParser.java)。
* 重写 `JavaParser` 中 `parse` 方法， `parse` 方法支持单行到单行或者多行间的转换，函数返回值类型为 `List<Row>`。
* **注意:** parse的返回值`List<Row>`中每行必须由原始行数据以及`TransformSpec`中`plugin`的`output`所有字段对应的值共同组成，例如：

`TransformSpec`中`plugin`如下， 其中`com.qiniu.AppendTwoFieldsJavaParser`功能是为每行数据添加2个字段：

```
"plugin":   {
     "name": "com.qiniu.AppendTwoFieldsJavaParser",
     "output": [
           {
              "name":"Field1",
              "type":"string"
           },{
              "name":"Field2",
              "type":"string"
           }
   	]
  }
```
那么`parse` 返回值`parseResult`(类型为`List<Row>`)为：

``` java
// 1.初始化每行输出数据resultRow，主要由原始行数据组成
	List<Object> resultRow = new ArrayList<Object>(Arrays.asList(((GenericRow)originData).values()));
// 2.根据plugin中output的字段顺序Field1和Field2，依次在每行输出数据后追加Field1和Field2相应值Value1和Value2
	resultRow.add(Value1);
	resultRow.add(Value2);
// 3.由于本parse方法是单行到单行间的转换，最终返回的 `parseResult`的size为1，只包含resultRow
	List<Row> parseResult = new ArrayList<Row>();
	parseResult.add(new GenericRow(resultRow.toArray()));
```
，上面`parse`是单行到单行间的转换，因此输出的parseResult中仅包含一行数据，该行数据需要在输入行数据后面继续按顺序追加`output`中所有字段。如果对于是单行到N行转换的parse，则输出的parseResult中包含N行数据，其中每行数据仍需按顺序继续追加`output`中所有字段。


### Scala语言编写plugin

* 自定义 `plugin` 需要继承 `ScalaParser` 抽象类，[ScalaParser example](https://github.com/qiniu/transform-plugin-sdk/blob/develop/src/main/scala/com/qiniu/pipeline/sdk/examples/AppendDateTimeParser.scala)。
* 重写 `ScalaParser` 中 `parse` 方法， `parse` 方法支持单行到单行或者多行间的转换，函数返回值类型为 `Seq[Row]`。
* **注意:** parse的返回值中必须由原始行数据以及在`TransformSpec`中`plugin`的`output`所有字段对应的值共同组成，例如：

`TransformSpec`中`plugin`如下， 其中`com.qiniu.AppendTwoFieldsScalaParser`功能是为每行数据添加2个字段：

```
"plugin":   {
     "name": "com.qiniu.AppendTwoFieldsScalaParser",
     "output": [
           {
              "name":"Field1",
              "type":"string"
           },{
              "name":"Field2",
              "type":"string"
           }
   	]
  }
```
那么`parse` 返回值`parseResult`(类型为`Seq[Row]`)为：

``` scala
// 1.初始化每行输出数据resultRow，主要由原始行数据组成
	val resultRow = originData.toSeq
// 2.根据plugin中output的字段顺序Field1和Field2，依次在每行输出数据后追加Field1和Field2相应值Value1和Value2	
	val resultRow = orignalRow ++ Seq(Value1, Value2)
// 3.由于本parse方法是单行到单行间的转换，最终返回的 `parseResult`的size为1，只包含resultRow
	val parseResult = Seq(Row.fromSeq(resultRow))
```
，上面`parse`是单行到单行间的转换，因此输出的parseResult中仅包含一行数据，该行数据需要在输入行数据后面继续按顺序追加`output`中所有字段。如果对于是单行到N行转换的parse，则输出的parseResult中包含N行数据，其中每行数据仍需按顺序继续追加`output`中所有字段。


### Tips相关辅助函数说明
* 获取原始行数据中某个key对应的value:

**Java**

``` java
// 1.首先获取key在原始行数据中的index
	int index = getFieldIndex(key);
// 2.原始每行数据
	List<Object> newRow = new ArrayList<Object>(Arrays.asList(((GenericRow)originData).values()));
// 3.获取key对应的Value值, 其类型Type为源repo中该key的类型，注意当key为date类型时候，类型Type为Long类型。
	Type value = (Type)newRow.get(index);
```

**Scala**

``` scala
// 1.首先获取key在原始行数据中的index
	val index:Int = getFieldIndex(key)
// 2.原始每行数据
	val orignalRow: Seq[Any] = originData.toSeq
// 3.获取key对应的Value, 其类型Type为源repo中该key的类型，注意当key为date类型时候，类型Type为Long类型。
	val value: Type = newRow.get(index).asInstanceOf[Type];
```

* 获取`TransformSpec`中`plugin`的`output`所有字段：

**Java**

``` java
List<String> pluginOutputs = getPluginFields();
```

**Scala**

``` scala
// 注意JList为java中List接口，非Scala中List
	val pluginOutputs: JList[String] = getPluginFields
```