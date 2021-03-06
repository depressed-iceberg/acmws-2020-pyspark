# acmws-2020-pyspark
Repository for *PySpark* Lab Session at ACM Winter School - 2020

## Prerequisite

Login to the IISc cluster (**10.24.24.2**) using SSH. Windows users can use Putty. Please use the username allocated for you in the following google sheet:

https://docs.google.com/spreadsheets/d/1CMKjWcLXCFOsa-kCa8KoplGjQDjKFRhpjoWYoKJAc5I/edit?usp=sharing

The password is **acmws@iisc**

## Task 1 - Cluster Information

Visit http://10.24.24.2:8088 in your browser. Find the following information: Total memory, Total cores, Total active nodes, Memory per node, Cores per node and Scheduler type.

## Task 2 - Inspecting the dataset

Run the following *HDFS* commands one by one in your terminal to inspect the dataset and observe the output,

```
# List all the files
hdfs dfs -ls /ml/small 

# Print first few lines of the files
hdfs dfs -head /ml/small/movies.csv
hdfs dfs -head /ml/small/ratings.csv
hdfs dfs -head /ml/small/tags.csv
hdfs dfs -head /ml/small/links.csv
```

## Task 3 - Launching PySpark

Once logged into the cluster, run the following command:
```
pyspark --master yarn --deploy-mode client --conf spark.ui.port=5001 spark.yarn.archive=hdfs:///spark-libs.jar --num-executors 1 --executor-cores 2 --executor-memory 2g
```
and you should get an output similar to the following,
```
Python 2.7.5 (default, Apr 11 2018, 07:36:10)
[GCC 4.8.5 20150623 (Red Hat 4.8.5-28)] on linux2
Type "help", "copyright", "credits" or "license" for more information.
2020-01-13 23:19:13 WARN  NativeCodeLoader:60 - Unable to load native-hadoop library for your platform... using builtin-java classes where applicable
Setting default log level to "WARN".
To adjust logging level use sc.setLogLevel(newLevel). For SparkR, use setLogLevel(newLevel).
2020-01-13 23:19:15 WARN  Utils:66 - Service 'SparkUI' could not bind on port 4040. Attempting port 4041.
Welcome to
      ____              __
     / __/__  ___ _____/ /__
    _\ \/ _ \/ _ `/ __/  '_/
   /__ / .__/\_,_/_/ /_/\_\   version 2.4.0
      /_/

Using Python version 2.7.5 (default, Apr 11 2018 07:36:10)
SparkSession available as 'spark'.
>>>
```

## Task 4 - Caching dataset in Spark

Enter the following statements in *PySpark* terminal to read and cache the datset in memory,
```
m = sc.textFile("hdfs:///ml/full/movies.csv").cache()
r = sc.textFile("hdfs:///ml/full/ratings.csv").cache()
t = sc.textFile("hdfs:///ml/full/tags.csv").cache()
l = sc.textFile("hdfs:///ml/full/links.csv").cache()
```

## Task 5 - How many distinct users have tagged movies? 
```
tu = t.map(lambda l : l.split(",")[0]).distinct()
print 'Number of Users: ', tu.count()-1
```

## Task 6 - How many users have both given ratings and tagged movies?
```
ru = r.map(lambda l : l.split(",")[0]).distinct()
print 'Number of Users: ', tu.intersection(ru).count() - 1
```

## Task 7 - What are the most number of genres assigned to any movie?
```
# Unicode reader
# Reference: https://docs.python.org/2/library/csv.html#examples
import csv

def unicode_csv_reader(unicode_csv_data, dialect=csv.excel, **kwargs):
    csv_reader = csv.reader(utf_8_encoder(unicode_csv_data), dialect=dialect, **kwargs)
    for row in csv_reader:
        yield [unicode(cell, 'utf-8') for cell in row]

def utf_8_encoder(unicode_csv_data):
    for line in unicode_csv_data:
        yield line.encode('utf-8')

mp = m.map(lambda l: tuple(unicode_csv_reader([l,]))[0])
mp = mp.filter(lambda l: l[0] != u'movieId').map(lambda l: (len(l[2].split('|')), l))
mc = mp.map(lambda l: l[0]).max()
mcm = mp.lookup(mc)
print mc, mcm
```


## Task 8 - Which are the genres with the most and least number of movies?
```
mp = m.map(lambda l: tuple(unicode_csv_reader([l,]))[0])
mgs = mp.flatMap(lambda l : l[2].split("|")).map(lambda g : (g, 1)).reduceByKey(lambda a, b: a + b)
msg = mgs.map(lambda (g,s) : (s,g))
gmin = msg.min()
gmax = msg.max()
print 'Genre with min films is',gmin[1],'with count',gmin[0]
print 'Genre with max films is',gmax[1],'with count',gmax[0]
```

Exit the *PySpark* terminal by entering `quit()`. 

## References

1. https://spark.apache.org/docs/2.4.0/rdd-programming-guide.html
2. http://spark.apache.org/docs/2.4.0/api/python/pyspark.html
3. http://files.grouplens.org/datasets/movielens/ml-latest-README.html,
