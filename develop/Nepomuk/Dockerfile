FROM tomcat:8.0

MAINTAINER Julia Damerow https://github.com/jdamerow

# run installs in noninteractive mode
ENV DEBIAN_FRONTEND noninteractive

RUN apt-get -y update
RUN apt-get install -y maven
RUN apt-get install -y git-all
RUN apt-get install -y subversion

# install JDK 8
RUN \
  echo "deb http://ppa.launchpad.net/webupd8team/java/ubuntu precise main" | tee -a /etc/apt/sources.list  && \
  echo "deb-src http://ppa.launchpad.net/webupd8team/java/ubuntu precise main" | tee -a /etc/apt/sources.list && \
  echo oracle-java8-installer shared/accepted-oracle-license-v1-1 select true | /usr/bin/debconf-set-selections && \
  apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv-keys EEA14886 && \
  apt-get -y update && \
  apt-get -y install oracle-java8-installer && \
  rm -rf /var/lib/apt/lists/*

# Set environment variables.
ENV JAVA_HOME /usr/lib/jvm/java-8-oracle

ARG kafka_host=kafka:9092
ARG nepomuk_url=http://nepomuk:8080/nepomuk/
ARG request_version=v0.2.1-SNAPSHOT
ARG util_version=v0.4.2-SNAPSHOT
ARG geco.septemberutil.version=v0.2

RUN mkdir gitrepos
WORKDIR gitrepos
RUN git clone https://github.com/diging/giles-eco-requests.git && cd giles-eco-requests/giles-eco-requests && mvn clean install -Dgeco.requests.version=$request_version

RUN git clone https://github.com/diging/giles-eco-util.git && cd giles-eco-util/util && mvn clean install -Dgeco.util.version=$util_version

# install september-util
RUN git clone https://github.com/diging/giles-eco-september-util.git && cd giles-eco-september-util/september-util && mvn clean install -Dgeco.septemberutil.version=$geco.septemberutil.version -Dgeco.requests.version=$request_version -Dgeco.util.version=$util_version

RUN git clone https://github.com/diging/giles-eco-nepomuk.git
WORKDIR giles-eco-nepomuk/nepomuk
RUN mvn clean package -Dnepomuk.db.files=/data/dbs/nepomuk/ -Dnepomuk.digilibBaseDir=/data -Dnepomuk.images.folder=imgs -Dnepomuk.baseDir=/data/ -Dnepomuk.texts.folder=texts -Dnepomuk.pdfs.folder=pdfs -Dnepomuk.otherFiles.folder=other -Dlog.level=info -Dnepomuk.kafka.hosts=$kafka_host -Dnepomuk.base.url=$nepomuk_url -Dgeco.requests.version=$request_version -Dgeco.util.version=$util_version -Dzookeeper.host=zookeeper -Ddb.database.url=jdbc:mysql://mysql:3306/nepomuk -Ddb.user=nepomuk -Ddb.password=NepomukPassword -Dgeco.september.util.version=$geco.septemberutil.version

# deploy Nepomuk
RUN cp ./target/nepomuk.war /usr/local/tomcat/webapps/

WORKDIR /usr/local/tomcat/
RUN mkdir persist
WORKDIR persist
RUN mkdir db
#WORKDIR ..
#RUN mkdir files
#WORKDIR files
#RUN mkdir imgs
#RUN mkdir texts
#RUN mkdir pdfs
#RUN mkdir other
