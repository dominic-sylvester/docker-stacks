# Source: https://hub.docker.com/r/jupyter/pyspark-notebook
# Copyright (c) Jupyter Development Team.
# Distributed under the terms of the Modified BSD License.
ARG BASE_CONTAINER=jupyter/scipy-notebook
FROM $BASE_CONTAINER

LABEL maintainer="simicd"

USER root

# Spark dependencies
ENV APACHE_SPARK_VERSION 2.4.4
ENV HADOOP_VERSION 2.7

RUN apt-get -y update && \
    apt-get install --no-install-recommends -y openjdk-8-jre-headless ca-certificates-java && \
    rm -rf /var/lib/apt/lists/*

RUN cd /tmp && \
    wget -q http://mirrors.ukfast.co.uk/sites/ftp.apache.org/spark/spark-${APACHE_SPARK_VERSION}/spark-${APACHE_SPARK_VERSION}-bin-hadoop${HADOOP_VERSION}.tgz && \
    echo "2E3A5C853B9F28C7D4525C0ADCB0D971B73AD47D5CCE138C85335B9F53A6519540D3923CB0B5CEE41E386E49AE8A409A51AB7194BA11A254E037A848D0C4A9E5 *spark-${APACHE_SPARK_VERSION}-bin-hadoop${HADOOP_VERSION}.tgz" | sha512sum -c - && \
    tar xzf spark-${APACHE_SPARK_VERSION}-bin-hadoop${HADOOP_VERSION}.tgz -C /usr/local --owner root --group root --no-same-owner && \
    rm spark-${APACHE_SPARK_VERSION}-bin-hadoop${HADOOP_VERSION}.tgz
RUN cd /usr/local && ln -s spark-${APACHE_SPARK_VERSION}-bin-hadoop${HADOOP_VERSION} spark

# Mesos dependencies
# Install from the Xenial Mesosphere repository since there does not (yet)
# exist a Bionic repository and the dependencies seem to be compatible for now.
# COPY mesos.key /tmp/
# RUN apt-get -y update && \
#     apt-get install --no-install-recommends -y gnupg && \
#     apt-key add /tmp/mesos.key && \
#     echo "deb http://repos.mesosphere.io/ubuntu xenial main" > /etc/apt/sources.list.d/mesosphere.list && \
#     apt-get -y update && \
#     apt-get --no-install-recommends -y install mesos=1.2\* && \
#     apt-get purge --auto-remove -y gnupg && \
#     rm -rf /var/lib/apt/lists/*

# Spark and Mesos config
ENV JAVA_HOME /usr/lib/jvm/java-8-openjdk-amd64/
ENV SPARK_HOME /usr/local/spark
ENV PYTHONPATH $SPARK_HOME/python:$SPARK_HOME/python/lib/py4j-0.10.7-src.zip
ENV MESOS_NATIVE_LIBRARY /usr/local/lib/libmesos.so
ENV SPARK_OPTS --driver-java-options=-Xms1024M --driver-java-options=-Xmx4096M --driver-java-options=-Dlog4j.logLevel=info
# Define USERPROFILE environment variable to match Windows so that PowerShell script can successfully run
ENV USERPROFILE /home/$NB_USER



### Set the working directory to /powershell
WORKDIR /powershell

# Install .NET Core 3.0 and PowerShell
# Source: https://dotnet.microsoft.com/download/linux-package-manager/ubuntu18-04/sdk-3.0.100
RUN wget -q https://packages.microsoft.com/config/ubuntu/18.04/packages-microsoft-prod.deb -O packages-microsoft-prod.deb && \
    sudo dpkg -i packages-microsoft-prod.deb && \
    sudo apt-get update && \
    sudo apt-get install -y software-properties-common && \
    sudo apt-get update && \
    sudo add-apt-repository universe && \
    sudo apt-get update && \
    sudo apt-get install apt-transport-https && \
    sudo apt-get update && \
    sudo apt-get install -y dotnet-sdk-3.1 && \
    sudo apt-get install -y powershell && \
    rm packages-microsoft-prod.deb && \
    apt-get clean

# Upgrade git to newest version (requires changing repository source to git-core)
RUN sudo add-apt-repository ppa:git-core/ppa -y && \
    sudo apt-get update && \
    sudo apt-get install git -y

# Copy the Spark configuration file to the conf folder where Spark is installed
# This includes for instance additional packages such as Delta Lake which will
# be included in each SparkSession
# Additionally, copy the logger settings in log4j.properties
# (set log level to WARN messages and higher instead of INFO)
COPY spark-defaults.conf $SPARK_HOME/conf/
COPY log4j.properties $SPARK_HOME/conf/


# Switch from root/superuser to normal user and change directory
WORKDIR /home/$NB_USER
USER $NB_UID

# Create the folder /repos which will be linked to a data volume with the current user
# It's possible to mount a data volume without creating the folder here first
# but then access is given to 'root' user only
RUN mkdir /home/$NB_USER/repos

# Get the PowerShell script and execute
# This installs Jupyter extensions, additional Pyton packages and applies Jupyter styling
COPY workplace-setup.ps1 /tmp/
RUN pwsh -c "/tmp/workplace-setup.ps1"

# Give permissions to the user for all files & folders under /home/username
RUN fix-permissions $CONDA_DIR && \
    fix-permissions /home/$NB_USER
