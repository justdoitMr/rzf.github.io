## Flink版WordCount

### 批处理

```java
/**
 * 批处理的wordCount程序
 */
public class WordCount {

    public static void main(String[] args) throws Exception {

//      创建执行环境
        ExecutionEnvironment env = ExecutionEnvironment.getExecutionEnvironment();

//        从文件中读取数据
        String path="D:\\soft\\idea\\work\\work08\\src\\main\\resources\\words.txt";

//        读取数据,读取文本文件是按照行读取
//        DataSet主要用来做离线的数据处理
        DataSet<String> ds = env.readTextFile(path);

//        对数据及进行处理,把每一行中的单词分开，处理成(word,1)形式
//        groupby(0):按照第一个位置处的元素进行排序,sum(1)表示按照第二个位置处的元素累加
        DataSet<Tuple2<String, Integer>> result = ds.flatMap(new MyFlatMap()).groupBy(0).sum(1);

        result.print();
    }


//    定义MyFlatMap类，实现FlatMapFunction接口
    public static class MyFlatMap implements FlatMapFunction<String, Tuple2<String,Integer>> {

    //        o表示输入进来的数据，处理好的数据，从collector输出
    public void flatMap(String s, Collector<Tuple2<String, Integer>> collector) throws Exception {

//        按照空格进行分词
        String[] words = s.split(" ");
        
//        遍历所有word,循环输出
        for (String word:words) {

//            输出数据
            collector.collect(new Tuple2<String ,Integer>(word,1));
        }
    }
}
}

```

### 流式处理

```java
public class StreamWordCount {

    public static void main(String[] args) throws Exception {

//        创建执行环境
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
//       设置并行度
        env.setParallelism(8);

        String path="D:\\soft\\idea\\work\\work08\\src\\main\\resources\\words.txt";
//        读取数据
        DataStream<String> dss = env.readTextFile(path);

//        基于数据流进行转换计算,按照第一个位置的元素分组，第二个位置的元素累加
        DataStream<Tuple2<String, Integer>> result = dss.flatMap(new WordCount.MyFlatMap()).keyBy(0).sum(1);

        result.print();

//        执行任务
        env.execute();
    }
}
```

