# Spark 2.x & CDH 6 Vagrantfile

Vagrantfile with the Cloudera CDH 6 & Spark 2.x configuration.
The VM is provisioned with basic HDFS, Yarn, Hive, Spark & MySQL (Metastore DB). 

After the VM has been provisioned, run Spark with the below command:

```sh
sudo -u hdfs /opt/spark/bin/spark-shell
```

#### Dependencies

- Vagrant 1.7+
- VirtualBox 5+
- Internet connection
- (optionally) Spark 2.x without-hadoop compiled with the Hive support, see below.

#### Spark with Hive

Official Spark 1.6/2.x (without-hadoop) does not support Hive and SqlContext. You have to compile your own version by downloading the Spark source code and calling the below command in the project's home directory:

Spark 2.x
```sh
./dev/make-distribution.sh \
    --name without-hadoop \
    --tgz \
    --pip \
    --r \
    -Phadoop-2.6 \
    -Psparkr \
    -Phadoop-provided \
    -Phive \
    -Phive-thriftserver \
    -Phive-provided \
    -Pyarn \
    -Pnetlib-lgpl \
    -Dparquet.version=1.8.1 \
    -DzincPort=3038 \
    -DskipTests \
    -Dmaven.javadoc.skip=true
```

Once this is done copy over the produced Spark TGZ file into the "spark" directory of this project. Your Spark version will be detected and installed instead.
