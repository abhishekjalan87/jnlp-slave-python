FROM  jenkins/inbound-agent:latest-jdk11 

USER root

ENV POSTGRES_VERSION 12
ENV PYTHON_VERSION 3.8.6
ENV PIP_VERSION 3

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
RUN apt-get -y install openssl

##Install Python3.8.3 & Pip3
RUN wget --force-html "https://www.python.org/ftp/python/${PYTHON_VERSION}/Python-${PYTHON_VERSION}.tgz"
RUN tar -xvf Python-${PYTHON_VERSION}.tgz
RUN cd Python-${PYTHON_VERSION} && ./configure --with-ssl && sudo apt-get install zlib1g-dev && sudo make && sudo make install
RUN sudo apt-get -y update && apt-get install -y python3-dev libffi-dev libssl-dev libgdal-dev python3-pip zip gettext-base 
RUN sudo ln -sf /usr/local/bin/python3 /usr/bin/python

# RUN sudo ln -sf /usr/local/bin/python3 /usr/bin/python && \
#     sudo ln -sf /usr/local/bin/pip${PIP_VERSION} /usr/bin/pip
RUN python -V && \
    pip3 -V
RUN which openssl

RUN pip3 install ansible 
RUN mkdir -p /home/jenkins/.ansible && \
    mkdir -p /home/jenkins/.ssh && \
    chown -R 1000:1000 /home/jenkins/.ansible && \
    chown -R 1000:1000 /home/jenkins/.ssh

##Install Docker Compose
RUN sudo curl -L "https://github.com/docker/compose/releases/download/1.26.0/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
RUN sudo chmod +x /usr/local/bin/docker-compose
RUN docker-compose --version

##Install AWS CLI
RUN sudo apt-get install -y awscli 

##Install PG
RUN apt-get install -y wget ca-certificates build-essential curl
RUN wget --quiet -O - https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo apt-key add -
RUN sh -c 'echo "deb http://apt.postgresql.org/pub/repos/apt/ $(lsb_release -sc)-pgdg main" > /etc/apt/sources.list.d/PostgreSQL.list'
RUN apt-get update
RUN apt-get install -y postgresql-${POSTGRES_VERSION} postgresql-client-${POSTGRES_VERSION} postgresql-contrib-${POSTGRES_VERSION} gnupg2
RUN sudo sh -c "echo \"jenkins ALL=(ALL) NOPASSWD: ALL\" >> /etc/sudoers"
RUN sudo sh -c "echo \"postgres ALL=(ALL) NOPASSWD: ALL\" >> /etc/sudoers"

USER postgres

RUN    /etc/init.d/postgresql start &&\
    psql --command "CREATE USER docker WITH SUPERUSER PASSWORD 'docker';" &&\
    createdb -O docker docker
RUN echo "host all  all    0.0.0.0/0  md5" >> /etc/postgresql/${POSTGRES_VERSION}/main/pg_hba.conf
RUN echo "listen_addresses='*'" >> /etc/postgresql/${POSTGRES_VERSION}/main/postgresql.conf

USER root
RUN echo "deb http://apt.postgresql.org/pub/repos/apt/ `lsb_release -cs`-pgdg main" |sudo tee  /etc/apt/sources.list.d/pgdg.list
RUN apt-get update \
    && apt-get install -y postgis postgresql-${POSTGRES_VERSION}-postgis-3


EXPOSE 5432

USER jenkins