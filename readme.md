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

## Configuration

### Getting started
Copy the env_template file and rename it to .env.  This will be the secret env file that should never be committed (it is in the .gitignore by default).

Each of the ports should be set to localhost, unless there is a requirement for external access.  If this is the case, ensure the ports are not open to public in the firewall or via AWS security groups.

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

