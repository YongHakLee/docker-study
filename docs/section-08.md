# Building a Multi-Container Application

## Single Container Deployment Issues

- THe app was simple - no outside dependencies
- Our image was built multiple times
- How do we connect to a database from a container?

## Application Overview

- User submits number -> React App -> Express Server
- Express Server -> Postgres: Stores a permanent list of indices that have been received.
- Express Server -> Redis: Stores all indices and calculated values as key-value pairs.
- Redis -> Worker, Worker -> Redis: Watches Reids for new indices. Pulls each new indice, calculates new value than puts it back into redis.

## Setup Application
