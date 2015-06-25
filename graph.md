title:graph编程
date:2014-11-18 17:00
category:机器学习
tags:graph, odps

# eclipse配置
把最新的odps的tar包里面的lib加到external jar。

# 上传数据
用dship上传测试数据， 测试数据为：

```shell
1\t2:2,3:1,4:4
2\t1:2,3:2,4:1
3\t1:1,2:2,5:1
4\t1:4,2:1,5:1
5\t3:1,4:1
```

这里要解决的是最短路径的问题，实际表示图结构是：

![](http://orange-tree.oss-cn-hangzhou.aliyuncs.com/blog-images/youxiang.jpg)

# 编译jar
在eclipse里面输出jar包。


# 执行
执行如下shell脚本:

```sql
    odpscmd -e  drop table t_benhua_sssp_in ; create table  t_benhua_sssp_in (v bigint, es string)
odpscmd -e  create table if not exists t_benhua_sssp_out (v bigint, l bigint)
    ./dship  upload sssp_in.txt  t_benhua_sssp_in -fd '\t'
    odpscmd -e  read t_benhua_sssp_in
    odpscmd -e  add jar  benhua_graph_test.jar -f
    odpscmd -e  jar -libjars benhua_graph_test.jar -classpath ./benhua_graph_test.jar graph_test.SSSP 1 t_benhua_sssp_in t_benhua_sssp_out
    odpscmd -e  read t_benhua_sssp_out
```


# 输出
    正确的输出结果如下， 结合图很好理解， 是所有节点距离1的最短距离。

```
    +------------+------------+
    | v          | l          |
    +------------+------------+
    | 5          | 2          |
    | 2          | 2          |
    | 1          | 0          |
    | 3          | 1          |
    | 4          | 3          |
    +------------+------------+
```

# 代码

```java
    package graph_test;

    import java.io.IOException;

    import com.aliyun.odps.io.WritableRecord;
    import com.aliyun.odps.graph.Combiner;
    import com.aliyun.odps.graph.ComputeContext;
    import com.aliyun.odps.graph.Edge;
    import com.aliyun.odps.graph.GraphJob;
    import com.aliyun.odps.graph.GraphLoader;
    import com.aliyun.odps.graph.MutationContext;
    import com.aliyun.odps.graph.Vertex;
    import com.aliyun.odps.graph.WorkerContext;
    import com.aliyun.odps.io.LongWritable;
    import com.aliyun.odps.data.TableInfo;

    public class SSSP {

        public static final String START_VERTEX = "sssp.start.vertex.id";

        public static class SSSPVertex extends
            Vertex<LongWritable, LongWritable, LongWritable, LongWritable> {

                private static long startVertexId = -1;

                public SSSPVertex() {
                    this.setValue(new LongWritable(Long.MAX_VALUE));
                }

                public boolean isStartVertex(
                        ComputeContext<LongWritable, LongWritable, LongWritable, LongWritable> context) {
                    if (startVertexId == -1) {
                        String s = context.getConfiguration().get(START_VERTEX);
                        startVertexId = Long.parseLong(s);
                    }
                    return getId().get() == startVertexId;
                }

                @Override
                    public void compute(
                            ComputeContext<LongWritable, LongWritable, LongWritable, LongWritable> context,
                            Iterable<LongWritable> messages) throws IOException {
                        long minDist = isStartVertex(context) ? 0 : Integer.MAX_VALUE;

                        for (LongWritable msg : messages) {
                            if (msg.get() < minDist) {
                                minDist = msg.get();
                            }
                        }

                        if (minDist < this.getValue().get()) {
                            this.setValue(new LongWritable(minDist));
                            if (hasEdges()) {
                                for (Edge<LongWritable, LongWritable> e : this.getEdges()) {
                                    context.sendMessage(e.getDestVertexId(), new LongWritable(minDist
                                                + e.getValue().get()));
                                }
                            }
                        } else {
                            voteToHalt();
                        }
                    }

                @Override
                    public void cleanup(
                            WorkerContext<LongWritable, LongWritable, LongWritable, LongWritable> context)
                    throws IOException {
                        context.write(getId(), getValue());
                    }
            }

        public static class MinLongCombiner extends
            Combiner<LongWritable, LongWritable> {

                @Override
                    public void combine(LongWritable vertexId, LongWritable combinedMessage,
                            LongWritable messageToCombine) throws IOException {
                        if (combinedMessage.get() > messageToCombine.get()) {
                            combinedMessage.set(messageToCombine.get());
                        }
                    }

            }

        public static class SSSPVertexReader extends
            GraphLoader<LongWritable, LongWritable, LongWritable, LongWritable> {

                @Override
                    public void load(
                            LongWritable recordNum,
                            WritableRecord record,
                            MutationContext<LongWritable, LongWritable, LongWritable, LongWritable> context)
                    throws IOException {
                        SSSPVertex vertex = new SSSPVertex();
                        vertex.setId((LongWritable) record.get(0));
                        String[] edges = record.get(1).toString().split(",");
                        for (int i = 0; i < edges.length; i++) {
                            String[] ss = edges[i].split(":");
                            vertex.addEdge(new LongWritable(Long.parseLong(ss[0])),
                                    new LongWritable(Long.parseLong(ss[1])));
                        }

                        context.addVertexRequest(vertex);
                    }

            }

        public static void main(String[] args) throws IOException {
            if (args.length < 2) {
                System.out.println("Usage: <startnode> <input> <output>");
                System.exit(-1);
            }

            GraphJob job = new GraphJob();
            job.setGraphLoaderClass(SSSPVertexReader.class);
            job.setVertexClass(SSSPVertex.class);
            job.setCombinerClass(MinLongCombiner.class);

            job.set(START_VERTEX, args[0]);
            job.addInput(TableInfo.builder().tableName(args[1]).build());
            job.addOutput(TableInfo.builder().tableName(args[2]).build());

            long startTime = System.currentTimeMillis();
            job.run();
            System.out.println("Job Finished in "
                    + (System.currentTimeMillis() - startTime) / 1000.0 + " seconds");
        }
    }
```
