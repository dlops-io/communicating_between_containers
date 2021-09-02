# Communication between Containers

This tutorial is assuming you have gone through the previous [Introduction to Development using Containers](https://github.com/dlops-io/intro_to_containers). As a refreshed the architecture we followed was:

![Docker with Persistent Store](https://storage.googleapis.com/public_colab_images/docker/docker_with_persistent_storage.png)

## Prerequisites
* Have Docker installed
* Cloned this repository to your local machine with a terminal up and running
* Check that your Docker is running with the following command

`docker run hello-world`

### Install Docker 
Install `Docker Desktop`

#### Ensure Docker Memory
- To make sure we can run multiple container go to Docker>Preferences>Resources and in "Memory" make sure you have selected > 4GB

### Install VSCode  
Follow the [instructions](https://code.visualstudio.com/download) for your operating system.  
If you already have a preferred text editor, skip this step.  

### Clone the github repository
- Clone or download from [here](https://github.com/dlops-io/communicating_between_containers)


## Make sure we do not have any running containers and clear up an unused images
* Run `docker container ls`
* Stop any container that is running
* Run `docker system prune`
* Run `docker image ls`

## Frontend App Container
### Starting the Container
Type the command 
-  `cd frontend-app`
- Run `sh docker-shell.sh` or `docker-shell.bat` for windows

To install a new node package use `npm install <package name>` from the docker shell

To run development web server run `http-server` from the docker shell

Test the API service by going to `http://localhost:8080/`

## Download Images Container
### Starting the Container
First let is create a `persistent-folder` that will be used to save the downloaded images and also mounted to multiple containers so we can share content between containers.
- Create a folder `persistent-folder` at the same level as our container folders. So you should have your directories like this:
```
├── api-service
├── database-server
├── download-images
├── frontend-app
└── persistent-folder
```

Type the command 
-  `cd download-images`
- Run `sh docker-shell.sh` or `docker-shell.bat` for windows

in the `download-images` directory to startup (build and run) the docker container. The container contains a Linux kernel and will install all dependencies for the image downloader. Running this command for the first time will take around 90 seconds (depending on the system you're using). This command automatically builds and runs the Docker container, where you should automatically be entered into the Linux kernel in the **app** Python virtual environment.

If you `exit` the Docker container, and run the command `docker images | grep download-images`, you'll see some nice information about the image name, tag, image ID, creation date, and size

### Running Downloader
Now that you are inside the Docker container, run the example command

`python -m cli --nums 10 --search "oyster mushrooms" "crimini mushrooms" "amanita mushrooms" --opp "search"`

`python -m cli --opp "verify"`

where 
- `nums` is the number of images to download
- `search` contains the search terms
- `opp` is the operation whether to `search` or `verify` the images

Images will be downloaded in a sub-directory named *persistent-folder/dataset*

## API Service Container
### Starting the Container
Type the command 
-  `cd api-service`
- Run `sh docker-shell.sh` or `docker-shell.bat` for windows

To install a new python package use `pipenv install requests` from the docker shell

To run development api service run `uvicorn_server` from the docker shell

Test the API service by going to `http://localhost:9500/`

---
---
---

In the this section we will be setting up containers so that they talk to each other. The following architecture shows what we are going to do today:

![Docker with Persistent Store](https://storage.googleapis.com/public_colab_images/docker/docker_with_network.png)

## Database Server Container

Here we will setup a database server and a database client. We will use PostgreSQL as our database server and [dbmate](https://github.com/amacneil/dbmate) as our database client and migration tool. 

### Starting the Container
Type the command 
-  `cd database-server`
- Run `sh docker-shell.sh` or `docker-shell.bat` for windows
- Can exit the docker shell without shutting down by typing `ctrl+d`
- Can reconnect to docker shell by typing...
- Check migration status: `dbmate status`
- To shut down docker container, type `ctrl+c`

#### Connecting to the database
* Run `psql postgres://itc:awesome@itcdb-server:5436/itcdb` in the docker shell
* Format to connect to postgres: postgres://<user is>:<password>@i<database server>:<port>/<database name>

* Since we do not have any tables created we can check on some system tables, Run this select query `select table_catalog,table_schema,table_name,table_type from information_schema.tables limit 10;`

* Next let us create a table in the database
* Exit from the DB prompt so we are back in the dbmate prompt
* Run `dbmate new image_metadata`, this will create a migration file

#### Dbmate Commands Reference

```sh
dbmate --help    # print usage help
dbmate new       # generate a new migration file
dbmate up        # create the database (if it does not already exist) and run any pending migrations
dbmate create    # create the database
dbmate drop      # drop the database
dbmate migrate   # run any pending migrations
dbmate rollback  # roll back the most recent migration
dbmate down      # alias for rollback
dbmate status    # show the status of all migrations (supports --exit-code and --quiet)
dbmate dump      # write the database schema.sql file
dbmate wait      # wait for the database server to become available
```

* In the migration file we just created added the following table creation scripts:
```
CREATE TABLE image_metadata (
    id BIGSERIAL PRIMARY KEY,
    label TEXT NOT NULL,
    path TEXT NOT NULL
);

DROP TABLE IF EXISTS image_metadata;
```

* Run the DB migration so we create the table in the database. Run `dbmate up` (see db/migrations for what tables are created).
* Run `psql postgres://itc:awesome@itcdb-server:5436/itcdb` in the docker shell and run the query `select * from image_metadata;` The table should exist but no data.
* Exit from the DB prompt so we are back in the dbmate prompt
* Run `dbmate rollback` to test our removing tables from the database
* Run `dbmate up` again so we have the new table for storing image meta data in our database


