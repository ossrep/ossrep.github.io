# ossrep

Tools required: 
* podman
* podman-compose

## Local Development

The local development starts a Keycloak instance. 

To start, use the following command. 
```shell
podman-compose -f local-dev/compose.yaml up -d 
```

To stop, use the following command. 
```shell
podman-compose -f local-dev/compose.yaml down
```

Cleanup
```shell
podman volume rm local-dev_postgres_data
```

### Keycloak

Keycloak is available at http://localhost:8001, u/p: admin  

The following users are setup for testing:   

* amy.admin / Pass123!  