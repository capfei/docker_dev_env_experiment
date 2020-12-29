# docker_dev_env_experiment

Hello everyone! The purpose of this repo is to give you an easy way to run a full 
development environment for Clearly Defined including:
* [website](https://github.com/clearlydefined/website)
* [service](https://github.com/clearlydefined/service)
* [crawler](https://github.com/clearlydefined/crawler)
* harvest store (mounted as a volume in the service container)
* definitions and curations mongo DB databases
* queues

We do this through [Docker](https://www.docker.com/) and [Docker Compose](https://docs.docker.com/compose/)

## Pre-reqs
To run this environment, you need to install
* [Docker](https://www.docker.com/)
* [Docker Compose](https://docs.docker.com/compose/)

## Getting Started

Start off by cloning this repository.

```bash
$ git clone https://github.com/clearlydefined/docker_dev_env_experiment
```

Change into that directory:

```bash
$ cd docker_dev_env_experiment
```

I prefer to clone my copies of the ClearlyDefined repos into this directory as well

```bash
$ git clone https://github.com/clearlydefined/website
$ git clone https://github.com/clearlydefined/service
$ git clone https://github.com/clearlydefined/crawler
```

Alternately, you can edit the **docker-compose.yml** file to point to where you have those repos cloned on your local system:

**docker-compose.yml**
```bash
version: "3.8"
services:
  web:
    build:
      context: <path-to-website-repo-on-your-system>
      dockerfile: DevDockerfile
    ports: 
      - "3000:3000"
    stdin_open: true
  service:
    build:
      context: <path-to-service-repo-on-your-system>
      dockerfile: DevDockerfile
    ports:
      - "4000:4000"
    env_file: .env
    links:
      - clearlydefined_mongo_db
  crawler:
    context: <path-to-crawler-repo-on-your-system>
    env_file: .env
    ports:
      - "5000:5000"
```

**NOTE**:
While this is still in development, you need to check out the `nell/dev-docker-file` branch for each of the
three repos.

### Setting up environmental variables

This environment handles environmental variables a little differently from the [historical Clearly Defined dev environment instructions](https://docs.clearlydefined.io/contributing-code).

The docker-compose.yml file loads environmental variables from a **.env** file.

To set this up, copy the **sample_env** file in this repo to **.env**

```bash
$ cp sample_env .env
```

And add in appropriate values to the .env file:

(You will need a [GitHub token](https://docs.github.com/en/free-pro-team@latest/github/authenticating-to-github/creating-a-personal-access-token) with minimal permissions)

(You can use the same GitHub token for both CURATION_GITHUB_TOKEN and CRAWLER_GITHUB_TOKEN)


**.env**
```
# Curation GitHub Info
CURATION_GITHUB_REPO="sample-curated-data"
CURATION_GITHUB_BRANCH="master"
CURATION_GITHUB_OWNER="clearlydefined"
CURATION_GITHUB_REPO="curated-data-dev"
CURATION_GITHUB_TOKEN="<Your GitHub Personal Access Token>


# Curation Store Info
CURATION_MONGO_CONNECTION_STRING="mongodb://clearlydefined_mongo_db"
CURATION_MONGO_DB_NAME="clearlydefined"
CURATION_MONGO_COLLECTION_NAME="curations"
CURATION_PROVIDER="github"
CURATION_STORE_PROVIDER="mongo"

# Definition Store Info
DEFINITION_STORE_PROVIDER="mongo"
DEFINITION_MONGO_CONNECTION_STRING="mongodb://clearlydefined_mongo_db"
DEFINITION_MONGO_DB_NAME="clearlydefined"
DEFINITION_MONGO_COLLECTION_NAME="definitions-paged"

# Harvest Store Info
HARVEST_STORE_PROVIDER="file"

# Note - this is mounted as a volume 
# into the container for the 
# clearly defined service
# see docker-compose.yml for more details
FILE_STORE_LOCATION="/tmp/harvested_data"

# Crawler Info
CRAWLER_GITHUB_TOKEN="<Your GitHub Personal Access Token>"
```

Now, from withing your **docker_dev_env_experiment** directory, run:

```bash
$ docker-compose build
$ docker-compose up
```

And head to http://localhost:3000 to see your running website UI along with some seeded data!

You can also query the service API with:

```bash
curl http://localhost:4000
```

## What You're Running

Now, let's go through what your are running, container by container.

* Clearly Defined Website 
* Clearly Defined Service
* Clearly Defined Crawler
* Clearly Defined Mongo DB
* Clearly Defined Mongo DB Seed

### Clearly Defined Website

This is the Clearly Defined React UI. It's what you see when you open your browser and go to http://locahost:3000. It connects to the Clearly Defined Service API.

### Clearly Defined Service

This is the backend of Clearly Defined, you can use it through the Website UI or through 
querying it directly through the command line.

Any Clearly Defined environment needs a place to store raw harvest information. In the case of this development environment, we use the file store for storing harvest information (our production setup uses Azure blob storage).

This Docker setup creates a volume for the harvested data and mounts it in the Service container.

```bash
$ curl http://localhost:4000

{"status":"OK"}

$ curl localhost:4000/definitions/maven/mavencentral/org.flywaydb/flyway-maven-plugin/5.0.7

{"described":{"sourceLocation":{"type":"sourcearchive","provider":"mavencentral","url":"http://central.maven.org/maven2/org/flywaydb/flyway-maven-plugin/5.0.7/flyway-maven-plugin-5.0.7-sources.jar","revision":"5.0.7","namespace":"org.flywaydb","name":"flyway-maven-plugin"},"releaseDate":"2018-01-30","urls":{"registry":"http://central.maven.org/maven2/org/flywaydb/flyway-maven-plugin","version":"http://central.maven.org/maven2/org/flywaydb/flyway-maven-plugin/5.0.7","download":"http://central.maven.org/maven2/org/flywaydb/flyway-maven-plugin/5.0.7/flyway-maven-plugin-5.0.7.jar"},"tools":["clearlydefined/1"],"toolScore":{"total":100,"date":30,"source":70},"score":{"total":100,"date":30,"source":70}},"licensed":{"declared":"NOASSERTION","toolScore":{"total":15,"declared":0,"discovered":0,"consistency":15,"spdx":0,"texts":0},"score":{"total":15,"declared":0,"discovered":0,"consistency":15,"spdx":0,"texts":0}},"coordinates":{"type":"maven","provider":"mavencentral","namespace":"org.flywaydb","name":"flyway-maven-plugin","revision":"5.0.7"},"_meta":{"schemaVersion":"1.6.1","updated":"2019-11-04T21:59:21.238Z"},"scores":{"effective":57,"tool":57}}

$ curl http://localhost:4000/curations/maven/mavencentral/org.flywaydb/flyway-maven-plugin/5.0.7?expand=prs

{"curations":{},"contributions":[{"pr":{"number":387,"id":254753509,"state":"open","title":"update flyway maven plugin to the artistic license","body":"\n**Type:** Incorrect\n\n**Summary:**\nupdate flyway maven plugin to the artistic license\n\n**Details:**\nFixed the problem\n\n**Resolution:**\nChanged to the correct license\n\n**Affected definitions**:\n- flyway-maven-plugin 5.0.7","created_at":"2019-02-20T18:53:22Z","updated_at":"2019-02-20T18:53:24Z","closed_at":null,"merged_at":null,"merge_commit_sha":"377d70874899b17c054881929fdc1c4f7dd87ace","user":{"login":"clearlydefinedbot"},"head":{"sha":"cef2ce0577899f9ae429f3750fbf8ec34afb6f76","repo":{"id":115941547}},"base":{"sha":"1f8ee8bbe8200c494bdfa458b5b589dc5c0d9862","repo":{"id":115941547}}},"files":[{"path":"curations/maven/mavencentral/org.flywaydb/flyway-maven-plugin.yaml","coordinates":{"type":"maven","provider":"mavencentral","namespace":"org.flywaydb","name":"flyway-maven-plugin"},"revisions":[{"revision":"5.0.7","data":{"licensed":{"declared":"Artistic-1.0-Perl"}}}]}]}]}

$ curl http://localhost:4000/harvest/maven/mavencentral/org.flywaydb/flyway-maven-plugin/5.0.7?form=raw

{"clearlydefined":{"1":{"_metadata":{"type":"maven","url":"cd:/maven/mavencentral/org.flywaydb/flyway-maven-plugin/5.0.7","fetchedAt":"2018-03-06T00:08:41.835Z","links":{"self":{"href":"urn:maven:mavencentral:org.flywaydb:flyway-maven-plugin:revision:5.0.7:tool:clearlydefined:1","type":"resource"}
(...)
```

### Clearly Defined Crawler

TODO


### Clearly Defined Mongo DB

This container holds a Mongo database called **clearlydefined**

The database contains two collections:
* curations (contains curations)
* definitions-paged (contains definitions)

The reason the definitions database is called definitions-paged is because, previously, the definitions collection was not paged. The pagination was added in [this January 2019 pull request](https://github.com/clearlydefined/service/pull/364). Our production Azure setup includes both definitions and definitions-paged collections - the definitions-paged collection is the one that is actively used. This development environment includes the definitions-paged collection in order to more closely mirror production.

These collections are seeded using the following container.

### Clearly Defined Mongo Seed

This container exists only to seed initial data into the Clearly Defined Mongo DB. It populates both the collections and definitions-paged collections with sample data.