SQLContext and DataFrame
========================

``SQLContext`` plays a key role for manipulating ``DataFrame`` in Apache Spark. It is able to read 
from different sources, including **RDD**, **list** in Python, **pandas.DataFrame** , or JSON files, 
so as to construct the corresponding **distributed** ``DataFrame`` for further data manipulation.

Construct Distributed DataFrame from SQLContext
-----------------------------------------------

Herein, we demonstrate three fashions for transforming different data sources into ``DataFrame``.

* ``jsonRDD()``: Transforming an **RDD** with one JSON object per string into distributed 
  ``DataFrame``.

  When an **RDD** stores one JSON object per string, we can construct the corresponding 
  ``DataFrame`` by means of ``SQLContext.jsonRDD()`` function.

  Example::

    from pyspark import SparkConf, SparkContext
    from pyspark.sql import SQLContext, DataFrame
    
    APPNAME = 'SQLContext_Example'
    json_data = ["{'name': 'John', 'Math': 60, 'English': 68, 'Chinese': 78}",
                 "{'name': 'Mary', 'Math': 72, 'English': 56, 'Chinese': 98}"]
    
    if __name__ == '__main__':
        conf = SparkConf().setAppName(APPNAME)
        sc = SparkContext(conf = conf)
        sqlContext = SQLContext(sc)
    
        rdd = sc.parallelize(json_data)
        df = sqlContext.jsonRDD(rdd)
    
        df.show()

* ``createDataFrame()``: Transforming an **RDD** of **tuple**/**list**, **list** in Python or 
  **pandas.DataFrame** into ``DataFrame``.

  Example::

    from pyspark import SparkConf, SparkContext
    from pyspark.sql import SQLContext, DataFrame
    
    APPNAME = 'SQLContext_Example'
    json_data = ["{'name': 'John', 'Math': 60, 'English': 68, 'Chinese': 78}",
                 "{'name': 'Mary', 'Math': 72, 'English': 56, 'Chinese': 98}"]
    
    if __name__ == '__main__':
        conf = SparkConf().setAppName(APPNAME)
        sc = SparkContext(conf = conf)
        sqlContext = SQLContext(sc)
    
        # we use map() function to get a list of dictionary objects.
        # note that we can also transform the list into an RDD, but 
        # we don't intend to do it here.
        data = map(json.loads, json_data)

        # create the corresponding DataFrame to the list of dictionary objects
        df = sqlContext.createDataFrame(data)
    
        df.show()

  Note that the ``SQLContext.createDataFrame(rdd)`` tries to infer the schema of the given rdd
  object. When passing the ``json_data`` or an **RDD** derived from it to ``createDataFrame()``, 
  we will get the following error. ::

    TypeError: Can not infer schema for type: <type 'str'>

  If that is the case, we can use the fashion introduced previously, i.e., 
  ``SQLContext.jsonRDD(rdd)``. Or we can generate an RDD of **Row**, or **namedtuple**, or 
  **dict** and pass it to the ``createDataFrame()``.

* Generating the distributed ``DataFrame`` corresponding to a given file.

  Apart from generating the distributed ``DataFrame`` from different **RDD**\ s or objects, we can 
  also generate it from files of **JSON** format. In doing so, we need to read data by means of 
  ``DataFrameReader`` which can be obtained from ``SQLContext.read``.

  Example::

    from pyspark import SparkConf, SparkContext
    from pyspark.sql import SQLContext, DataFrame
    
    APPNAME = 'SQLContext_Example'
    FILE_PATH = 'scores.json'
    json_data = ["{'name': 'John', 'Math': 60, 'English': 68, 'Chinese': 78}",
                 "{'name': 'Mary', 'Math': 72, 'English': 56, 'Chinese': 98}"]
    
    if __name__ == '__main__':
        conf = SparkConf().setAppName(APPNAME)
        sc = SparkContext(conf = conf)
        sqlContext = SQLContext(sc)
    
        # loads a JSON file, returning the result as a DataFrame.
        # when loading a parquet file, we can use the following code
        # df = sqlContext.read.parquet(FILE_PATH)
        df = sqlContext.read.json(FILE_PATH)
    
        df.show()

  Similarly, we can generate **DataFrame** from a `Parquet <https://parquet.apache.org/>`_ file by 
  means of ``DataFrameReader`` with ``parquet(FILE_PATH)``.

Save DataFrame to Files
-----------------------

Saving a DataFrame to file is very intuitive and simple. We first obtain DataFrameWriter by means 
of ``DataFrame.write`` and then call its ``save()`` function. In the following example, the
argument ``mode`` specifies the behavior of the save operation when data already exists, which 
takes the following values:

* ``append``: Append contents of this DataFrame to existing data.
* ``overwrite``: Overwrite existing data.
* ``ignore``: Silently ignore this operation if data already exists.
* ``error`` (default case): Throw an exception if data already exists.

Note that, the default file format is parquet file when the argument ``format`` is not specified.

Example::

  from pyspark import SparkConf, SparkContext
  from pyspark.sql import SQLContext
  
  APPNAME = 'GeorgeApp'
  INPUT_FILE_PATH = 'hdfs://hadoop1:9000/user/tsungchih/test.csv'
  
  if __name__ == '__main__':
      conf = SparkConf().setAppName(APPNAME)
      sc = SparkContext(conf = conf)
      sqlContext = SQLContext(sc)
  
      rdd = sc.textFile(INPUT_FILE_PATH)
      tmpRdd = rdd.map(lambda line: line.split(","))\
                  .filter(lambda tup: tup[0] == "2006")\
                  .map(lambda tup: (tup[0], tup[3]))
      df = sqlContext.createDataFrame(tmpRdd, ("year", "age"))
      df.show()
      df.write.save('result.dat', mode = 'overwrite')
      df.write.save('result.json', mode = 'overwrite', format = 'json')
  
      sc.stop()



