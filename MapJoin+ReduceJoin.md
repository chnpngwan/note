### reduceJoin的应用

![1654518182714](https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/220606-1654518182714.png)

#### 代码实现

##### orderbean

```java
public class OrderBean implements Writable {
    private String id;
    private String pid;
    private String pname;
    private int amount;

    @Override
    public String toString() {
        //由于最终输出只需要3列
        return id + "\t" + pname + "\t" + amount;
    }
// get与set方法

    @Override
    public void write(DataOutput out) throws IOException {
        out.writeUTF(id);
        out.writeUTF(pid);
        out.writeUTF(pname);
        out.writeInt(amount);
    }

    @Override
    public void readFields(DataInput in) throws IOException {
        this.id = in.readUTF();
        this.pid = in.readUTF();
        this.pname = in.readUTF();
        this.amount = in.readInt();
    }
```

##### mapper

```java
public class OrderMapper extends Mapper<LongWritable, Text, Text, OrderBean> {

    private Text pid = new Text();
    private OrderBean tableLine = new OrderBean();

    private String filename = "";

    /**
     * 在数据处理之前，先获取文件名
     *
     * @param context
     * @throws IOException
     * @throws InterruptedException
     */
    @Override
    protected void setup(Mapper<LongWritable, Text, Text, OrderBean>.Context context) throws IOException, InterruptedException {
        //获取文件名
        //首先获取切片
        InputSplit inputSplit = context.getInputSplit();

        //将切片转化为FileSplit，获取文件名
        if (inputSplit instanceof FileSplit) {
            FileSplit fs = (FileSplit) inputSplit;
            filename = fs.getPath().getName();
        }
    }

    /**
     * 将两个文件的数据包装为orderbean, 输出key是pid，value是orderbean
     * @param key
     * @param value
     * @param context
     * @throws IOException
     * @throws InterruptedException
     */
    @Override
    protected void map(LongWritable key, Text value, Mapper<LongWritable, Text, Text, OrderBean>.Context context) throws IOException, InterruptedException {
        //这一行数据可能来源于不同的文件，处理逻辑不同，需要区分数据来源
        String[] fields = value.toString().split("\t");

        //区分数据来源的方式，最稳妥的方式是通过输入文件的文件名判断
        //假设我们已知文件名

        if ("order.txt".equals(filename)) {
            //说明数据来源与order，三列分别是id, pid, amount
            pid.set(fields[1]);

            tableLine.setId(fields[0]);
            tableLine.setPid(fields[1]);
            tableLine.setAmount(Integer.parseInt(fields[2]));
            tableLine.setPname("");
        } else {
            //说明数据来源于pd，两列分别是pid, pname
            pid.set(fields[0]);

            tableLine.setPid(fields[0]);
            tableLine.setPname(fields[1]);
            tableLine.setId("");
            tableLine.setAmount(0);

        }

        context.write(pid, tableLine);
    }
}
```

##### reducer

```java
public class OrderReducer extends Reducer<Text, OrderBean, OrderBean, NullWritable> {

    private List<OrderBean> orders = new ArrayList<>();

    /**
     * 将一组的数据完成join
     *
     * @param key
     * @param values
     * @param context
     * @throws IOException
     * @throws InterruptedException
     */
    @Override
    protected void reduce(Text key, Iterable<OrderBean> values, Reducer<Text, OrderBean, OrderBean, NullWritable>.Context context) throws IOException, InterruptedException {
        //先找到pname
        String pname = null;
        orders.clear();

        Iterator<OrderBean> iterator = values.iterator();

        while (iterator.hasNext()) {

            OrderBean value = iterator.next();
            //判断这一行数据包不包含pname(如果包含pname，说明数据来源于pd.txt)
            if (!"".equals(value.getPname())) {
                pname = value.getPname();
            } else {
                //value只有一个对象，我们要新建对象保存信息
                OrderBean temp = new OrderBean();

                //将value的数据拷贝进temp
                temp.setId(value.getId());
                temp.setPid(value.getPid());
                temp.setPname(value.getPname());
                temp.setAmount(value.getAmount());

                orders.add(temp);
            }
        }
        //将所有来源于order的数据，设置pname并输出
        for (OrderBean order : orders) {
            order.setPname(pname);
            context.write(order, NullWritable.get());
        }

    }
}
```

##### driver

```java
public class OrderDriver {
    public static void main(String[] args) throws IOException, InterruptedException, ClassNotFoundException {
        Job job = Job.getInstance();
        job.setMapperClass(OrderMapper.class);
//        job.setReducerClass(OrderReducer.class);

        job.setReducerClass(OrderReducerOptimized.class);
        job.setMapOutputKeyClass(Text.class);
        job.setMapOutputValueClass(OrderBean.class);

        job.setOutputKeyClass(OrderBean.class);
        job.setOutputValueClass(NullWritable.class);

        FileInputFormat.setInputPaths(job, new Path("d:/BigData/input"));
        FileOutputFormat.setOutputPath(job, new Path("d:/BigData/output"));

        boolean b = job.waitForCompletion(true);

        System.exit(b ? 0 : 1);
    }
}
```

##### orderreducer提高

```java
public class OrderReducerOptimized extends Reducer<Text, OrderBean, OrderBean, NullWritable> {

    private List<OrderBean> orders = new ArrayList<>();

    /**
     * 将一组的数据完成join
     *
     * @param key
     * @param values
     * @param context
     * @throws IOException
     * @throws InterruptedException
     */
    @Override
    protected void reduce(Text key, Iterable<OrderBean> values, Reducer<Text, OrderBean, OrderBean, NullWritable>.Context context) throws IOException, InterruptedException {
        //先找到pname
        String pname = null;
//        orders.clear();
        //声名一个计数器用来计数这一组order.txt总共有几行
        int count = 0;

        for (OrderBean value : values) {

            //判断这一行数据包不包含pname(如果包含pname，说明数据来源于pd.txt)
            if (!"".equals(value.getPname())) {
                pname = value.getPname();
            } else {
                //不要一直new对象，会充满堆内存导致full gc，尽量复用上一组已经申请的对象
                OrderBean temp;
                if (count < orders.size()) {
                    //说明上一组数据处理时申请的对象还能用
                    temp = orders.get(count);
                } else {
                    //说明上一组数据处理的时候申请的对象都用完了
                    temp = new OrderBean();
                    orders.add(temp);
                }

                //将value的数据拷贝进temp
                temp.setId(value.getId());
                temp.setPid(value.getPid());
                temp.setPname(value.getPname());
                temp.setAmount(value.getAmount());

                //每处理一个来自order.txt的数据，count++
                count++;

            }
        }
        //将所有来源于order的数据，设置pname并输出
//        for (OrderBean order : orders) {
//            order.setPname(pname);
//            context.write(order, NullWritable.get());
//        }
        for (int i = 0; i < count; i++) {
            OrderBean order = orders.get(i);
            order.setPname(pname);
            context.write(order, NullWritable.get());
        }

    }
}
```

#### mapjoin的应用

![1654518886491](https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/220606-1654518886491.png)



##### MapJoinMapper

```java
public class MapJoinMapper extends Mapper<LongWritable, Text, Text, NullWritable> {

    //pMap就是pd.txt缓存进内存的结果
    private Map<String, String> pMap = new HashMap<>();

    private StringBuilder sb = new StringBuilder();

    private Text result = new Text();

    /**
     * 数据处理之前，将pd.txt缓存进内存
     *
     * @param context
     * @throws IOException
     * @throws InterruptedException
     */
    @Override
    protected void setup(Mapper<LongWritable, Text, Text, NullWritable>.Context context) throws IOException, InterruptedException {
        //在开始处理之前，将分布式缓存中的数据导入pMap
        //开流
        //首先获取分布式缓存的URI, cacheFiles[0]就是我们的pd.txt
        URI[] cacheFiles = context.getCacheFiles();

        FileSystem fileSystem = FileSystem.get(context.getConfiguration());
        //用HDFS对象开流
        FSDataInputStream dataInputStream = fileSystem.open(new Path(cacheFiles[0]));

        //按行读取数据，存进pMap
        //如果想按行读数据，要将字节流转换成字符流
        InputStreamReader inputStreamReader = new InputStreamReader(dataInputStream);
        BufferedReader reader = new BufferedReader(inputStreamReader);

        //现在开始按行处理
        String line = reader.readLine();
        while (line != null && line.length() != 0) {
            //切割数据
            String[] fields = line.split("\t");
            pMap.put(fields[0], fields[1]);

            line = reader.readLine();
        }

        //处理结束，关闭缓冲流
        IOUtils.closeStream(reader);

    }

    /**
     * map处理的表格，应该只有order.txt
     *
     * @param key
     * @param value
     * @param context
     * @throws IOException
     * @throws InterruptedException
     */
    @Override
    protected void map(LongWritable key, Text value, Mapper<LongWritable, Text, Text, NullWritable>.Context context) throws IOException, InterruptedException {
        //当执行map的时候，缓存已经结束了
        String[] fields = value.toString().split("\t");

        //拼接最终输出
        sb.setLength(0);
        sb.append(fields[0]).append("\t")
                .append(pMap.get(fields[1])).append("\t")
                .append(fields[2]);

        //包装并输出
        result.set(sb.toString());
        context.write(result, NullWritable.get());
    }
}
```

##### MapJoinMapper

```java
public class MapJoinDriver {
    public static void main(String[] args) throws IOException, InterruptedException, ClassNotFoundException {
        Job job = Job.getInstance(new Configuration());

        job.setMapperClass(MapJoinMapper.class);
        job.setNumReduceTasks(0);

        job.setMapOutputKeyClass(Text.class);
        job.setMapOutputValueClass(NullWritable.class);

        //MapJoin中要缓存的表格，以分布式缓存的形式设置到job里
        job.addCacheFile(URI.create("file:///d:/BigData/input/pd.txt"));

        FileInputFormat.setInputPaths(job, new Path("d:/BigData/input/order.txt"));
        FileOutputFormat.setOutputPath(job, new Path("d:/BigData/output"));

        boolean b = job.waitForCompletion(true);
        System.exit(b ? 0 : 1);
    }
}
```

### 数据压缩

![1654519070728](https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/20220606-1654519070728.png)

#### 压缩位置选择

![1654519122446](https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/20220606-1654519122446.png)

