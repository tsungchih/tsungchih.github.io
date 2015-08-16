Installation of Apache Zeppelin
===============================

In this section, we introduce how to custom build Apache Zeppelin. The latest version of Apache 
Zeppelin is **0.6.0-incubating** when this article is writing.

Set Up Environment
------------------

For building Apache Zeppelin, we need to have the following installed:

  * Java 1.7
  * Apache Maven 3.2+
  * Node.js Package Manager

If the above requirements has not been ready yet, we have to install it before build.

::

  sudo apt-add-repository ppa:chris-lea/node.js
  sudo add-apt-repository ppa:webupd8team/java

  sudo apt-get update
  sudo apt-get install oracle-java7-installer
  sudo apt-get install python-software-properties
  sudo apt-get install nodejs

  wget http://mirror.catn.com/pub/apache/maven/maven-3/3.3.3/binaries/apache-maven-3.3.3-bin.tar.gz
  tar xzf apache-maven-3.3.3-bin.tar.gz
  sudo cp -R apache-maven-3.3.3 /opt
  sudo update-alternatives --install /usr/bin/mvn mvn /opt/apache-maven-3.3.3/bin/mvn
  sudo update-alternatives --install /usr/bin/mvnDebug mvnDebug /opt/apache-maven-3.3.3/bin/mvnDebug
  mvn -v

Build Apache Zeppelin
---------------------

In this subsection, we show how to custom build Apache Zeppelin as a distribution with

  * Apache Spark 1.4.1
  * Hadoop 2.6.0
  * Yarn
  * Apache Ignite 1.1.0-incubating

Download Apache Zeppelin source of the latest version from `GitHub <https://github.com/apache/incubator-zeppelin>`_. 

::

  unzip incubator-zeppelin-master.zip
  cd incubator-zeppelin-master

Start custom build::

  mvn clean package -P build-distr -Pspark-1.4 -Dhadoop.version=2.6.0 -Phadoop-2.6 -Pyarn -Dignite.version=1.1.0-incubating -DskipTests

The archive is generated under *zeppelin-distribution/target* directory.

Install and Configure Apache Zeppelin
-------------------------------------

This subsection introduce how to install custom built Apache Zeppelin and configure it.

::

  tar xzf zeppelin-0.6.0-incubating-SNAPSHOT.tar.gz --no-same-owner
  sudo mv zeppelin-0.6.0-incubating-SNAPSHOT /opt/zeppelin-0.6.0
  cd /opt/zeppelin-0.6.0

Zeppelin configuration files can be found as follows.

::

  ./conf/zeppelin-env.sh
  ./conf/zeppelin-site.xml

There are templates for the above configuration files.

::

  ./conf/zeppelin-env.sh.template
  ./conf/zeppelin-site.xml.template

We can then copy them into the correct configuration files.

::

  cp ./conf/zeppelin-env.sh.template ./conf/zeppelin-env.sh
  cp ./conf/zeppelin-site.xml.template ./conf/zeppelin-site.xml

While ``./conf/zeppelin-env.sh`` defines environmental variables required by Zeppelin runtime, 
``./conf/zeppelin-site.xml`` includes settings for Zeppelin's daemons. Herein, we configure 
Zeppelin with standalone Spark 1.4.1 instead of Mesos or Yarn cluster. In this case, the following 
information should be embraced in ``./conf/zeppelin-env.sh``.

::

  # ./conf/zeppelin-env.sh
  export JAVA_HOME=/usr/lib/jvm/java-7-oracle
  export MASTER=spark://hadoop1:7077

Start Zeppelin
--------------

After having configured Zeppelin, we can start the Zeppelin service as follows.

::

  ./bin/zeppelin-daemon.sh start

We can then brows localhost:8080 in the browser.
