>  Enrique Catalá Bañuls
>> enrique@enriquecatala.com
>
>> [Twitter](https://twitter.com/enriquecatala) |
>> [Linkedin](https://www.linkedin.com/in/enriquecatala) |
>> [MVP Profile](https://mvp.microsoft.com/es-es/PublicProfile/5000312?fullName=Enrique%20Catala)

# mssql-server-samplesdb


This project will create a docker image with all the sample databases restored.



## How to create the image

You can create the image with the following command:

```cmd
docker-compose build
```

## How to run the container

```cmd
docker-compose up --build
```
_With the --build you are first building the image, and then instantiating the container_

**IMPORTANT:** StackOverflow2010 database is huge and it will require a couple of minutes to initialize. Please be patient. You can work and play within the other databases while the StackOverflow database is being prepared

## How to change the SQL Server base image

The [Dockerfile](./Dockerfile) specifies which base SQL Server Instance you want to use for your image. 

In case you want to change the version of the SQL Server used, please go edit the first line of the [Dockerfile](./Dockerfile)  and select your prefered version. For example

Change 

```docker
FROM mcr.microsoft.com/mssql/server:2019-GA-ubuntu-16.04
```

To

```docker
FROM mcr.microsoft.com/mssql/server:2017-latest-ubuntu
```

To get the latest SQL Server 2017 version with applied CU

__NOTE:__ To see which SQL Server versions, please go [here](https://hub.docker.com/_/microsoft-mssql-server) and select your "tag"

## How to add new databases to the image

It´s as easy as modifying the [Dockerfile](./Dockerfile), and adding the new backups you want to restore, and modifying the [setup.sql](./setup.sql) file with the RESTORE command.

## How to change the sa password

The password for the "sa" account is specified at the [docker-compose.yml](./docker-compose.yml) file.

# How it works?

Well, its a little tricky but when you find how it works, its very simple and stable:

[Dockerfile](/Dockerfile) makes 3 mayor steps

## Installing curl

This is the first thing we need to do, since we are going to download directly to the image, the databases we want

```docker
RUN apt-get update && apt-get install -y  \
	curl \
	apt-transport-https
```

**IMPORTANT:** Please have in mind that starting with SQL Server 2019, mssql server containers are non-root. We need to change to root for executing specific tasks like this one

## Downloading databases

Once we have the curl installed, we are now ready to download the databases, and that´s what you found here:

```docker
##############################################################
# DATABASES SECTION
#    1) Add here the databases you want to have in your image
#    2) Edit setup.sql and include the RESTORE commands
#

# Adventureworks databases
#
RUN curl -L -o AdventureWorks2017.bak https://github.com/Microsoft/sql-server-samples/releases/download/adventureworks/AdventureWorks2017.bak
RUN curl -L -o AdventureWorks2016.bak https://github.com/Microsoft/sql-server-samples/releases/download/adventureworks/AdventureWorks2016.bak
...
```

_NOTE: Here you can add-remove the databases you want_

## Restoring databases

This is the tricky part since involves 2 scripts and the final command to keep alive the image

### Entrypoint

```docker
COPY setup.* ./
COPY entrypoint.sh ./

RUN chmod +x setup.sh
RUN chmod +x entrypoint.sh

# This entrypoint start sql server, restores data and waits infinitely
ENTRYPOINT ["./entrypoint.sh"]
```

### Avoid container to stop after deploy

To avoid the container to stop after first run, you need to ensure that is waiting for something. the best solution is to add a sleep infinity...as simple as it sounds :)

```docker
CMD ["sleep infinity"]
```