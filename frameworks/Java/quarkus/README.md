# Quarkus Benchmarking Test

This is the Quarkus portion of a [benchmarking test suite](../) comparing a variety of web development platforms.

## Implementations

There are currently 2 implementations:

- Quarkus using RESTEasy Reactive and Hibernate ORM (classic Hibernate for DB operations, while handling web via the reactive stack)
- Quarkus using RESTEasy Reactive and Hibernate Reactive (fully reactive stack)

## Testing

    ./tfb --mode verify --test quarkus quarkus-hibernate-reactive

## Versions

* [Java OpenJDK 17](http://openjdk.java.net/)
* [Quarkus 3.1.0.CR1](https://quarkus.io)

## Test URLs

### Plaintext Test

    http://localhost:8080/plaintext

### JSON Encoding Test

    http://localhost:8080/json

### Database Query Test

    http://localhost:8080/db

### Database Queries Test

    http://localhost:8080/queries?queries=5

### Database Update Test

    http://localhost:8080/updates?queries=5

### Template rendering Test

    http://localhost:8080/fortunes

## Local development

Start a local postgreSQL instance:

    podman run --rm=true --network host --name quarkus_techempower_test -e POSTGRES_USER=benchmarkdbuser -e POSTGRES_PASSWORD=benchmarkdbpass -e POSTGRES_DB=hello_world -p 5432:5432 docker.io/library/postgres:15.2

Reconfigure the Quarkus application to create the DB and to connect to the local DB instance;
edit the `application.properties` resource, so to point to the database on localhost.
N.B. edit the `application.properties` resource of the module you intend to test, possibly 
both of them.

    quarkus.hibernate-orm.database.generation=create
    quarkus.datasource.reactive.url=postgresql://localhost:5432/hello_world

Build all modules

    mvn clean package

Now to start it and prepare for benchmarking;
get into the folder of the module to test.

To test the Quarkus version using Hibernate Reactive:

    cd resteasy-reactive-hibernate-reactive/target/quarkus-app/

*or*, to test with "classic" Hibernate ORM (non-blocking):

    cd resteasy-reactive-hibernate/target/quarkus-app/

Start the app, *from the specific folder*:

    ../../../run_quarkus.sh

Trigger the DB data population before running validations or benchmarks:

    http localhost:8080/createData

Now the app is ready to be test and/or benchmarked.

Generate load on the application to test / profile it.
I suggest to use `https://github.com/giltene/wrk2`.

Example run, assuming you have built wrk2 in `~/sources/wrk2` :

     ~/sources/wrk2/wrk -c 100 -d 60 -R 400 http://localhost:8080/db

The URL `http://localhost:8080/db` represents one specific benchmark; there are several more to try
but you will likely want to focus on them one at a time.
The other endpoints are listed above.
