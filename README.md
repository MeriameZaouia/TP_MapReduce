# TP MapReduce - Word Count

Ce projet implémente un programme MapReduce utilisant Hadoop pour effectuer un comptage de mots. Il se compose de trois classes principales :

1. **WordCountDriver** - Cette classe configure et exécute le job MapReduce.
2. **WordCountMapper** - Cette classe mappe les entrées texte en paires clé/valeur, où la clé est un mot et la valeur est le nombre d'occurrences (initialement 1).
3. **WordCountReducer** - Cette classe réduit les paires clé/valeur en additionnant les occurrences pour chaque mot.

## Structure du projet

### WordCountDriver

La classe `WordCountDriver` configure le job MapReduce et définit les entrées et sorties.

```java
package org.example.tp_Mapreduce;

import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.fs.Path;
import org.apache.hadoop.io.IntWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Job;
import org.apache.hadoop.mapreduce.lib.input.FileInputFormat;
import org.apache.hadoop.mapreduce.lib.input.TextInputFormat;
import org.apache.hadoop.mapreduce.lib.output.FileOutputFormat;

public class WordCountDriver {
    public static void main(String[] args) throws Exception {
        Configuration conf = new Configuration();
        Job job = Job.getInstance(conf);
        job.setJobName("TP WordCount");

        job.setMapperClass(WordCountMapper.class);
        job.setReducerClass(WordCountReducer.class);

        job.setInputFormatClass(TextInputFormat.class);

        job.setMapOutputKeyClass(Text.class);
        job.setMapOutputValueClass(IntWritable.class);

        job.setOutputKeyClass(Text.class);
        job.setOutputValueClass(IntWritable.class);

        FileInputFormat.addInputPath(job, new Path(args[0]));
        FileOutputFormat.setOutputPath(job, new Path(args[1]));

        System.exit(job.waitForCompletion(true) ? 0 : 1);
    }
}

```


### WordCountMapper

La classe `WordCountMapper` lit les entrées ligne par ligne, divise chaque ligne en mots, puis émet chaque mot avec une valeur de 1.

```java

package org.example.tp_Mapreduce;

import org.apache.hadoop.io.IntWritable;
import org.apache.hadoop.io.LongWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Mapper;

import java.io.IOException;

public class WordCountMapper extends Mapper<LongWritable, Text, Text, IntWritable> {
    @Override
    protected void map(LongWritable key, Text value, Mapper<LongWritable, Text, Text, IntWritable>.Context context) throws IOException, InterruptedException {
        String[] mots = value.toString().split(" ");
        for (String mot : mots) {
            context.write(new Text(mot), new IntWritable(1));
        }
    }
}
```


###  WordCountReducer

La classe `WordCountReducer` agrège les valeurs associées à chaque mot (clé) et émet le mot avec le total des occurrences.

```java

package org.example.tp_Mapreduce;

import org.apache.hadoop.io.IntWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Reducer;

import java.io.IOException;

public class WordCountReducer extends Reducer<Text, IntWritable, Text, IntWritable> {
    @Override
    protected void reduce(Text key, Iterable<IntWritable> values, Reducer<Text, IntWritable, Text, IntWritable>.Context context) throws IOException, InterruptedException {
        int sum = 0;
        for (IntWritable val : values) {
            sum += val.get();
        }
        context.write(key, new IntWritable(sum));
    }
}

```

### Exécution du programme

Pour exécuter ce programme MapReduce, vous devez spécifier le chemin d'entrée et le chemin de sortie comme arguments. Par exemple :

```bash

hadoop jar your-jar-file.jar org.example.tp_Mapreduce.WordCountDriver /path/to/input /path/to/ou

```