FROM tomcat:8.0

MAINTAINER Julia Damerow https://github.com/jdamerow

# run installs in noninteractive mode
ENV DEBIAN_FRONTEND noninteractive

RUN apt-get -y update
RUN apt-get install -y maven
RUN apt-get install -y git-all
#RUN apt-get install -y mysql-server
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
ARG giles_url=http://giles:8080/giles/
ARG request_version=v0.2.1-SNAPSHOT
ARG util_version=v0.4.2-SNAPSHOT
ARG geco.septemberutil.version=v0.2

# install giles-eco-requests
RUN mkdir gitrepos
WORKDIR gitrepos
RUN git clone https://github.com/diging/giles-eco-requests.git
WORKDIR giles-eco-requests/giles-eco-requests
RUN mvn clean install -Dgeco.requests.version=$request_version

# install giles-eco-util
WORKDIR ../..
RUN git clone https://github.com/diging/giles-eco-util.git
WORKDIR giles-eco-util/util
RUN mvn clean install -Dgeco.util.version=$util_version

# install september-util
WORKDIR ../..
RUN git clone https://github.com/diging/giles-eco-september-util.git
WORKDIR giles-eco-september-util/september-util
RUN mvn clean install -Dgeco.septemberutil.version=$geco.septemberutil.version -Dgeco.requests.version=$request_version -Dgeco.util.version=$util_version

# install spring social
WORKDIR ../..
RUN git clone https://github.com/jdamerow/spring-social-github.git
WORKDIR spring-social-github
RUN ./gradlew publishToMavenLocal

# install mitreid
WORKDIR ../..
RUN git clone https://github.com/jdamerow/spring-social-mitreid-connect.git
WORKDIR spring-social-mitreid-connect/mitreid-connect
RUN mvn clean install

# build giles
WORKDIR ../..
RUN git clone https://github.com/diging/giles-eco-giles-web.git
WORKDIR giles-eco-giles-web/giles-eco
RUN mvn clean package -P prod -Dgeco.giles.db_files=/data/dbs/giles/ -Dlog.level=info -Dgiles.kafka.hosts=$kafka_host -Dgiles.base.url=$giles_url -Dmitreid.show.login=false -Ddb.database.url=jdbc:mysql://mysql:3306/giles -Ddb.user=giles -Ddb.password=GilesPassword -Dgiles.file.tmp.dir=/usr/local/tomcat/giles-work/tmp -Ddigilib.url=http://digilib:8080/digilib/servlet/Scaler/ -Dgeco.requests.version=$request_version -Dzookeeper.host=zookeeper -Demail.enabled=false -Dgeco.util.version=$util_version -Dgeco.september.util.version=$geco.septemberutil.version

# deploy Giles
RUN cp ./target/giles.war /usr/local/tomcat/webapps/

WORKDIR /usr/local/tomcat/
RUN mkdir giles-work
WORKDIR giles-work
RUN mkdir tmp

COPY scripts/startup /tmp/
RUN chmod 755 /tmp/startup

CMD /tmp/startup
