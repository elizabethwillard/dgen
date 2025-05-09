# dGen Docker Usage Guide

By default, the dGen container uses the [Delaware residential agent file and SQL database](https://oedi-data-lake.s3.amazonaws.com/dgen/de_final_db/dgen_db.sql).

See the [Customizing the Model](#customizing-the-model) section below to learn how to use a different agent and SQL database and update the model scenario.

### Mac, Windows, and Linux quick start

This quickstart uses docker-compose to run dGen. The default path to store dGen data files, excel configurations, and results is ~/dgen_data/. This path is shared with your running containers. You can change this path but you will need to edit the `docker-compose.yml` file to reflect the data directory of your choice.

Prerequisites assume you are using a Mac or Windows computer, and that you have already installed [Docker Desktop](https://docs.docker.com/desktop/setup/install/mac-install/)

##### Create the ```dgen_data``` directory in your root directory
```bash
$ mkdir -p ~/dgen_data/ 
$ chmod 755 ~/dgen_data/ # If using a Windows operating system, you might get an error when running this command. This command will work if you use GitBash or a similar terminal, which includes common Unix commands such as chmod.
$ ls -l ~/dgen_data/ # This folder is expected to be empty, and after starting dGen you will see data files in this location.
```

##### Startup the dGen containers
``` bash
$ cd dgen/docker/ # Nagivate to the docker folder within the dgen repository
$ docker-compose up --build -d

# After running this command, data will start downloading in the ~/dgen_data/ folder. This could take a while depending on internet connection speeds. Please proceed with the next steps after the data is finished downloading.

# Check that you are ready to proceed by going to Docker Desktop, clicking on the postgis_1 container, and clicking on "Logs". There should be a message that says "database system is ready to accept connections". 

[+] Running 2/2
 ✔ Container dgen_1     Started 0.1s
 ✔ Container postgis_1  Started 0.0s
```

##### Connect to the running containers
``` bash
$ docker attach dgen_1 # Attach to dGen container
$ docker attach $(sudo docker ps --filter "name=dgen" --format "{{.ID}}") # If dgen_1 is not found
(dg3n) dgen@cc6e2e5f70b5:/opt/dgen_os/python$ python dgen_model.py # Run scenario
(dg3n) dgen@cc6e2e5f70b5:/opt/dgen_os/python$ exit # to exit
$ docker-compose up -d # If you exit, you have to re-up the container if you want to re-attach
```
### Viewing Results

To connect to the postgres server in pgAdmin, follow the same instructions in the main README file of this repository (Section C. Create Local Server").

On your host, run outputs are written to ~/dgen_data/runs/. Within the dgen container, run outputs are written to /data/runs/.

### Troubleshooting common issues

#### psycopg2.OperationalError: connection to server

Wait 5-10 minutes for the postgres database to finish starting.

#### General errors and issues

Try removing any datasets from your `~/dgen_data` and starting over. Make sure to provide time for the datasets to fully download on the re-attempt.

```bash
$ docker-compose down
$ rm -f ~/dgen_data/*
$ docker system prune -a
$ docker volume prune -f
```

### Disabling auto-start for the dGen virtual environment

By default, logging into the `dgen` container automatically activates the `dg3n` Conda environment. For ease of use, its recommended to leave this the default.

To disable this behavior, edit the `docker-compose.yml` file in this directory and set the following environment variable:

```yaml
services:
  dgen:
    environment:
      DGEN_DISABLE_AUTO_START: 1
```

### Customizing the Model

By default, the dGen container uses the [Delaware residential agent file and SQL database](https://oedi-data-lake.s3.amazonaws.com/dgen/de_final_db/dgen_db.sql).

You can find additional agent files and their matching SQL database files for the contiguous United states using the links below:
- [dGen Agent Files and SQL Databases on OpenEI](https://data.openei.org/submissions/1931)
- [dGen Dataset S3 Viewer](https://data.openei.org/s3_viewer?bucket=oedi-data-lake&prefix=dgen%2F)

You can customize the dataset used by overriding the DGEN_DATAFILE_URL and DGEN_AGENTFILE_URL variables in `docker-compose.yml`, and then editing `~/dgen_data/input_sheet_final.xlsm` using Excel.

The information below will describe the process of updating the `docker-compose.yml` file to use the [Colorado residential agent file and SQL database file](https://data.openei.org/s3_viewer?bucket=oedi-data-lake&prefix=dgen%2Fco_final_db%2F) as an example.

First, copy the link addresses from the download links of the desired agent file and the relevant SQL database file. Then, update `docker-compose.yml` to download and use the files of choice for the model runs by inserting the copied agent file link next to `DGEN_DATAFILE_URL:` and the copied SQL database file link next to `DGEN_AGENTFILE_URL:`. If this is not the first time you are setting up the Docker Container, set the `DGEN_FORCE_DELETE_DATABASE` variable to `1` to force remove any previously created database. This will result in dataloss from previous runs; if this is a concern, please make backups before proceeding with the following steps.

```yaml
services:
  postgis:
    environment:
      DGEN_DATAFILE_URL: https://oedi-data-lake.s3.amazonaws.com/dgen/co_final_db/dgen_db.sql
      DGEN_AGENTFILE_URL: https://oedi-data-lake.s3.amazonaws.com/dgen/co_final_db/agent_df_base_res_co_revised.pkl
      DGEN_FORCE_DELETE_DATABASE: 1 # Clear all the data in the database to reload the Colorado dataset, Warning this will remove your existing data.
```

Edit the excel document `~/dgen_data/input_sheet_final.xlsm` using Excel (Enable macros), and set the 'Region to Analyze' to `Colorado` and 'Markets' to `Only Residential`, and then click Save Scenario.

Next, restart your containers using the instructions below.  This will remove all your existing data and setup your Docker Container to use the Colorado files instead.

```bash
$ docker-compose down
[+] Running 3/3
 ✔ Container dgen_1  Removed  9.2s
 ✔ Container postgis_1  Removed  0.1s

 $ docker-compose up -d

# After running this command, data will start re-downloading in the ~/dgen_data/ folder. This could take a while depending on internet connection speeds. Please proceed with the next steps after the data is finished downloading.

# Check that you are ready to proceed by going to Docker Desktop, clicking on the postgis_1 container, and clicking on "Logs". There should be a message that says "database system is ready to accept connections".

 [+] Running 2/2
 ✔ Container dgen_1     Started 0.1s
 ✔ Container postgis_1  Started 0.2s
 ```

After you load the new Colorado dataset, reset the `DGEN_FORCE_DELETE_DATABASE` option to "0" to prevent future accidental data loss.

```yaml
services:
  postgis:
    environment:
      DGEN_FORCE_DELETE_DATABASE: 0
```

```bash
$ docker attach dgen_1
(dg3n) dgen@cc6e2e5f70b5:/opt/dgen_os/python$ ls -lh /data/dgen_db.sql
-rw-r--r-- 1 dgen dgen 705M Jan 29  2025 /data/dgen_db.sql
(dg3n) dgen@cc6e2e5f70b5:/opt/dgen_os/python$ python dgen_model.py # Run scenario
(dg3n) dgen@cc6e2e5f70b5:/opt/dgen_os/python$ exit # to exit
```

```bash
$ ls -l ~/dgen_data/runs/
drwxr-xr-x  5 username  group  160 Mar 12 15:14 results_20250312_211405
```

```bash
(dg3n) dgen@1b1fce587396:/opt/dgen_os/python$ ls -l /data/runs/
drwxr-xr-x 5 dgen dgen 160 Mar 12 21:14 results_20250312_211405
```

### Stop running containers
```bash
$ docker ps -a
```

```bash
CONTAINER ID   IMAGE                           COMMAND                  CREATED          STATUS          PORTS                    NAMES
259c30e6b518   docker-postgis                  "docker-entrypoint.s…"   12 minutes ago   Up 12 minutes   0.0.0.0:5432->5432/tcp   postgis_1
a775696276eb   docker-dgen                     "bash --login"           12 minutes ago   Up 4 seconds                             dgen_1
```

```bash
$ docker-compose down
```

```bash
[+] Running 3/3
 ✔ Container dgen_1  Removed  10.1s
 ✔ Container postgis_1  Removed  0.1s
```

### Warning: This will remove old running containers and data volumes.  This may be required if you need space.

```bash
$ docker system prune -a
$ docker volume prune -f
```
