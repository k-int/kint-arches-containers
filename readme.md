# K-Int docker containers for Arches dependencies.

## Containers
- PostgreSQL
- Elasticsearch (two nodes)
- Redis
- Geoserver

Works in progress:
- pg_featureserv
- pg_tileserv
- varnish

Future containers to be added:
- Nginx / Apache2
- Celery workers / supervisor
- Flower (UI for celery)

Previously removed:
- CouchDB (removed in 7.0.0)

## Instructions

### Getting started
Copy the env_template file and rename it to .env.  This will be the secret env file that should never be committed (it is in the .gitignore by default).

Each of the ports should be set to localhost, unless there is a requirement for external access.  If this is the case, ensure the ports are not open to public in the firewall or via AWS security groups.

Two compose files exist in this repo - one for development (docker-compose-dev.yml) which runs on http, and one for production (docker-compose-prod.yml) which uses your SSL certificates for a secure https connection.

To begin, run `docker compose -f [docker-compose-dev.yml / docker-compose-prod.ym] up -d`.
The -f flag specifies the compose file.
The -d flag spins up all containers in detatched mode (in the background). 

### Using variables in Arches
To use these variables in the Arches settings, one option is to copy and paste them, but this is problematic for a number of reasons.  It means both files will include the same passwords, in plain text, and the Arches settings.py will need updating if any change.

Using the `python-dotenv` pip package, we can point to a .env file and pull the variables directly.

Install the package using `pip install python-dotenv`.

If a settings_local.py file doesn't already exist in your Arches project, create one.  This can be used as a secondary settings file for secret configurations/password, as it is always in the .gitignore.  

Ensure your settings_local.py has the following at the top of the file.
```
try:
    from .{ARCHES_PROJECT_NAME}.settings import *
except ImportError:
    pass
``` 

Then, add the following lines to the settings_local.py file, and supply the path to this .env file.
```
from dotenv import load_dotenv
from pathlib import Path

dotenv_path = Path('path/to/.env')
load_dotenv(dotenv_path=dotenv_path)
```

Now the .env file has been loaded into the django settings, it can be used in configuration e.g.
```
ELASTICSEARCH_HTTP_PORT=os.getenv('ELASTIC_PORT')
```

## Dependency configuration

### PostgreSQL
Postgres has the option to be tuned to the specs of the server.  If this is not required, leave the default .env values.
Uncomment the `log_statement=all` line for debug logging.

### Elasticsearch
In production, elasticsearch should be run with xpack security as true, however, this requires SSL certificates.
If xpack security is false, ensure the Arches `ELASTICSEARCH_HOSTS` scheme is set to http and `ELASTICSEARCH_CONNECTION_OPTIONS` verify_certs is false.
Enter the env elasticsearch user password into `ELASTICSEARCH_CONNECTION_OPTIONS` basic_auth. In future, allow settings_local.py to use .env information.

## Troubleshooting

### Elasticsearch containers
For running multiple ES nodes, the error `max virtual memory areas vm.max_map_count [65530] is too low, increase to at least [262144]` may occur.  
To fix do the following (from the source https://stackoverflow.com/questions/51445846/elasticsearch-max-virtual-memory-areas-vm-max-map-count-65530-is-too-low-inc)
- Open `/etc/sysctl.conf` as root
- Set `vm.max_map_count = 262144`
- Reboot or run `sysctl --system` as root

