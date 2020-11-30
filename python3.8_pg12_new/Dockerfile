FROM  jenkins/inbound-agent:latest-jdk11 

LABEL Abhishek Jalan <abhishekjalan1987@gmail.com> 

USER root

ENV POSTGRES_VERSION 12
ENV PYTHON_VERSION 3.8.6
ENV PYTHON_PIP 3

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

#Upgrade Openssl for Python3.8.6
RUN apt-get install -y openssl make build-essential libssl-dev zlib1g-dev libbz2-dev \
    libreadline-dev libsqlite3-dev wget curl llvm libncurses5-dev libncursesw5-dev \
    xz-utils tk-dev libffi-dev liblzma-dev libgdal-dev python${PYTHON_PIP}-dev python${PYTHON_PIP}-pip zip gettext-base
RUN wget --force-html "https://www.python.org/ftp/python/${PYTHON_VERSION}/Python-${PYTHON_VERSION}.tgz" && \
    tar -xvf Python-${PYTHON_VERSION}.tgz && \
    cd Python-${PYTHON_VERSION} && ./configure && sudo apt-get install zlib1g-dev && sudo make -j 4 && sudo make install
RUN /usr/local/bin/python3.8 -m pip install --upgrade pip
RUN rm -f Python-${PYTHON_VERSION}.tgz && \
    unlink /usr/bin/python && \
    ln -s /usr/local/bin/python3.8 /usr/bin/python && \
    ln -s /usr/local/bin/pip${PYTHON_PIP} /usr/bin/pip

RUN pip install ansible 
RUN mkdir -p /home/jenkins/.ansible && \
    mkdir -p /home/jenkins/.ssh && \
    chown -R 1000:1000 /home/jenkins/.ansible && \
    chown -R 1000:1000 /home/jenkins/.ssh

##Install Docker Compose
RUN sudo curl -L "https://github.com/docker/compose/releases/download/1.26.0/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
RUN sudo chmod +x /usr/local/bin/docker-compose

##Install AWS CLI
RUN sudo apt-get install -y awscli 

##Install PG
RUN apt-get install -y wget ca-certificates build-essential curl && \
    wget --quiet -O - https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo apt-key add -
RUN sh -c 'echo "deb http://apt.postgresql.org/pub/repos/apt/ $(lsb_release -sc)-pgdg main" > /etc/apt/sources.list.d/PostgreSQL.list'
RUN apt-get update && \
    apt-get install -y postgresql-${POSTGRES_VERSION} postgresql-client-${POSTGRES_VERSION} postgresql-contrib-${POSTGRES_VERSION} gnupg2 && \
    sudo sh -c "echo \"jenkins ALL=(ALL) NOPASSWD: ALL\" >> /etc/sudoers" && \
    sudo sh -c "echo \"postgres ALL=(ALL) NOPASSWD: ALL\" >> /etc/sudoers"

USER postgres

RUN    /etc/init.d/postgresql start &&\
    psql --command "CREATE USER docker WITH SUPERUSER PASSWORD 'docker';" &&\
    createdb -O docker docker
RUN echo "host all  all    0.0.0.0/0  md5" >> /etc/postgresql/${POSTGRES_VERSION}/main/pg_hba.conf && \
    echo "listen_addresses='*'" >> /etc/postgresql/${POSTGRES_VERSION}/main/postgresql.conf

USER root
RUN echo "deb http://apt.postgresql.org/pub/repos/apt/ `lsb_release -cs`-pgdg main" |sudo tee  /etc/apt/sources.list.d/pgdg.list
RUN apt-get update \
    && apt-get install -y postgis postgresql-${POSTGRES_VERSION}-postgis-3


EXPOSE 5432

USER jenkins