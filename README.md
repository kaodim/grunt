# Description

**grunt** is an application to perform grunt work that can be delegated to a machine.
Its consist of
1. a Sinatra app, which runs on a **thin** server, with actions exposed as api and
2. a collection of rake tasks that can be invoked in case you do not want to run the app server

See `main.rb` and `Rakefile` for the usage of the action api and rake task.

# Dependency

1. Elasticsearch >= 7.0

# Tutorial

1. To start the server, run `rake thin:start`
2. To restart the server, run `rake thin:restart`

# Using Docker
## Run Rake Task for Log Parsing
Log parser sends logs to Elasticsearch for analytics. There are three ways to run log parser rake task.
1. Send logs to an Elasticsearch cluster running on localhost.
2. Send logs to a single node Elasticsearch cluster running on Docker.
3. Send logs to a three node Elasticsearch cluster running on Docker.

### Local ES
- Start:
`docker-compose up -d`
- Exec to Container:
`docker-compose exec grunt sh`
- Stop:
`docker-compose down`
### Single Node ES
- Start:
`docker-compose -f docker-compose.es.yml up -d`
- Exec to Container:
`docker-compose -f docker-compose.es.yml exec grunt sh`
- Stop:
`docker-compose -f docker-compose.es.yml down`
### Three Node ES
- Start:
`docker-compose -f docker-compose.escluster.yml up -d`
- Exec to Container:
`docker-compose -f docker-compose.escluster.yml exec grunt sh`
- Stop:
`docker-compose -f docker-compose.escluster.yml down`

## Settings
In `config/settings.yml` use the following settings for connecting to Elasticsearch when running from Docker.
### Using Local ES
```
elasticsearch:
  host: host.docker.internal
  port: 9200
```

### Using Docker ES
```
elasticsearch:
  host: es
  port: 9200
```

## Download logs from S3
- For AWS CLI, if not using the default profile, set your profile using `export AWS_PROFILE=`.
- Run the following script to download the log files.
- Fill in the values for `BUCKET`, `LOG_TYPE` and `LOG_FILE_PREFIX`.
- Log files will be stored in a folder with prefix `s3logs-`.

```
BUCKET=
LOG_TYPE=
LOG_FILE_PREFIX=
DESTINATION_PATH=./s3logs-$BUCKET/
files=$(aws s3 ls s3://$BUCKET/$LOG_TYPE/$LOG_FILE_PREFIX | awk '{print $4}')
for f in $files; do aws s3 cp s3://$BUCKET/$LOG_TYPE/$f $DESTINATION_PATH && gunzip $DESTINATION_PATH$f & done;
```

## Command for Rake Task
- Make sure you are in the repository and log folder `s3logs-` is in the same directory.
- This directory will be mounted inside container in the folder `/app` when run from Docker.

Command is as follows:
- To run for single file
```
bundle exec rake es:import_admin_log['/absolute/path/to/logfile.log']
```

- To run for multiple files
```
bundle exec rake es:import_admin_log['/absolute/path/to/logfile-1.log /absolute/path/to/logfile-2.log']
```

- To run for all files in `s3logs-*` folder
```
bundle exec rake es:import_admin_log["$(ls $PWD/s3logs-*/*.log | tr '\n' ' ')"]
```

- If the mappings doesnt really matter to you, you can use the default importer
```
bundle exec rake es:import_log['index-name','/absolute/path/to/logfile-1.log /absolute/path/to/logfile-2.log']
```

- To delete any index, just do
```
bundle exec rake es:delete_index['index-name']
```

## View logs using Kibana
Kibana will be running on [localhost:5601](http://localhost:5601)
