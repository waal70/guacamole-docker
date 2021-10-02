# guacamole-docker
Repository for docker-compose scripts

### Preparation
Change:
```env```
to
```.env```
and populate/change the relevant fields. It will contain passwords, make them as exotic as you like.

Run:

```sudo docker-compose up init-guac-db```

to initialize the db script. This will result in the creation of a ```initdb.sql```.

### Creation of containers and running them
Run:

```sudo docker-compose up -d```

### More explanation
Walkthrough of some highlighted settings in docker-compose.yml:
```
  guacd:
    image: guacamole/guacd:latest
```
    

This is the utility container for guacamole. For more info, check [the guacamole documentation](https://guacamole.apache.org/doc/gug/guacamole-architecture.html)
```
  init-guac-db:
    image: guacamole/guacamole:latest
    command: ["/bin/sh", "-c", "test -e /init/initdb.sql && echo 'init file already exists' || /opt/guacamole/bin/initdb.sh --mysql > /init/initdb.sql" ]
    volumes:
      - ${BASE:-~}guac:/init
```
Depending on your choice of database (in this case, MySQL), the database scheme needs to be populated. This will generate the script which we will
later use. Note we are creating a initdb.sql in the init folder, which we later will map to its rightful place.
```
  db:
    image: 'mariadb:latest'
    volumes:
      - ./data/mysql:/var/lib/mysql
      - ${BASE:-~}guac:/config
      - ${BASE:-~}guac/initdb.sql:/docker-entrypoint-initdb.d/init.sql
    depends_on:
      - init-guac-db
```
Here, we link the initdb.sql script to the docker-reserved init.sql. Upon the very first execution of this container, this script will
be automatically ran. This will complete our initialization of the database.
```
  guac:
      VIRTUAL_HOST:
    depends_on:
      - db
      - guacd
```
Because we are putting an nginx in front of guacamole, the VIRTUAL HOST need not be specified.
```
networks:
  default:
    external: true
    name: nginx_default
```
Here we are pointing to the network that was created in the docker-compose for nginx, so that the containers may interact among themselves.
What could be a TODO is to create one huge stack, that will also include the nginx install.
