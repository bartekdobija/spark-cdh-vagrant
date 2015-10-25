# Spark 1.5 & CDH 5 Vagrantfile

Vagrantfile with Cloudera CDH 5 & Spark 1.5 configuration.
The VM is provisioned with the basic configuration of HDFS, Yarn, Hive, Oozie, Spark & MySQL (Metastore). 

After the VM has been provision you can run Spark with the below command:

```sh
sudo -u hdfs /opt/spark/bin/spark-shell
```

#### Dependencies

- Vagrant 1.7+
- VirtualBox 5+
- Internet connection
- (optionally) Spark without-hadoop compiled with Hive support, see below.

#### Spark with Hive

Official Spark 1.5 (without-hadoop) is not compiled with the Hive support. You have to compile your own version by downloading the Spark source code and calling the below command in its home directory:

```sh
./make-distribution.sh --name without-hadoop --tgz -Phadoop-2.6 -Psparkr -Phadoop-provided -Phive -Phive-thriftserver -Pyarn -DskipTests -Dmaven.javadoc.skip=true 
```

Once this is done copy over the produced Spark TGZ file into the "spark" directory in this project. Your Spark will be detected and installed instead of the default, officially available version.
