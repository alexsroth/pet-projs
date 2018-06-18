# installing onprem
Server set up not included in this install process. For this install we will use the following server:

`centos@onprem-install.westeurope.cloudapp.azure.com`
## prerequisites
For this process, you will need to have 4 links, provided by CARTO, available to you.
### links
builder [installer_url]
```
https://s3.amazonaws.com/com.carto.onpremises.release/2.2.0/carto-builder-2.2.0-rhel-6-x86_64.tar.gz?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=AKIAJMLX2VFGMQPJKPAQ%2F20180613%2Fus-east-1%2Fs3%2Faws4_request&X-Amz-Date=20180613T102257Z&X-Amz-Expires=604800&X-Amz-SignedHeaders=host&X-Amz-Signature=c936c573cf30a9f51475229d40be12de5ccbb09edc18cb1429b3621618f31341
```
#### LDS [installer_url]
```
https://s3.amazonaws.com/com.carto.onpremises.release/1.1.0/carto-dataservices-1.1.0-rhel-6-x86_64.tar.gz?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=AKIAJMLX2VFGMQPJKPAQ%2F20180613%2Fus-east-1%2Fs3%2Faws4_request&X-Amz-Date=20180613T102308Z&X-Amz-Expires=604800&X-Amz-SignedHeaders=host&X-Amz-Signature=077bf044dcc329a39261d6b22a3e819d84aa9a5d9c543f3aa34872352224e3d2
```
#### data dumps:
##### [geocoder_url]
```
https://s3.amazonaws.com/data.cartodb.net/geocoding/dumps/geocoder-0.0.1.tar.gz?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=AKIAISZDD2JNL6KXA3QA%2F20180613%2Fus-east-1%2Fs3%2Faws4_request&X-Amz-Date=20180613T102927Z&X-Amz-Expires=604800&X-Amz-SignedHeaders=host&X-Amz-Signature=1a93afc3a0e31807d67b0ce1f0abdfe070af76d1a8c1e554689e4ec7ada32b73
```
##### [observatory_url]
```
https://cartodb-observatory-data.s3.amazonaws.com/do-release-2017_11_29_dac4af93bd/obs.dump?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=AKIAIG4M35F3TZJL2EBA%2F20180613%2Fus-east-1%2Fs3%2Faws4_request&X-Amz-Date=20180613T102927Z&X-Amz-Expires=604800&X-Amz-SignedHeaders=host&X-Amz-Signature=172162caeb651c8678072aa0a755f76f97e4da0c2a4f1e144dfc0ba2e1a5f59e
```
## getting started
First, you will want to access your server, if you are not already. Our server is called `onprem-install`

```console
$ ssh onprem-install
```
next we will use cURL to download the builder installer
```console
$ curl -o cartobuilder.tar.gz "https://s3.amazonaws.com/com.carto.onpremises.release/2.2.0/carto-builder-2.2.0-rhel-6-x86_64.tar.gz?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=AKIAJMLX2VFGMQPJKPAQ%2F20180613%2Fus-east-1%2Fs3%2Faws4_request&X-Amz-Date=20180613T102257Z&X-Amz-Expires=604800&X-Amz-SignedHeaders=host&X-Amz-Signature=c936c573cf30a9f51475229d40be12de5ccbb09edc18cb1429b3621618f31341"
```
once done, the compressed folder will be in the directory. you can check using `ls`
```console
$ ls
cartobuilder.tar.gz
```
we will need to extract the compressed installer:
```console
$ tar xfvz cartobuilder.tar.gz
```
the xfvz options of tar stands for `eXtract File Verbose Zipped`. You will want to change your directory to the newly extracted folder:
```console
$ ls
carto-builder-2.2.0-rhel-6-x86_64  cartobuilder.tar.gz
$ cd carto-builder-2.2.0-rhel-6-x86_64/
$ ls
carto  carto-builder.config  carto-setup.sh  db_dumps  lib  packages  product  setup  templates  tools
```
This directory is where you will get the opportunity to alter the `carto-builder.config` file. Let's enter it now to see what is in it:

```console
$ cat carto-builder.config
CARTO_VERSION="2.2.0"
CARTO_CLIENT_BUILD="carto"
CARTO_USER="carto"
CARTO_PASSWORD="location"
CARTO_EMAIL="carto@example.com"
CARTO_ORG="organization"
CARTO_DOMAIN="onprem-install.westeurope.cloudapp.azure.com"

# NOTE: Disk quota is in megabytes
CARTO_DISK_QUOTA="1048576"
CARTO_ORG_SEATS="100"
CARTO_ORG_VIEWER_SEATS="1000"
# NOTE: File size is in bytes
CARTO_MAX_IMPORT_FILE_SIZE="1057286400"
CARTO_MAX_IMPORT_TABLE_ROW_COUNT="1000000"

# We will define data and snapshot paths from these two
CARTO_DATA_ROOT_PATH="/data"
CARTO_SNAPSHOT_ROOT_PATH="/data"
# This directory will contain the files shared between builder and resque
CARTO_SHARED_DIR_PATH="/data/shared"
CARTO_TMP_DIR_PATH="/data/tmp"

CARTO_LOG_PATH="/data/log"
CARTO_BACKUP_PATH="/backup"


#=====================================
## Builder
#=====================================
# Allowed IP Configuration (CARTO_XXXX_ADDRESS):        | "127.0.0.1" | "192.168.1.64" | "sql-api.carto.com" |
# Allowed BIND IP Configuration (CARTO_XXXX_BIND): | "127.0.0.1" | "192.168.1.64" | "0.0.0.0"           |
# NOTE: Make sure you configure properly both ADDRESS and BIND vars to have a consistent configuration.

# [monit]
CARTO_MONIT=true

# [redis]
CARTO_REDIS_ADDRESS="127.0.0.1"
CARTO_REDIS_BIND="127.0.0.1"
CARTO_REDIS_PORT="6379"
# [builder postgresql]
CARTO_BUILDER_POSTGRES_ADDRESS="127.0.0.1"
CARTO_BUILDER_POSTGRES_BIND="127.0.0.1"
CARTO_BUILDER_POSTGRES_PORT="5432"
# [nginx]
CARTO_NGINX_ADDRESS="127.0.0.1"
CARTO_NGINX_BIND="0.0.0.0"
CARTO_NGINX_HTTP_PORT="80"
CARTO_NGINX_HTTPS_PORT="443"
# [sql-api]
CARTO_SQL_API_ADDRESS="127.0.0.1"
CARTO_SQL_API_BIND="127.0.0.1"
CARTO_SQL_API_PORT="8080"
CARTO_SQL_API_ENTITIES_ACCESS="false"
# [varnish]
CARTO_VARNISH_ADDRESS="127.0.0.1"
CARTO_VARNISH_BIND="127.0.0.1"
CARTO_VARNISH_PORT="6081"
CARTO_VARNISH_ADMIN_PORT="6082"
# [maps-api]
CARTO_MAPS_API_ADDRESS="127.0.0.1"
CARTO_MAPS_API_BIND="127.0.0.1"
CARTO_MAPS_API_PORT="8181"
# [builder]
CARTO_UNICORN_ADDRESS="127.0.0.1"
CARTO_UNICORN_BIND="127.0.0.1"
CARTO_UNICORN_PORT="8888"
# [builder - resque]
# [mailer]
MAIL_FROM=""
MAIL_ADDRESS=""
MAIL_PORT=""
MAIL_USERNAME=""
MAIL_USERNAME_PASSWORD=""
MAIL_AUTHENTICATION=""
MAIL_ENABLE_STARTTLS=false

# Connectors
CARTO_ARCGIS_CONNECTOR_ENABLED=false

#=====================================
## Dataservices
#=====================================
# Allowed IP Configuration (CARTO_XXXX_ADDRESS):        | "127.0.0.1" | "192.168.1.64" | "sql-api.carto.com" |

# CARTO_DATASERVICES_ADDRESS="127.0.0.1"
CARTO_DATASERVICES_PORT="5433"
CARTO_DATASERVICES_USER="geocoder_api" ## postgres user [ dataservices client -> dataservices server]
CARTO_DATASERVICES_DB="db_dataservices"

# Builder analysis
CARTO_DS_ANALYSIS_GEOCODER=false
CARTO_DS_ANALYSIS_HIRES=false
CARTO_DS_ANALYSIS_ISOLINES=false
CARTO_DS_ANALYSIS_ROUTING=false
CARTO_DS_ANALYSIS_DOBSERVATORY=false

# Quotas
CARTO_DS_GEOCODER_QUOTA=400000
CARTO_DS_ROUTING_QUOTA=20000
CARTO_DS_ISOLINES_QUOTA=20000
CARTO_DS_DO_SNAPSHOT_QUOTA=400000
CARTO_DS_DO_GENERAL_QUOTA=999999999
```

The notable things to adjust here are:
```console
CARTO_USER=""
CARTO_PASSWORD=""
CARTO_EMAIL=""
CARTO_ORG=""
CARTO_DOMAIN=""(set this to the server address)

# NOTE: Disk quota is in megabytes
CARTO_DISK_QUOTA=""
CARTO_ORG_SEATS=""
CARTO_ORG_VIEWER_SEATS=""
# NOTE: File size is in bytes
CARTO_MAX_IMPORT_FILE_SIZE=""
CARTO_MAX_IMPORT_TABLE_ROW_COUNT=""
```

to enter and edit any of this information, you can use nano:

```console
$ nano carto-builder.config
```

switch to the root, and install gcc if you don't already have it:

```console
$ sudo su
# yum install gcc
```

run the installer:

```console
# ./carto-setup.sh builder
Welcome to CARTO on-premises 2.2.0 installer.

The following account will be created to manage the CARTO Builder:

 - Domain: onprem-install.westeurope.cloudapp.azure.com
 - Organization: organization
 - Admin: carto
 - Password: (not shown, it's available on ./carto-builder.config)

The CARTO Builder will use the following paths:

 - Data: /data
 - Logs: /data/log

NOTE: edit ./carto-builder.config file if you want to change something.

Do you want to continue?' (Y/n): Y
```

This will start the install process.

**current known error:** in product/, there is a file named `builder.NEWS.md` that needs to be named `builder.NEWS` instead.

**fix:**
```console
# cp product/builder.NEWS.md product/builder.NEWS
```
