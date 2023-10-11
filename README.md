# Dockerizing WordPress and Database Optimization

This project involves Dockerizing a WordPress application and optimizing the database for improved performance. Follow these instructions to set up the project.

## Table of Contents
1. [Dockerization](#dockerization)
    - [Dockerfile](#dockerfile)
    - [Docker Compose](#docker-compose)
2. [Database Optimization](#database-optimization)
3. [Additional Notes](#additional-notes)

## Dockerization

### Dockerfile

I start by creating a Dockerfile for WordPress, using the latest image of Wordpress.

```Dockerfile
# Use the official WordPress image as the base image
FROM wordpress:latest

# Then defined the environmental variables for database configuration
ENV WORDPRESS_DB_HOST=db
ENV WORDPRESS_DB_USER=root
ENV WORDPRESS_DB_PASSWORD=my-secret-password
ENV WORDPRESS_DB_NAME=my_wordpress_db

## Services Configuration
# No need to start apache service as wordpress image already have the configuration for this function.

```

Then defined services and network in Docker Compose configuration:

- **wordpress**: In the `wordpress` service, specified the build context, port mapping, common network to configure the connection to the database. Also used "depends_on" keyword, to make sure that database container is started before wordpress container.

- **db**: Then put `mysql` service, along with the environment variable, volume attachment and common network.

- **network**: Then create a network so that the wordpress image can use database name to point to it in place of IP address, as on restarting the container the IP address of the container can be changed but if both the containers are in same network then wordpress can be connected using the name of the container.

```

version: '3.8'
services:
  wordpress:
    build:
      context: .
      dockerfile: Dockerfile
    ports:
      - "80:80"
    depends_on:
      - db
    networks:
      - mynetwork
  db:  
    image: mysql:latest
    volumes:
      - ./mysql_data:/var/lib/mysql
    environment:
      MYSQL_ROOT_PASSWORD: my-secret-password
      MYSQL_DATABASE: my_wordpress_db
      MYSQL_QUERY_CACHE_TYPE: 1
      MYSQL_QUERY_CACHE_SIZE: 64M
    networks:
      - mynetwork
networks:
  mynetwork:
    driver: bridge


```


## Database Optimization

I implemented various strategies to optimize the database for improved performance. Here I have listed some of the techniques used in optimisation of database.

- **Indexing**: Indexing in a database is the process of creating data structures (indexes) that significantly speed up data retrieval operations by providing quick access to specific rows in a table based on the indexed column values.

- **Caching**: Caching in a database involves temporarily storing frequently accessed data in memory to reduce the need for repeated disk reads.

-**Query Optimisation**: The process of selecting an efficient execution plan for processing a query is known as query optimization.

- **Database Tuning**: In this, we adjust database server settings like buffer sizes and connection limits. These optimizations aim to enhance the overall performance.

In this setup I have used Caching because caching can be easily scaled by adding more memory, which is often more cost-effective than scaling the database infrastructure. It also decreases the load on the database server, freeing up resources for other operations.
Compared to other optimization techniques like indexing and query optimization, caching provides immediate performance benefits without the need for complex query rewriting or restructuring. However, it is most effective when used in conjunction with other optimization strategies for comprehensive database performance improvements.



## Additional Notes

The above setup have one vulnerability, that is the password of the database is given in open and anyone can see the password, which is not a good use case.
So for rectifying this issue, we can use one functionality of docker swarm that is "secret" so we can make secret using the folowing command but before this we have to make the docker swarm active.

```bash
docker swarm init
```

```bash
echo -n "mysql_root_password" | docker secret create my-secret-password -
```

This is will make the password secure and we can use it in docker-compose file by using **MY_SQL_PASSWORD_FILE: /run/secret/my-secret-password and before this we have to use one more block that is secret and give it the name of the secret.

Also we can see our secret using:

```bash
docker secret ls
```