Working with Key-Value Pairs
============================

Before working with key-value pairs, we need to generate an RDD containing a list of key-value 
pairs, i.e., an RDD with a list of (K, V) tuples, denoted by RDD[(K, V)]. In general, K is an 
integer or string and V could be any object which is useful for us to derive the desired results.

The Relationship Among aggregateByKey(), reduceByKey(), and combineByKey()
--------------------------------------------------------------------------

There are three main operators for us to aggregate values in an RDD: ``aggregateByKey``, 
``reduceByKey``, and ``combineByKey``. These operators are capable of deriving exactly the same 
results from an identical RDD.

``combineByKey`` is more general then ``aggregateByKey``. Actually, the implementation of 
``aggregateByKey``, ``reduceByKey`` and ``groupByKey`` is achieved by ``combineByKey``. 
``aggregateByKey`` is similar to ``reduceByKey`` but you can provide initial values when 
performing aggregation.

As the name suggests, ``aggregateByKey`` is suitable for aggregating values by keys, example 
aggregations such as **sum**, **average**, etc. The rule here is that the extra computation spent 
for map side combine can reduce the size sent out to other nodes and driver. If your function has 
satisfied this rule, you probably should use ``aggregateByKey``.

``combineByKey`` is more general and flexible for us to specify whether we would like to 
perform map side combine. However, it is more complex to use. At minimum, you need to implement 
three functions: ``createCombiner``, ``mergeValue``, ``mergeCombiners``.

Using aggregateByKey()
----------------------

``aggregateByKey()`` used to aggregate the values for each key. ::

  aggregateByKey(zeroValue, seqFunc, combFunc, numPartitions=None)

Aggregate the values for each key, using given combine functions and a neutral "zero value". 
This function can return a different result type, U, than the type of the values in this RDD, V. 
Thus, we need one operation for merging a V into a U and one operation for merging two U's, The 
former operation is used for merging values within a partition, and the latter is used for merging 
values between partitions. To avoid memory allocation, both of these functions are allowed to 
modify and return their first argument instead of creating a new U.

Example::

  # we use (0, 0) as the zeroValue, the former element indicates sum and the latter is the count. 
  aggregateByKey((0, 0), lambda x, value: (x[0] + value, x[1] + 1), lambda x, y: (x[0] + y[0], x[1] + y[1]))

Using reduceByKey()
-------------------

Example::

  dataList = [(1, "apache"), (2, "www"), (2, "104"), (3, "plus"), (1, "spark"), (3, "com"), (2, "com"), (3, "tw"), (2, "tw")]
  dataRdd = sc.parallelize(dataList)
  dataRdd.map(lambda item: (item[0], list(item[1]))).reduceByKey(lambda a, b: a + b).collect()


Using combineByKey()
--------------------

``combineByKey()`` is a generic function to combine the elements for each key using a custom set 
of aggregation functions. ::

  combineByKey(createCombiner, mergeValue, mergeCombiners, numPartitions=None)

This function turns an RDD[(K, V)] into a result of type RDD[(K, C)], for a “combined type” C.
Note that V and C can be different – for example, one might group an RDD of type (Int, Int) into 
an RDD of type (Int, List[Int]).

Three functions need to be specified for ``combineByKey()``:

* ``createCombiner``, which turns a V into a C (e.g., creates a one-element list), we may treat it 
  as the constructor of ``combineByKey()`` for constructing a combiner.
* ``mergeValue``, to merge a V into a C (e.g., adds it to the end of a list), this function is 
  called when a new value comes into a combiner.
* ``mergeCombiners``, to combine two C’s into a single one. this function is called in the final 
  stage of ``combineByKey`` for merging two combiners.

The first example is an implementation of ``reduceByKey`` by means of ``combineByKey``. That is, 
we try to use combineByKey to achieve the same results as ``reduceByKey``. ::

  dataRdd.combineByKey(lambda value: [value], lambda x, value: x.append(value), lambda x, y: x + y).collect()

The second example is aimed at calculating averages for different keys. ::

  dataList = [("apache", 1), ("spark", 2), ("104", 2), ("com", 1), ("104", 1), ("spark", 3)]
  dataRdd = sc.parallelize(dataRdd)
  tmpRdd = dataRdd.combineByKey(lambda value: (value, 1),
                                lambda x, value: (x[0] + value, x[1] + 1),
                                lambda x, y: (x[0] + y[0], x[1] + y[1]))
  tmpRdd.map(lambda (key, (sum, count)): (key, sum / (count * 1.0))).collect()

  [('com', 1.0), ('spark', 2.5), ('apache', 1.0), ('104', 1.5)]
