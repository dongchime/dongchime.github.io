---
layout: post
title:  "IntelliJ와 maven을 이용한 hadoop 테스트 환경 설정"
date:   2019-12-20 16:21:22 +0900
categories: jekyll update
---

### 사전 준비사항
- java 8 이후의 버전 설치
- IntelliJ (maven plugin 포함) 설치

###  새 프로젝트 생성
- 새 프로젝트 생성할 때 Maven 선택

### pom.xml에 mapreduce dependency 추가하기
```xml
<dependencies>
    <dependency>
        <groupId>org.apache.hadoop</groupId>
        <artifactId>hadoop-mapreduce-client-core</artifactId>
        <version>3.0.0</version>
    </dependency>
    <dependency>
        <groupId>org.apache.hadoop</groupId>
        <artifactId>hadoop-common</artifactId>
        <version>3.0.0</version>
    </dependency>
    <dependency>
        <groupId>org.apache.hadoop</groupId>
        <artifactId>hadoop-mapreduce-client-jobclient</artifactId>
        <version>3.0.0</version>
    </dependency>
</dependencies>
```

- [ERROR] Source option 5 is no longer supported. Use 6 or later. 에러가 뜰 때 아래 내용 추가.

```xml
<build>
    <plugins>
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-compiler-plugin</artifactId>
            <version>3.7.0</version>
            <configuration>
                <source>11</source>
                <target>11</target>
            </configuration>
        </plugin>
    </plugins>
</build>
```

### MapReduce 예제코드 
- MapReduce의 Hello World 급인 WordCount 예제를 테스트 해봅시다.
- src/main/java 디렉토리에서 패키지를 하나 생성합니다.
- 생성한 패키지 내에 WordCount class를 생성합니다.
- 아래 코드를 복붙합니다. 
- 'WordCount.java'

```java
package packagename;

import java.io.IOException;
import java.util.StringTokenizer;

import org.apache.hadoop.conf.Configured;
import org.apache.hadoop.fs.Path;
import org.apache.hadoop.io.IntWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Job;
import org.apache.hadoop.mapreduce.Mapper;
import org.apache.hadoop.mapreduce.Reducer;
import org.apache.hadoop.mapreduce.lib.input.FileInputFormat;
import org.apache.hadoop.mapreduce.lib.input.TextInputFormat;
import org.apache.hadoop.mapreduce.lib.output.FileOutputFormat;
import org.apache.hadoop.mapreduce.lib.output.TextOutputFormat;
import org.apache.hadoop.util.Tool;
import org.apache.hadoop.util.ToolRunner;

public class WordCount extends Configured implements Tool{
    
    public static void main(String[] args) throws Exception {
        ToolRunner.run(new WordCount(), args);
    }
    
    public static class WCMapper extends Mapper<Object, Text, Text, IntWritable>{

        Text word = new Text();
        IntWritable one = new IntWritable(1);
        
        @Override
        protected void map(Object key, Text value, Mapper<Object, Text, Text, IntWritable>.Context context)
                throws IOException, InterruptedException {
            
            StringTokenizer st = new StringTokenizer(value.toString());
            
            while(st.hasMoreTokens()) {
                word.set(st.nextToken());
                context.write(word, one);
            }
            
        }
        
    }
    
    public static class WCReducer extends Reducer<Text, IntWritable, Text, IntWritable>{

        IntWritable oval = new IntWritable();
        
        @Override
        protected void reduce(Text key, Iterable<IntWritable> values,
                Reducer<Text, IntWritable, Text, IntWritable>.Context context) throws IOException, InterruptedException {
            
            int sum = 0;
            for(IntWritable value : values) {
                sum += value.get();
            }
            oval.set(sum);
            
            context.write(key, oval);
        }
        
    }

    public int run(String[] args) throws Exception {
        
        Job myjob = Job.getInstance(getConf());
        myjob.setJarByClass(WordCount.class);
        myjob.setMapperClass(WCMapper.class);
        myjob.setReducerClass(WCReducer.class);
        myjob.setMapOutputKeyClass(Text.class);
        myjob.setMapOutputValueClass(IntWritable.class);
        myjob.setOutputFormatClass(TextOutputFormat.class);
        myjob.setInputFormatClass(TextInputFormat.class);
        FileInputFormat.addInputPath(myjob, new Path(args[0]));
        FileOutputFormat.setOutputPath(myjob, new Path(args[0]).suffix(".out"));
        
        myjob.waitForCompletion(true);
        
        return 0;
    }
    
}
```

### WordCount 테스트 코드
- src/test/java 내에 위와 같은 패키지 이름을 가진 패키지를 하나 생성합니다.
- 패키지 내에 WordCountTest class를 생성합니다. 
- WordCountTest.java

```java
package packagename;

import org.apache.hadoop.util.ToolRunner;

public class WordCountTest {
    public static void main(String[] args) throws Exception {
        ToolRunner.run(new WordCount(), new String[] {"src/test/resources/testfile.txt"});
    }
}
```

- src/test 에 resources 디렉토리를 생성한 후 그 안에 word count 하고 싶은 텍스트가 있는 testfile.txt를 추가합니다.
- 테스트 코드를 빌드 및 실행합니다.
- src/test/resources/testfile.txt.out 디렉토리에서 output을 확인해 볼 수 있습니다.




