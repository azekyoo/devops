# DevOps

#### Why should we run the container with a flag -e to give the environment variables?

Using the `-e` flag lets you pass environment variables to containers, making configuration flexible, portable, and secure without modifying the container's code. It ensures consistency across environments while simplifying automation and scaling.

#### Why do we need a volume to be attached to our postgres container?

A volume is needed for a PostgreSQL container to persist data, so that the database remains intact across container restarts or removals. Without a volume, all data would be lost when the container stops.

#### Database documentation

Essential commands: 

```docker build -t <YOUR_USERNAME>/database```

```docker network create app-network```

```docker run --name my-db-container --network app-network -v ${PWD}/postgresql/data:/var/lib/postgresql/data -d <YOUR_USERNAME>/database```

```docker run -p "8090:8080" --net=app-network --name=adminer -d adminer```

Dockerfile:
```
FROM postgres:14.1-alpine

ENV POSTGRES_DB=db \
   POSTGRES_USER=usr \
   POSTGRES_PASSWORD=pwd

COPY ./my-init-scripts/ /docker-entrypoint-initdb.d/
```