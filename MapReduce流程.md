##### 笔记

- 序列化	{手机号案例MapReduce原理：}
- MapReduce框架原理
- jar包提交之前会检查提交信息{jar包， 切片信息， 配置文件}

##### 流量统计测试

##### flownBean

```java

public class FlowBean implements Writable {

    private Long upFlow;
    private Long downFlow;
    private Long sumFlow;
    
// get与set 方法
    public void set(Long upFlow, Long downFlow) {
        this.upFlow = upFlow;
        this.downFlow = downFlow;
        this.sumFlow = upFlow + downFlow;
    }

    public void write(DataOutput dataOutput) throws IOException {
        dataOutput.writeLong(upFlow);
        dataOutput.writeLong(downFlow);
        dataOutput.writeLong(sumFlow);
    }

    public void readFields(DataInput dataInput) throws IOException {

        upFlow = dataInput.readLong();
        downFlow = dataInput.readLong();
        sumFlow = dataInput.readLong();
    }

    @Override
    public String toString() {
        return upFlow + "\t" + downFlow + "\t" + sumFlow;
    }
}
```

##### mapper

```java
public class FlowMapper extends Mapper<LongWritable, Text, Text, FlowBean> {

    private Text phone = new Text();
    private FlowBean flowBean = new FlowBean();

    /**
     * 将一行数据封装成（手机号，流量）的kv对
     * @param key
     * @param value
     * @param context
     * @throws IOException
     * @throws InterruptedException
     */
    @Override
    protected void map(LongWritable key, Text value, Context context) throws IOException, InterruptedException {
        //拿到一行数据，按照\t切分
        String[] fields = value.toString().split("\t");

        //封装手机号
        phone.set(fields[1]);

        //封装流量
        flowBean.set(
               Long.parseLong(fields[fields.length - 3]),
               Long.parseLong(fields[fields.length - 2])
        );

        //将phone和手机号输出
        context.write(phone, flowBean);
    }
}
```

##### reducer

```java
public class FlowReducer extends Reducer<Text, FlowBean, Text, FlowBean> {

    private FlowBean result = new FlowBean();
    /**
     * 按照手机号进行分组后的结果，在这里累加
     * @param key 手机号
     * @param values 手机号所有的流量
     * @param context
     * @throws IOException
     * @throws InterruptedException
     */
    @Override
    protected void reduce(Text key, Iterable<FlowBean> values, Reducer<Text, FlowBean, Text, FlowBean>.Context context) throws IOException, InterruptedException {
        //将一个手机号的所有流量进行累加
        long sumUpFlow = 0;
        long sumDownFlow = 0;

        for (FlowBean value : values) {
            sumDownFlow += value.getDownFlow();
            sumUpFlow += value.getUpFlow();
        }
        result.set(sumUpFlow, sumDownFlow);

        //将累加后的流量输出
        context.write(key, result);
    }
}
```

##### driver

```java
public class FlowDriver {
    public static void main(String[] args) throws IOException, InterruptedException, ClassNotFoundException {
        Configuration configuration = new Configuration();
//        configuration.set("mapreduce.framework.name","yarn");

        Job job = Job.getInstance(configuration);

        job.setMapperClass(FlowMapper.class);
        job.setReducerClass(FlowReducer.class);

        job.setMapOutputKeyClass(Text.class);
        job.setMapOutputValueClass(FlowBean.class);
        job.setOutputKeyClass(Text.class);
        job.setOutputValueClass(FlowBean.class);

        //将输入格式改成CombineTextInputFormat
        job.setInputFormatClass(CombineTextInputFormat.class);
        //设置多大切一片（可以跨文件）
        CombineTextInputFormat.setMaxInputSplitSize(job, 1024 * 1024);

        FileInputFormat.setInputPaths(job, new Path("d:/BigData/input"));
        FileOutputFormat.setOutputPath(job, new Path("d:/BigData/output"));

        boolean b = job.waitForCompletion(true);
        System.exit(b ? 0 : 1);
    }
}
```

# 第2**章** **Hadoop**序列化

## **2.1 序列化概述**

***\*1）什么是序列化\****

***\*序列化\****就是把内存中的对象，转换成字节序列（或其他数据传输协议）以便于存储到磁盘（持久化）和网络传输。 

***\*反序列化\****就是将收到字节序列（或其他数据传输协议）或者是磁盘的持久化数据，转换成内存中的对象。

*\*2\*\*）为什么要序列化\****

一般来说，“活的”对象只生存在内存里，关机断电就没有了。而且“活的”对象只能由本地的进程使用，不能被发送到网络上的另外一台计算机。 然而序列化可以存储“活的”对象，可以将“活的”对象发送到远程计算机。

***\*3）为什么不用\*******\*Java\*******\*的序列化\****

Java的序列化是一个重量级序列化框架（Serializable），一个对象被序列化后，会附带很多额外的信息（各种校验信息，Header，继承体系等），不便于在网络中高效传输。所以，Hadoop自己开发了一套序列化机制（Writable）。

***\*4\*******\*）\*******\*Hadoop\*******\*序列化特点\****

**Ø** ***\*紧凑 ：\****高效使用存储空间。

**Ø** ***\*快速：\****读写数据的额外开销小。

**Ø** ***\*互操作：\****`支持多语言的交互`

## 22 自定义bean对象实现序列化接口（Writable）

在企业开发中往往常用的基本序列化类型不能满足所有需求，比如在Hadoop框架内部传递一个bean对象，那么该对象就需要实现序列化接口。

具体实现bean对象序列化步骤如下7步。

（1）必须实现Writable接口

（2）反序列化时，需要反射调用空参构造函数，所以必须有空参构造

```
public FlowBean() {

​	super();

}
```

（3）重写序列化方法

```
@Override

public void write(DataOutput out) throws IOException {

​	out.writeLong(upFlow);

​	out.writeLong(downFlow);

​	out.writeLong(sumFlow);

}
```

（4）重写反序列化方法

```
@Override

public void readFields(DataInput in) throws IOException {

​	upFlow = in.readLong();

​	downFlow = in.readLong();

​	sumFlow = in.readLong();

}
```

（5）注意反序列化的顺序和序列化的顺序完全一致。

（6）要想把结果显示在文件中，需要重写toString()，可用"\t"分开，方便后续用。

（7）如果需要将自定义的bean放在key中传输，则还需要实现Comparable接口，因为MapReduce框中的Shuffle过程要求对key必须能排序。详见后面排序案例。

```JAVA
@Override

public int compareTo(FlowBean o) {

​	// 倒序排列，从大到小

​	return this.sumFlow > o.getSumFlow() ? -1 : 1;

}
```

# MapReduce框架原理

![1654096951740](https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/220601-1654096951740.png)



![1654089790487](https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/220601-1654089790487.png)

![1654089934441](https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/220601-1654089934441.png)

![1654089866947](https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/220601-1654089866947.png)

1. InputFormat

   1. 将输入数据分成几份，每份交给一个MapTask去处理（getSplit方法）

      - 这个分是一个逻辑划分

      - 每个部分的名字叫切片（InputSplit），每个切片分配一个MapTask处理

      - 对于MapRedcue，切片发生在客户端，任务提交的时候

        ```java
        boolean b = job.waitForCompletion(true);
            submit();
                //连接yarn集群
                connect();
                //通过submitter向集群内部提交任务
                submitter.submitJobInternal(Job.this, cluster);
                    //检查输出
                    checkSpecs(job);               //要求输出文件夹必须设置切不存在
        
                    //获取Job提交的临时路径
                    Path jobStagingArea = JobSubmissionFiles.getStagingDir(cluster, conf);
                    JobID jobId = submitClient.getNewJobID();
                    Path submitJobDir = new Path(jobStagingArea, jobId.toString());   //提交临时路径
        
                    //拷贝jar包到HDFS
                    copyAndConfigureFiles(job, submitJobDir);
        
                    //切片到HDFS
                    int maps = writeSplits(job, submitJobDir);
        
                    //拷贝配置文件到HDFS
                    writeConf(conf, submitJobFile);
        
                    //提交任务
                    status = submitClient.submitJob(jobId, submitJobDir.toString(), job.getCredentials());
        ```

      - 默认怎么切

        ```java
        int maps = writeSplits(job, submitJobDir);
            maps = writeNewSplits(job, jobSubmitDir);
            //找框架有没有实例化InputFormat并调用getSplit方法
                //实例化InputFormat，默认类型为TextInputFormat
                InputFormat<?, ?> input = ReflectionUtils.newInstance(job.getInputFormatClass(), conf);
                //切片
                List<InputSplit> splits = input.getSplits(job);
        ```

        ```java
        input.getSplits(job);
            //根据配置文件获取规定的最小切片大小
            long minSize = Math.max(getFormatMinSplitSize(), getMinSplitSize(job));
            //根据配置文件获取规定的最大切片大小
            long maxSize = getMaxSplitSize(job);
            //生成空数组准备保存切片信息
            List<InputSplit> splits = new ArrayList<InputSplit>();
            //查看所有的输入文件
            List<FileStatus> files = listStatus(job);
            //遍历每一个输入的文件
            for (FileStatus file: files) {
                //对每一个文件单独切片
                
                //获取文件路径
                Path path = file.getPath();
                //获取文件长度
                long length = file.getLen();
                
                //判断文件可不可以切片（根据文件是否压缩判断能否切片）
                if (isSplitable(job, path)) {
                    //获取文件块大小（默认128M）
                    long blockSize = file.getBlockSize();
                    //计算切片大小
                    long splitSize = computeSplitSize(blockSize, minSize, maxSize);
                        //默认情况下，切片大小就是HDFS的块大小
                        Math.max(minSize, Math.min(maxSize, blockSize));
                    //开始切片，剩余没切的就是文件长度
                    long bytesRemaining = length;
                    //剩余部分按照文件1.1倍判断，防止出现过小切片
                    while (((double) bytesRemaining)/splitSize > 1.1) {
                        //切一片，并将信息储存进splits
                        int blkIndex = getBlockIndex(blkLocations, length-bytesRemaining);
                        splits.add(makeSplit(path, length-bytesRemaining, splitSize,
                                    blkLocations[blkIndex].getHosts(),
                                    blkLocations[blkIndex].getCachedHosts()));
                        //迭代bytesRemaining
                        bytesRemaining -= splitSize;
                    }
                    
                    //如果剩余的部分不为0，单独成一片
                    if (bytesRemaining != 0) {
                        int blkIndex = getBlockIndex(blkLocations, length-bytesRemaining);
                        splits.add(makeSplit(path, length-bytesRemaining, bytesRemaining,
                                   blkLocations[blkIndex].getHosts(),
                                   blkLocations[blkIndex].getCachedHosts()));
                    }
                }
            }
        ```

        - 首先每个文件单独切片
        - 切片大小默认等于块大小
        - 切片时按照切片大小的**1.1**倍判断，防止出现过小的切片

   2. 对于每一个MapTask要处理的那部分数据，InputFormat会将这部分数据打碎成行，从而交给Mapper去处理

      - 每一个**MapTask**会调用**InputFormat**的**createRecordReader**方法获取一个**RecordReader**

      - **RecordReader**负责将**InputSplit**数据打碎成KV对，将KV对交给mapper

      - 默认情况下**RecordReader**会将**InputSplit**按行打碎成KV对（LongWritable，Text）

      - **RecordReader**工作方式类似于迭代器，框架通过以下代码调用**RecordReader**读取数据

        ```java
        while (context.nextKeyValue()) {
            map(context.getCurrentKey(), context.getCurrentValue(), context);
        }
        ```

   3. InputFormat多种实现类总结

      | InputFormat实现类       | 切片方式               | Key                  | Value              |
      | ----------------------- | ---------------------- | -------------------- | ------------------ |
      | TextInputFormat         | 默认切片方式           | 偏移量               | 行内容             |
      | NLineInputFormat        | N行一切片              | 偏移量               | 行内容             |
      | KeyValueTextInputFormat | 默认切片方式           | 按分隔符切分，左半部 | 按分隔符切分，右半 |
      | FixedLengthInputFormat  | 默认切片方式           | 读取数据的偏移量     | 定长字节数组内容   |
      | CombineTextInputFormat  | 跨文件按照固定大小切片 | 偏移量               | 行内容             |

2. Shuffle

   1. 排序（为了分组）
      - MapTask
        1. 数据经过Mapper处理，写到缓冲区
        2. 缓冲中的数据如果满了，快排后写到磁盘
        3. 多次写到磁盘的文件（全部内部有序），进行归并操作，得到完整的输出文件（MapTask输出）

      - ReduceTask
        1. 将所有MapTask数据全部下载到本地，再次进行归并排序，得到完整的有序文件，给Reducer输入

   2. 分区（为了将数据分给多个ReduceTask）
      - 分区的个数取决于ReduceTask个数
      - MapTask从缓冲区开始就分区，排序是在分区内部排序，归并也对同一个分区进行归并
      - MapTask最终得到的是一个分区且区内按照key有序的输出文件
      - 有几个ReduceTask，就会将数据分成几个区，ReduceTask各自处理对应分区的数据

   #### MapReduce工作流程

   ![1654090245362](https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/220601-1654090245362.png)

![1654090264305](https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/220601-1654090264305.png)

##### shufulle机制

![1654097260957](https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/220601-1654097260957.png)