FROM  jenkins/inbound-agent:latest-jdk11 

USER root
RUN apt-get update && \
    apt-get -y install apt-transport-https \
    ca-certificates \
    curl \
    gnupg2 \
    software-properties-common && \
    curl -fsSL https://download.docker.com/linux/$(. /etc/os-release; echo "$ID")/gpg > /tmp/dkey; apt-key add /tmp/dkey && \
    add-apt-repository \
    "deb [arch=amd64] https://download.docker.com/linux/$(. /etc/os-release; echo "$ID") \
    $(lsb_release -cs) \
    stable" && \
    apt-get update && \
    apt-get -y install docker-ce 
RUN usermod -a -G docker jenkins
RUN apt-get update && apt-get install -y python3.7 python3-pip libgdal-dev zip gettext-base
RUN ln -sf /usr/bin/python3.7 /usr/bin/python && \
    ln -sf /usr/bin/pip3.7 /usr/bin/pip
RUN pip3 install ansible 
RUN mkdir -p /home/jenkins/.ansible && \
    mkdir -p /home/jenkins/.ssh && \
    chown -R 1000:1000 /home/jenkins/.ansible && \
    chown -R 1000:1000 /home/jenkins/.ssh

##Install PG
RUN apt-get install -y wget ca-certificates build-essential curl
RUN wget --quiet -O - https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo apt-key add -
RUN sh -c 'echo "deb http://apt.postgresql.org/pub/repos/apt/ $(lsb_release -sc)-pgdg main" > /etc/apt/sources.list.d/PostgreSQL.list'
RUN apt-get update
RUN apt-get install -y postgresql-12 postgresql-client-12 postgresql-contrib-12 gnupg2
RUN sudo sh -c "echo \"jenkins ALL=(ALL) NOPASSWD: ALL\" >> /etc/sudoers"
RUN sudo sh -c "echo \"postgres ALL=(ALL) NOPASSWD: ALL\" >> /etc/sudoers"

USER postgres

RUN    /etc/init.d/postgresql start &&\
    psql --command "CREATE USER docker WITH SUPERUSER PASSWORD 'docker';" &&\
    createdb -O docker docker
RUN echo "host all  all    0.0.0.0/0  md5" >> /etc/postgresql/12/main/pg_hba.conf
RUN echo "listen_addresses='*'" >> /etc/postgresql/12/main/postgresql.conf

USER root
RUN echo "deb http://apt.postgresql.org/pub/repos/apt/ `lsb_release -cs`-pgdg main" |sudo tee  /etc/apt/sources.list.d/pgdg.list
RUN apt-get update \
    && apt-get install -y postgis postgresql-12-postgis-3


EXPOSE 5432

USER jenkins
