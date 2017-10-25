FROM ubuntu:16.04
MAINTAINER Cheng-Yueh Liu && Yuan-Di Li <cyliustack@gmail.com && lisoftdi@gmail.com>
RUN apt-get update && apt-get install -y openssh-server sshpass gcc g++ make  iputils-ping git vim wget gfortran iperf python3-pip python-pip
RUN mkdir /var/run/sshd
RUN echo 'root:root' | chpasswd
RUN sed -i 's/PermitRootLogin prohibit-password/PermitRootLogin yes/' /etc/ssh/sshd_config
RUN apt-get update && apt-get install bzip2 && apt-get install curl && apt-get install -y zip

# SSH login fix. Otherwise user is kicked off after login
RUN sed 's@session\s*required\s*pam_loginuid.so@session optional pam_loginuid.so@g' -i /etc/pam.d/sshd

ENV NOTVISIBLE "in users profile"
RUN echo "export VISIBLE=now" >> /etc/profile
EXPOSE 22
RUN echo -e  'y\n'| ssh-keygen -N ''  -f /root/.ssh/id_rsa
RUN echo "root" > /root/password.txt
RUN cat /root/password.txt
RUN cd /root && git clone https://github.com/cyliustack/benchmark
RUN cp /root/benchmark/tools/ssh_copy_id_to_all.sh /root/
RUN wget http://www.mpich.org/static/downloads/3.2/mpich-3.2.tar.gz
RUN tar -xvf mpich-3.2.tar.gz
RUN cd mpich-3.2 && ./configure --prefix=/opt/mpich && make -j4 && make install
RUN cd /root/benchmark/mpi/gemm && make
RUN echo export PATH=/opt/mpich/bin:$PATH >> ~/.bashrc
RUN echo export TERM=xterm >> ~/.bashrc
RUN echo export LD_LIBRARY_PATH=/opt/mpich/lib:$LD_LIBRARY_PATH >> ~/.bashrc
RUN echo "    StrictHostKeyChecking no" >> /etc/ssh/ssh_config
RUN cp /root/benchmark/mpi/tools/demo.sh /root

#install java8
RUN apt-get install -y software-properties-common
RUN add-apt-repository ppa:webupd8team/java -y && apt-get update
RUN echo oracle-java7-installer shared/accepted-oracle-license-v1-1 select true | /usr/bin/debconf-set-selections && apt-get install -y oracle-java8-installer
RUN apt-get -y install oracle-java8-set-default
RUN echo "JAVA_HOME=/usr/lib/jvm/java-8-oracle" >> /etc/environment && echo "JRE_HOME=/usr/lib/jvm/java-8-oracle/jre" >> /etc/environment
RUN echo "export JAVA_HOME=/usr/lib/jvm/java-8-oracle" >> ~/.bashrc && echo "export JRE_HOME=/usr/lib/jvm/java-8-oracle/jre" >> ~/.bashrc
#install maven
RUN cd /usr/local/ && wget http://www-eu.apache.org/dist/maven/maven-3/3.3.9/binaries/apache-maven-3.3.9-bin.tar.gz
RUN cd /usr/local/ && tar xzf apache-maven-3.3.9-bin.tar.gz && ln -s apache-maven-3.3.9 apache-maven
RUN cd /etc/profile.d/ && wget https://raw.githubusercontent.com/softDi/clusim/yuandi/vcluster/flink/apache-maven.sh

#install anaconda
#RUN cd /root && wget http://repo.continuum.io/archive/Anaconda2-4.3.0-Linux-x86_64.sh && wget http://repo.continuum.io/archive/Anaconda3-4.3.0-Linux-x86_64.sh

#install scala && sbt
ENV SCALA_VERSION 2.12.3
ENV SBT_VERSION 0.13.16
# Scala expects this file
RUN touch /usr/lib/jvm/java-8-oracle/release
# Install Scala
## Piping curl directly in tar
RUN curl -fsL https://downloads.typesafe.com/scala/$SCALA_VERSION/scala-$SCALA_VERSION.tgz | tar xfz - -C /root/ && echo >> /root/.bashrc && echo 'export PATH=~/scala-$SCALA_VERSION/bin:$PATH' >> /root/.bashrc
# Install sbt
RUN curl -L -o sbt-$SBT_VERSION.deb https://dl.bintray.com/sbt/debian/sbt-$SBT_VERSION.deb && dpkg -i sbt-$SBT_VERSION.deb && rm sbt-$SBT_VERSION.deb && apt-get update && apt-get install sbt && sbt sbtVersion


#install flink
RUN cd /root && wget https://archive.apache.org/dist/flink/flink-1.2.1/flink-1.2.1-bin-hadoop27-scala_2.11.tgz
RUN cd /root && tar zxvf flink-1.2.1-bin-hadoop27-scala_2.11.tgz && mv flink-1.2.1 flink
RUN cd /root/flink/conf && rm masters && wget https://raw.githubusercontent.com/softDi/clusim/yuandi/vcluster/flink/masters
RUN cd /root/flink/conf && rm slaves && wget https://raw.githubusercontent.com/softDi/clusim/yuandi/vcluster/flink/slaves
RUN cd /root/flink/conf && sed -i='' {s/localhost/172\.17\.0\.2/g} flink-conf.yaml
RUN cd /root/flink/conf && echo "fs.hdfs.hadoopconf: /usr/local/hadoop/" >> flink-conf.yaml 
RUN cd /root/flink/ && mkdir benchmark && wget -O hamlet.txt http://www.gutenberg.org/cache/epub/1787/pg1787.txt
RUN cd /root/flink/benchmark && git clone https://github.com/aritter/twitter_nlp  && git clone https://github.com/tillrohrmann/cep-monitoring && git clone https://github.com/FlinkML/flink-tensorflow.git && git clone https://github.com/brakmic/TwitterClient.git && git clone https://github.com/brakmic/TwitterFlink.git
RUN cd /root/flink/benchmark/TwitterClient/ && apt-get install -y python-pyodbc
#https://github.com/jiangwen365/pypyodbc && https://github.com/tweepy/tweepy && https://github.com/avian2/unidecode
RUN pip3 install tweepy && pip3 install unidecode && pip3 install pypyodbc && pip3 install colorama
RUN pip install tweepy && pip install unidecode && pip install pypyodbc && pip install colorama
#flink-tensorflow
#https://github.com/FlinkML/flink-tensorflow
RUN cd /root/flink/benchmark/flink-tensorflow/ && wget http://storage.googleapis.com/download.tensorflow.org/models/inception5h.zip

#install tensorflow
RUN pip3 install tensorflow && pip install tensorflow
RUN cd /root/ && git clone https://github.com/tensorflow/benchmarks.git tensorflowbenchmarks

#install hdfs
RUN wget http://www.us.apache.org/dist/hadoop/common/hadoop-2.7.4/hadoop-2.7.4.tar.gz && tar -zxf ./hadoop-2.7.4.tar.gz -C /usr/local
RUN cd /usr/local/ && mv ./hadoop-2.7.4/ ./hadoop 
RUN cd /usr/local/hadoop/etc/hadoop/ && rm -f slaves core-site.xml hdfs-site.xml mapred-site.xml.template yarn-site.xml
RUN cd /usr/local/hadoop/etc/hadoop/ && wget https://raw.githubusercontent.com/softDi/clusim/yuandi/vcluster/flink/hdfs/slaves && wget https://raw.githubusercontent.com/softDi/clusim/yuandi/vcluster/flink/hdfs/core-site.xml && wget https://raw.githubusercontent.com/softDi/clusim/yuandi/vcluster/flink/hdfs/hdfs-site.xml && wget https://raw.githubusercontent.com/softDi/clusim/yuandi/vcluster/flink/hdfs/mapred-site.xml && wget https://raw.githubusercontent.com/softDi/clusim/yuandi/vcluster/flink/hdfs/yarn-site.xml
RUN echo "export HADOOP_HOME=/usr/local/hadoop/" >> ~/.bashrc 

#RUN
CMD ["/usr/sbin/sshd", "-D"]





