Basic RDD Operations
====================

In this section, we introduce some basic operations provided by RDDs. These operations could be 
categorized into *transformations* and *actions*:
 
* *transformations*: create a new RDD from an existing one, and 
* *actions*: return a value to the driver program after running a computation on the RDD. 

For example, ``map`` is a *transformation* passing each element in the given dataset to a function 
and returning a new RDD embracing the results. On the other hand, ``reduce`` is an *action* that 
aggregates all the elements of the RDD using a user-defined function and returns its output to the 
driver program (although there is also a parallel ``reduceByKey`` that returns a distributed 
dataset).

All *transformations* in Spark are **lazy**, in that they do not compute their results right 
away. Instead, they just remember the transformations applied to some base dataset (e.g. a file). 
The *transformations* are only computed when an *action* requires a result to be returned to the 
driver program. This design enables Spark to run more efficiently – for example, we can realize 
that a dataset created through ``map`` will be used in a ``reduce`` and return only the result of 
the ``reduce`` to the driver, rather than the larger mapped dataset.

By default, each transformed RDD may be recomputed each time you run an action on it. However, you 
may also persist an RDD in memory using the ``persist`` (or ``cache``) method, in which case Spark 
will keep the elements around on the cluster for much faster access the next time you query it. 
There is also support for persisting RDDs on disk, or replicated across multiple nodes.

See the `common transformations`_ and the `common actions`_ supported by Spark.

.. _common transformations: http://spark.apache.org/docs/latest/programming-guide.html#transformations
.. _common actions: http://spark.apache.org/docs/latest/programming-guide.html#actions

Using map() and reduce()
------------------------

``map`` and ``reduce`` are two of the most fundamental functions when using Spark to process 
dataset. ``reduce`` currently reduces partitions locally.

Here is a simple example we used to demonstrate the ``map`` and the ``reduce`` function. A 
teacher records scores of Math, English and Chinese for each student into a csv file. Now that the 
teacher would like to calculate the average of each subject in the end of a semester. In this case, 
we can use ``map`` and ``reduce`` in Spark to achieve it. The following are two examples for 
deriving the same result. While the first example is simpler, the second one is more descriptive.

Example 1::

  # -*- coding: utf-8 -*-
  
  from pyspark import SparkConf, SparkContext
  
  APPNAME = 'GeorgeApp'
  
  # strdata contains a list of strings
  strdata = ["John, 89, 66, 88", "Mary, 75, 60, 78", "George, 82, 86, 90"]
  
  if __name__ == '__main__':
      conf = SparkConf().setAppName(APPNAME)
      sc = SparkContext(conf = conf)
  
      # generate an RDD from strdata
      rdd = sc.parallelize(strdata)
  
      # 1. the map function extracts all scores for each element.
      # 2. the reduce function sums up scores and counts by each subject
      #    and returns a tuple object as the result.
      result = rdd.map(lambda line: map(float, line.split(",")[1:] + [1])) \
                  .reduce(lambda x, y: (x[0] + y[0],
                                        x[1] + y[1],
                                        x[2] + y[2],
                                        x[3] + y[3]))
      print("Averages:")
      print("\tMath = %3.2f" % (result[0] / result[3],))
      print("\tEnglish = %3.2f" % (result[1] / result[3],))
      print("\tChinese = %3.2f" % (result[2] / result[3],))
  
      sc.stop()

Example 2::

  # -*- coding: utf-8 -*-
  
  from pyspark import SparkConf, SparkContext
  
  APPNAME = 'GeorgeApp'
  
  # strdata contains a list of strings
  strdata = ["John, 89, 66, 88", "Mary, 75, 60, 78", "George, 82, 86, 90"]
  
  if __name__ == '__main__':
      conf = SparkConf().setAppName(APPNAME)
      sc = SparkContext(conf = conf)
  
      # generate an RDD from strdata
      rdd = sc.parallelize(strdata)
  
      # 1. the first map function extracts all scores for each element.
      # 2. the second map function forms a dictionary object for each element 
      #    generated from the previous map function.
      # 3. the final reduce function sums up scores and counts by each subject
      #    and returns a dictionary object as the result.
      result = rdd.map(lambda line: map(float, line.split(",")[1:])) \
                  .map(lambda lst: {"Math": lst[0],
                                    "English": lst[1],
                                    "Chinese": lst[2],
                                    "count": 1 }) \
                  .reduce(lambda x, y: {"Math": x["Math"] + y["Math"],
                                        "English": x["English"] + y["English"],
                                        "Chinese":x["Chinese"] + y["Chinese"],
                                        "count": x["count"] + y["count"]})
      print("Averages:")
      print("\tMath = %3.2f" % (result["Math"] / result["count"],))
      print("\tEnglish = %3.2f" % (result["English"] / result["count"],))
      print("\tChinese = %3.2f" % (result["Chinese"] / result["count"],))
  
      sc.stop()


