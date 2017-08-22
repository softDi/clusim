FROM ubuntu:16.04
MAINTAINER Cheng-Yueh Liu && Yuan-Di Li <cyliustack@gmail.com && lisoftdi@gmail.com> 
RUN apt-get update && apt-get install -y openssh-server sshpass gcc g++ make  iputils-ping git vim wget gfortran iperf
RUN mkdir /var/run/sshd
RUN echo 'root:root' | chpasswd
RUN sed -i 's/PermitRootLogin prohibit-password/PermitRootLogin yes/' /etc/ssh/sshd_config

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

#install flink 
RUN cd /root && wget https://archive.apache.org/dist/flink/flink-1.2.1/flink-1.2.1-bin-hadoop27-scala_2.11.tgz
RUN cd /root && tar zxvf flink-1.2.1-bin-hadoop27-scala_2.11.tgz && mv flink-1.2.1 flink
RUN cd /root/flink/conf && rm masters && wget https://raw.githubusercontent.com/softDi/clusim/yuandi/vcluster/flink/masters
RUN cd /root/flink/conf && rm slaves && wget https://raw.githubusercontent.com/softDi/clusim/yuandi/vcluster/flink/slaves
RUN cd /root/flink/conf && sed -i='' {s/localhost/172\.17\.0\.2/g} flink-conf.yaml 
RUN cd /root/flink/ && mkdir benchmark && wget -O hamlet.txt http://www.gutenberg.org/cache/epub/1787/pg1787.txt
RUN cd /root/flink/benchmark && git clone https://github.com/aritter/twitter_nlp && git clone https://github.com/citlab/flink-realtime-streaming/ && git clone https://github.com/tillrohrmann/cep-monitoring && git clone https://github.com/neoluc/Spark-Streaming-PageRank


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

#RUN 
CMD ["/usr/sbin/sshd", "-D"]





