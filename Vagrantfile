# Anaconda dependencies
$anaconda_deps = <<SCRIPT

  ANACONDA_INSTALLER=https://3230d63b5fc54e62148e-c95ac804525aac4b6dba79b00b39d1d3.ssl.cf1.rackcdn.com/Anaconda-2.3.0-Linux-x86_64.sh

  if [ ! -d "/usr/local/anaconda" ]; then
    echo "Anaconda installation..." \
      && echo "downloading binaries" \
      && wget ${ANACONDA_INSTALLER} -q -P /tmp/ \
      && echo "running installer" \
      && bash /tmp/Anaconda-2.3.0-Linux-x86_64.sh -b -f -p /usr/local/anaconda
  fi

SCRIPT

# Spark dependencies
$spark_deps = <<SCRIPT

  SPARK_VER=spark-1.6.2-bin-without-hadoop
  SPARK_LINK=/opt/spark

  if [ ! -e ${SPARK_LINK} ]; then
    adduser spark
    echo "Spark installation..."
    if [ "$(ls -la /vagrant/spark/ | grep ${SPARK_VER}.tgz | wc -l)" == "1" ]; then
      cp -f /vagrant/spark/${SPARK_VER}.tgz /tmp/
    else
      wget http://d3kbcqa49mib13.cloudfront.net/${SPARK_VER}.tgz -q -P /tmp/
    fi
    tar zxf /tmp/${SPARK_VER}.tgz -C /opt/ \
      && ln -s /opt/${SPARK_VER} ${SPARK_LINK} \
      && chown -R spark:spark /opt/${SPARK_VER}
  fi

  [ ! -e ${SPARK_LINK} ] && echo "Spark installation has failed!" && exit 1

  echo "Spark configuration..."
  echo "configuring /etc/profile.d/spark.sh"
  echo 'export PATH=$PATH'":${SPARK_LINK}/bin" > /etc/profile.d/spark.sh

  echo "configuring /opt/spark/conf/spark-env.sh"
  cat << SPCNF > /opt/spark/conf/spark-env.sh
HADOOP_CONF_DIR=/etc/hadoop/conf/
SPARK_DIST_CLASSPATH=\\$(hadoop classpath):${SPARK_LINK}/hive/lib/*
LD_LIBRARY_PATH=\\${LD_LIBRARY_PATH}:/usr/lib/hadoop/lib/native
SPCNF

  echo "configuring ${SPARK_LINK}/conf/spark-defaults.conf"
  cat << SPCNF > ${SPARK_LINK}/conf/spark-defaults.conf
# enable logging of spark application events
spark.eventLog.enabled true
spark.eventLog.dir hdfs:///tmp/spark-logs
spark.history.fs.logDirectory hdfs:///tmp/spark-logs
spark.yarn.historyServer.address \\${hadoopconf-yarn.resourcemanager.hostname}:18080
spark.history.fs.cleaner.enabled true
spark.shuffle.service.enabled true
# Execution Behavior
spark.broadcast.blockSize 4096
# Dynamic Resource Allocation (YARN)
spark.dynamicAllocation.enabled true
spark.speculation true
spark.scheduler.mode FAIR
spark.kryoserializer.buffer.max 1000m
spark.driver.maxResultSize 0
spark.serializer org.apache.spark.serializer.KryoSerializer
spark.yarn.preserve.staging.files false
spark.master yarn
spark.rdd.compress true
# Local execution of selected Spark functions
spark.localExecution.enabled true
spark.sql.parquet.binaryAsString true
spark.sql.parquet.compression.codec snappy
# use lz4 compression for broadcast variables as Snappy is not supported on MacOSX
spark.broadcast.compress true
spark.io.compression.codec lz4
spark.driver.extraLibraryPath /usr/lib/hadoop/lib/native
spark.executor.extraLibraryPath /usr/lib/hadoop/lib/native
spark.executor.extraJavaOptions -verbose:gc -XX:+PrintGCDetails -XX:+PrintGCTimeStamps -XX:+UseCompressedOops
spark.driver.extraJavaOptions -XX:+UseCompressedOops -XX:MaxPermSize=1g
spark.executor.extraClassPath /usr/local/lib/jdbc/mysql/*.jar
spark.driver.extraClassPath /usr/local/lib/jdbc/mysql/*.jar
SPCNF

  echo "configuring ${SPARK_LINK}/conf/hive-site.xml"
  cat << HIVECNF > ${SPARK_LINK}/conf/hive-site.xml

<configuration>
<property>
  <name>javax.jdo.option.ConnectionURL</name>
  <value>jdbc:mysql://localhost:3306/hive?createDatabaseIfNotExist=true</value>
</property>
<property>
  <name>javax.jdo.option.ConnectionUserName</name>
  <value>hive</value>
</property>
<property>
  <name>javax.jdo.option.ConnectionPassword</name>
  <value>hive</value>
</property>
<property>
  <name>javax.jdo.option.ConnectionDriverName</name>
  <value>com.mysql.jdbc.Driver</value>
</property>
<property>
  <name>hive.metastore.uris</name>
  <value>thrift://cdh.instance.com:9083</value>
</property>
<property>
  <name>hive.metastore.warehouse.dir</name>
  <value>hdfs:///user/hive/warehouse</value>
</property>
</configuration>

HIVECNF

  echo "installing YARN shuffle jar" \
    && mkdir -p /usr/lib/hadoop-yarn/lib/ \
    && cp -f ${SPARK_LINK}/lib/spark-*-yarn-shuffle.jar /usr/lib/hadoop-yarn/lib/


HIVE_VER=1.2.2
if [ ! -e ${SPARK_LINK}/hive ]; then
  wget http://mirrors.whoishostingthis.com/apache/hive/hive-${HIVE_VER}/apache-hive-${HIVE_VER}-bin.tar.gz -q -P ${SPARK_LINK} \
    && tar zxf ${SPARK_LINK}/apache-hive-${HIVE_VER}-bin.tar.gz -C ${SPARK_LINK} \
    && ln -s ${SPARK_LINK}/apache-hive-${HIVE_VER}-bin ${SPARK_LINK}/hive
fi



SCRIPT

# MySQL dependencies
$mysql_deps = <<SCRIPT

  MYSQL_REPO=https://dev.mysql.com/get/mysql-community-release-el6-5.noarch.rpm
  MY_CNF=/etc/my.cnf
  DEV_PASSWORD=hadoop

  [ ! -e /etc/yum.repos.d/mysql-community.repo ] && rpm -ivh ${MYSQL_REPO}

  yum install -y mysql-community-server

  if [ -e /etc/init.d/mysqld ] && [ -z "$(grep -R vagrant ${MY_CNF})" ]; then
    echo "# InnoDB settings" >> ${MY_CNF}
    echo "default_storage_engine = innodb" >> ${MY_CNF}
    echo "innodb_file_per_table = 1" >> ${MY_CNF}
    echo "innodb_flush_log_at_trx_commit = 2" >> ${MY_CNF}
    echo "innodb_log_buffer_size = 64M" >> ${MY_CNF}
    echo "innodb_buffer_pool_size = 1G" >> ${MY_CNF}
    echo "innodb_thread_concurrency = 8" >> ${MY_CNF}
    echo "innodb_flush_method = O_DIRECT" >> ${MY_CNF}
    echo "innodb_log_file_size = 512M" >> ${MY_CNF}
    echo "explicit_defaults_for_timestamp = 1" >> ${MY_CNF}
    chkconfig mysqld on \
      && service mysqld start \
      && /usr/bin/mysqladmin -u root password "${DEV_PASSWORD}" &> /dev/null \
      && echo "# vagrant provisioned" >> ${MY_CNF}

    mysql -u root -p${DEV_PASSWORD} \
      -e "create schema if not exists hive; grant all on hive.* to 'hive'@'localhost' identified by 'hive'" \
    && mysql -u root -p${DEV_PASSWORD} \
      -e "create schema if not exists oozie; grant all on oozie.* to 'oozie'@'localhost' identified by 'oozie'"
  fi

SCRIPT

# Cloudera CDH dependencies
$cloudera_deps = <<SCRIPT

  CLOUDERA_REPO=http://archive.cloudera.com/cdh5/redhat/6/x86_64/cdh/cloudera-cdh5.repo

  # Add Cloudera repository
  if [[ ! -e /etc/yum.repos.d/cloudera-cdh5.repo ]]; then
    wget ${CLOUDERA_REPO} -q -P /tmp/ \
      && awk '{ gsub("cdh/5/","cdh/5.7.1/"); print }' /tmp/cloudera-cdh5.repo > /etc/yum.repos.d/cloudera-cdh5.repo
  fi

  # Cloudera seems to kill fast download ops so we throttle Yum
  if [ "$(grep throttle /etc/yum.conf | wc -l)" == "0" ]; then
    echo "throttle=3M" >> /etc/yum.conf
  fi

  # Cloudera Hadoop installation
  yum install -y hadoop hadoop-conf-pseudo hadoop-hdfs-datanode hadoop-hdfs-journalnode  \
  hadoop-hdfs-namenode hadoop-hdfs-secondarynamenode hadoop-hdfs-zkfc \
  hadoop-libhdfs-devel hadoop-mapreduce-historyserver hadoop-yarn-nodemanager \
  hadoop-yarn-resourcemanager zookeeper zookeeper-native zookeeper-server \
  oozie oozie-client kite sqoop hive hive-metastore hive-server2 hive-hcatalog \
  hive-jdbc avro-libs pig kite impala* openssl-devel openssl

  cat << HDPCNF > /etc/hadoop/conf/mapred-site.xml

<configuration>
<property>
  <name>mapred.job.tracker</name>
  <value>cdh.instance.com:8021</value>
</property>
<property>
  <name>mapreduce.framework.name</name>
  <value>yarn</value>
</property>
<property>
  <name>mapreduce.jobhistory.address</name>
  <value>cdh.instance.com:10020</value>
</property>
<property>
  <name>mapreduce.jobhistory.webapp.address</name>
  <value>cdh.instance.com:19888</value>
</property>
<property>
  <name>mapreduce.task.tmp.dir</name>
  <value>/var/lib/hadoop-mapreduce/cache/\\${user.name}/tasks</value>
</property>
<property>
  <name>mapreduce.map.memory.mb</name>
  <value>512</value>
</property>
<property>
  <name>mapreduce.reduce.memory.mb</name>
  <value>512</value>
</property>
<property>
  <name>yarn.app.mapreduce.am.staging-dir</name>
  <value>/user</value>
</property>
<property>
  <name>mapreduce.job.user.classpath.first</name>
  <value>true</value>
</property>
</configuration>

HDPCNF

  cat << YRNCNF > /etc/hadoop/conf/yarn-site.xml

<configuration>
<property>
  <name>yarn.resourcemanager.hostname</name>
  <value>cdh.instance.com</value>
</property>
<property>
  <name>yarn.nodemanager.resource.cpu-vcores</name>
  <value>16</value>
</property>
<property>
  <name>yarn.nodemanager.aux-services</name>
  <value>mapreduce_shuffle,spark_shuffle</value>
</property>
<property>
  <name>yarn.nodemanager.aux-services.mapreduce_shuffle.class</name>
  <value>org.apache.hadoop.mapred.ShuffleHandler</value>
</property>
<property>
  <name>yarn.nodemanager.aux-services.spark_shuffle.class</name>
  <value>org.apache.spark.network.yarn.YarnShuffleService</value>
</property>
<property>
  <name>yarn.log-aggregation-enable</name>
  <value>true</value>
</property>
<property>
  <name>yarn.dispatcher.exit-on-error</name>
  <value>true</value>
</property>
<property>
  <name>yarn.nodemanager.local-dirs</name>
  <value>/var/lib/hadoop-yarn/cache/\\${user.name}/nm-local-dir</value>
</property>
<property>
  <name>yarn.nodemanager.log-dirs</name>
  <value>/var/log/hadoop-yarn/containers</value>
</property>
<property>
  <name>yarn.nodemanager.remote-app-log-dir</name>
  <value>/var/log/hadoop-yarn/apps</value>
</property>
<property>
  <name>yarn.application.classpath</name>
  <value>\\$HADOOP_CONF_DIR,\\$HADOOP_COMMON_HOME/*,\\$HADOOP_COMMON_HOME/lib/*,\\$HADOOP_HDFS_HOME/*,
    \\$HADOOP_HDFS_HOME/lib/*,\\$HADOOP_MAPRED_HOME/*,\\$HADOOP_MAPRED_HOME/lib/*,\\$HADOOP_YARN_HOME/*,
    \\$HADOOP_YARN_HOME/lib/*
  </value>
</property>
</configuration>

YRNCNF

  cat << HDFSCNF > /etc/hadoop/conf/hdfs-site.xml

   <configuration>
     <property>
       <name>dfs.replication</name>
       <value>1</value>
     </property>
     <property>
       <name>dfs.safemode.extension</name>
       <value>0</value>
     </property>
     <property>
        <name>dfs.safemode.min.datanodes</name>
        <value>1</value>
     </property>
     <property>
        <name>hadoop.tmp.dir</name>
        <value>/var/lib/hadoop-hdfs/cache/\\${user.name}</value>
     </property>
     <property>
        <name>dfs.namenode.name.dir</name>
        <value>file:///var/lib/hadoop-hdfs/cache/\\${user.name}/dfs/name</value>
     </property>
     <property>
        <name>dfs.namenode.checkpoint.dir</name>
        <value>file:///var/lib/hadoop-hdfs/cache/\\${user.name}/dfs/namesecondary</value>
     </property>
     <property>
        <name>dfs.datanode.data.dir</name>
        <value>file:///var/lib/hadoop-hdfs/cache/\\${user.name}/dfs/data</value>
     </property>
     <property>
       <name>dfs.client.read.shortcircuit</name>
       <value>true</value>
     </property>
     <property>
       <name>dfs.client.file-block-storage-locations.timeout.millis</name>
       <value>10000</value>
     </property>
     <property>
       <name>dfs.domain.socket.path</name>
       <value>/var/run/hadoop-hdfs/dn._PORT</value>
     </property>
     <property>
       <name>dfs.datanode.hdfs-blocks-metadata.enabled</name>
       <value>true</value>
     </property>
     <property>
       <name>dfs.namenode.rpc-bind-host</name>
       <value>0.0.0.0</value>
     </property>
     <property>
       <name>dfs.namenode.acls.enabled</name>
       <value>true</value>
     </property>
   </configuration>

HDFSCNF

  # format namenode
  if [ ! -e /var/lib/hadoop-hdfs/cache/hdfs ]; then
    echo "Formatting HDFS..." \
      && sudo -u hdfs hdfs namenode -format -force &> /dev/null
  fi

  MYSQL_JDBC=mysql-connector-java-5.1.37
  MYSQL_JDBC_SOURCE=http://dev.mysql.com/get/Downloads/Connector-J/${MYSQL_JDBC}.tar.gz

  mkdir -p /usr/local/lib/jdbc/mysql \
    && echo "Downloading MySQL JDBC drivers" \
    && wget ${MYSQL_JDBC_SOURCE} -q -P /tmp/ \
    && echo "Installing MySQL JDBC drivers" \
    && tar zxf /tmp/${MYSQL_JDBC}.tar.gz -C /tmp/ \
    && mkdir -p /usr/lib/oozie/libext \
    && cp /tmp/${MYSQL_JDBC}/mysql-connector-java*.jar /usr/lib/hive/lib/ \
    && cp /tmp/${MYSQL_JDBC}/mysql-connector-java*.jar /usr/local/lib/jdbc/mysql/ \
    && cp /tmp/${MYSQL_JDBC}/mysql-connector-java*.jar /usr/lib/oozie/libext/

  cat << HIVECNF > /etc/hive/conf/hive-site.xml

<configuration>
<property>
  <name>javax.jdo.option.ConnectionURL</name>
  <value>jdbc:mysql://localhost:3306/hive?createDatabaseIfNotExist=true</value>
</property>
<property>
  <name>javax.jdo.option.ConnectionUserName</name>
  <value>hive</value>
</property>
<property>
  <name>javax.jdo.option.ConnectionPassword</name>
  <value>hive</value>
</property>
<property>
  <name>javax.jdo.option.ConnectionDriverName</name>
  <value>com.mysql.jdbc.Driver</value>
</property>
<property>
  <name>hive.metastore.uris</name>
  <value>thrift://cdh.instance.com:9083</value>
</property>
<property>
  <name>hive.metastore.warehouse.dir</name>
  <value>hdfs:///user/hive/warehouse</value>
</property>
</configuration>

HIVECNF

  # auto-start services
  chkconfig hadoop-hdfs-namenode on \
    && chkconfig hadoop-hdfs-datanode on \
    && chkconfig hadoop-yarn-resourcemanager on \
    && chkconfig hadoop-yarn-nodemanager on \
    && chkconfig hadoop-mapreduce-historyserver on \
    && chkconfig hive-metastore on \
    && chkconfig hive-server2 on \
    && chkconfig oozie on

  # start Hadoop processses
  if [ ! "$(ps aux | grep hdfs-namenode | wc -l)" == "2" ]; then
    service hadoop-hdfs-namenode start
  fi

  if [ ! "$(ps aux | grep datanode | wc -l)" == "2" ]; then
    service hadoop-hdfs-datanode start
  fi

  if [ ! "$(ps aux | grep resourcemanager | wc -l)" == "2" ]; then
    service hadoop-yarn-resourcemanager start
  fi

  if [ ! "$(ps aux | grep nodemanager | wc -l)" == "2" ]; then
    service hadoop-yarn-nodemanager start
  fi

  echo "Creating HDFS directory structure" \
    && sudo -u hdfs hdfs dfs -mkdir -p {/user/{hadoop_oozie,spark,hive/warehouse,oozie/share/lib},/tmp,/jobs,/var/log/hadoop-yarn,/user/history} \
    && sudo -u hdfs hdfs dfs -chown -R hive:hive /user/hive \
    && sudo -u hdfs hdfs dfs -chown -R spark:spark /tmp/spark-logs \
    && sudo -u hdfs hdfs dfs -chown -R mapred:hadoop /user/history \
    && sudo -u hdfs hdfs dfs -chmod -R 1777 /user/history \
    && sudo -u hdfs hdfs dfs -chown -R oozie:oozie /user/oozie \
    && sudo -u hdfs hdfs dfs -chown -R hadoop_oozie:hadoop_oozie /user/hadoop_oozie \
    && sudo -u hdfs hdfs dfs -chown -R yarn:mapred /var/log/hadoop-yarn \
    && sudo -u hdfs hdfs dfs -chmod -R 1777 /

  # history server must start after hdfs privileges have been setup
  if [ ! "$(ps aux | grep historyserver | wc -l)" == "2" ]; then
    service hadoop-mapreduce-historyserver start
  fi

  # start Hive processses
  if [ ! "$(ps aux | grep HiveMetaStore | wc -l)" == "2" ]; then
    service hive-metastore start
  fi

  if [ ! "$(ps aux | grep HiveServer2 | wc -l)" == "2" ]; then
    service hive-server2 start
  fi

  # Oozie configuration
  echo "Deploying oozie-site.xml"
  cat << OOZCNF > /etc/oozie/conf/oozie-site.xml

<configuration>
<property>
  <name>oozie.service.JPAService.create.db.schema</name>
  <value>true</value>
</property>
<property>
  <name>oozie.service.JPAService.validate.db.connection</name>
  <value>true</value>
</property>
<property>
  <name>oozie.service.JPAService.jdbc.driver</name>
  <value>com.mysql.jdbc.Driver</value>
</property>
<property>
  <name>oozie.service.JPAService.jdbc.url</name>
  <value>jdbc:mysql://localhost:3306/oozie?createDatabaseIfNotExist=true</value>
</property>
<property>
  <name>oozie.service.JPAService.jdbc.username</name>
  <value>oozie</value>
</property>
<property>
  <name>oozie.service.JPAService.jdbc.password</name>
  <value>oozie</value>
</property>
<property>
  <name>oozie.service.ProxyUserService.proxyuser.oozie.hosts</name>
  <value>*</value>
</property>
<property>
  <name>oozie.service.ProxyUserService.proxyuser.oozie.groups</name>
  <value>*</value>
</property>
<property>
  <name>oozie.service.ProxyUserService.proxyuser.hue.hosts</name>
  <value>*</value>
</property>
<property>
  <name>oozie.service.ProxyUserService.proxyuser.hue.groups</name>
  <value>*</value>
</property>
<property>
  <name>use.system.libpath.for.mapreduce.and.pig.jobs</name>
  <value>true</value>
</property>
<property>
  <name>oozie.service.PurgeService.purge.old.coord.action</name>
  <value>true</value>
</property>
<property>
  <name>oozie.use.system.libpath</name>
  <value>true</value>
</property>
<property>
  <name>oozie.credentials.credentialclasses</name>
  <value>
    hcat=com.github.bartekdobija.oozieutils.creds.TestCreds,
    hive=com.github.bartekdobija.oozieutils.creds.TestCreds,
    hbase=com.github.bartekdobija.oozieutils.creds.TestCreds
  </value>
</property>
</configuration>

OOZCNF

  OOZIE_UTILS=https://github.com/bartekdobija/oozie-utils/releases/download/0.7/oozieutils-0.7.jar

  # create an Oozie database if not exists and upload sharelib
  if [ ! -f /var/lib/mysql/oozie/WF_JOBS.frm ]; then

    mkdir -p /user/oozie/share/lib \
      && chown -R oozie:oozie /user/oozie \
      && rm -fR /etc/oozie/conf/hadoop-conf \
      && ln -s /etc/hadoop/conf /etc/oozie/conf/hadoop-conf

    echo "Creating Oozie database" \
      && /usr/lib/oozie/bin/ooziedb.sh create -run \
      && chown -R oozie:oozie /var/log/oozie \
      && sudo -u oozie /usr/lib/oozie/bin/oozie-setup.sh sharelib create \
        -fs hdfs://localhost/user/oozie/share/lib/ \
        -locallib /usr/lib/oozie/oozie-sharelib \
      && rm -fR /usr/lib/oozie/libserver/oozieutils* \
      && wget ${OOZIE_UTILS} -q -P /usr/lib/oozie/libserver/
  fi

  echo "registering Spark configuration in Oozie" \
    && ln -f -s /opt/spark/conf /etc/oozie/conf/spark-conf

  if [ ! "$(ps aux | grep oozie | wc -l)" == "2" ]; then
    service oozie start
  fi

  echo "export OOZIE_URL=http://localhost:11000/oozie" > /etc/profile.d/oozie.sh

SCRIPT

# OS configuration
$system_config = <<SCRIPT

  # disable IPv6
  if [ "$(grep disable_ipv6 /etc/sysctl.conf | wc -l)" == "0" ]; then
    echo "net.ipv6.conf.all.disable_ipv6=1" >> /etc/sysctl.conf \
      && sysctl -f /etc/sysctl.conf
  fi

  # this should be a persistent config
  ulimit -n 65536
  ulimit -s 10240
  ulimit -c unlimited


  DEV_USER=hadoop_oozie
  DEV_PASSWORD=hadoop
  PROXY_CONFIG=/etc/profile.d/proxy.sh

  service iptables stop && chkconfig iptables off

  if grep ryanair /etc/resolv.conf; then
    echo "export http_proxy=http://internalproxy.corp.ryanair.com:3128" > ${PROXY_CONFIG} \
      && echo "export https_proxy=http://internalproxy.corp.ryanair.com:3128" >> ${PROXY_CONFIG}
  else
    rm -fR ${PROXY_CONFIG}
  fi

  # Add entries to /etc/hosts
  ip=$(ifconfig eth1 | awk -v host=$(hostname) '/inet addr/ {print substr($2,6)}')
  host=$(hostname)
  echo "127.0.0.1 localhost" > /etc/hosts
  echo "$ip $host" >> /etc/hosts

  # Add a dev user - don't worry about the password
  if ! grep ${DEV_USER} /etc/passwd; then
    echo "Creating user ${DEV_USER}" && useradd -p $(openssl passwd -1 ${DEV_PASSWORD}) ${DEV_USER} \
      && echo "${DEV_USER}  ALL=(ALL)  NOPASSWD:  ALL" > /etc/sudoers.d/hadoop_oozie
  fi

#  if [ "$(grep vm.swappiness /etc/sysctl.conf | wc -l)" == "0" ]; then
#    echo "vm.swappiness=10" >> /etc/sysctl.conf && sysctl vm.swappiness=10
#  fi

SCRIPT

$javajdk = <<SCRIPT
  if [[ ! -e /usr/java/default ]]; then
    wget -q --no-cookies --no-check-certificate --header "Cookie: oraclelicense=accept-securebackup-cookie" "http://download.oracle.com/otn-pub/java/jdk/8u151-b12/e758a0de34e24606bca991d704f6dcbf/jdk-8u151-linux-x64.rpm" \
      && yum -y remove java-1.6* \
      && rpm -i jdk-8u151-linux-x64.rpm
    echo "export JAVA_HOME=/usr/java/default" > /etc/profile.d/java.sh
  fi
SCRIPT

$information = <<SCRIPT
  ip=$(ifconfig eth1 | awk -v host=$(hostname) '/inet addr/ {print substr($2,6)}')
  echo "Guest IP address: $ip"
  echo "Namenode UI available at: http://$ip:50070"
  echo "Resource Manager UI available at: http://$ip:8088"
  echo "Oozie endpoint available at: http://$ip:11000/oozie"
  echo "Spark historyserver available at: http://$ip:18080"
  echo "Spark 1.6 available under /opt/spark"
  echo "MySQL root password: hadoop"
  echo "You may want to add the below line to /etc/hosts:"
  echo "$ip cdh.instance.com"
SCRIPT

Vagrant.configure(2) do |config|

  config.vm.box = "boxcutter/centos66"
  config.vm.hostname = "cdh.instance.com"
  config.vm.network :public_network

  config.vm.provider "virtualbox" do |vb|
    vb.name = "dev-spark-env"
    vb.cpus = 4
    vb.memory = 8192
    vb.customize ["modifyvm", :id, "--nicpromisc2", "allow-all"]
    vb.customize ["modifyvm", :id, "--cpuexecutioncap", "100"]
  end

  config.vm.provision :shell, :name => "system_config", :inline => $system_config
  config.vm.provision :shell, :name => "javajdk", :inline => $javajdk
  #config.vm.provision :shell, :name => "anaconda_deps", :inline => $anaconda_deps
  config.vm.provision :shell, :name => "mysql_deps", :inline => $mysql_deps
  config.vm.provision :shell, :name => "spark_deps", :inline => $spark_deps
  config.vm.provision :shell, :name => "cloudera_deps", :inline => $cloudera_deps
  config.vm.provision :shell, :name => "information", :inline => $information

end
