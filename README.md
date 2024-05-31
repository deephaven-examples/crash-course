# Deephaven with Crash Course notebooks built in

This repository contains a Deephaven deployment with [crash course notebooks](https://deephaven.io/core/docs/tutorials/crash-course/) built in. The notebooks are designed to help you get started with Deephaven and learn the basics of working with tables, plots, data I/O, and more.

Some notebooks from the crash course are left out because they do not deal with Deephaven's Python API, which is the focus of this repository. The notebooks included in this deployment are:

- [Architecture overview](https://deephaven.io/core/docs/tutorials/crash-course/architecture-overview/)
- [Create tables](https://deephaven.io/core/docs/tutorials/crash-course/create-tables/)
- [Table operations](https://deephaven.io/core/docs/tutorials/crash-course/table-ops/)
- [Query strings](https://deephaven.io/core/docs/tutorials/crash-course/query-strings/)
- [Python integrations](https://deephaven.io/core/docs/tutorials/crash-course/py-integrations/)
- [Plots](https://deephaven.io/core/docs/tutorials/crash-course/plots/)
- [Export data](https://deephaven.io/core/docs/tutorials/crash-course/export-data/)

The following notebooks are omitted, but are still useful for learning about Deephaven:

- [Getting started](https://deephaven.io/core/docs/tutorials/crash-course/get-started/)
- [Configure your instance](https://deephaven.io/core/docs/tutorials/crash-course/configure/)
- [Wrapping up](https://deephaven.io/core/docs/tutorials/crash-course/crash-course-wrap-up/)

## Prerequisites

This repository has the same prerequisites as listed in our [Docker installation guide](https://deephaven.io/core/docs/tutorials/docker-install/#prerequisites). They are:

- git
- Docker (version `20.10.8` or later)
  - Docker Compose v2
- Windows 10 or later (for Windows users only)
- WSL 2 (For users running Linux via Windows)

To check if Docker is installed:

```sh
docker version
docker compose version
docker run hello-world
```

## Using this repository

This repository contains a basic installation of Deephaven Community Core. It includes [example data](https://github.com/deephaven/examples) and the crash course notebooks. To get started, first clone this repository and move into the cloned repo:

```sh
git clone git@github.com:deephaven-examples/crash-course.git
cd crash-course
```

Then, pull the Docker images specified in the `docker-compose.yml` file:

```sh
docker compose pull
```

Lastly, start Deephaven:

```sh
docker compose up
```

Navigate to `http://localhost:10000/ide` in your web browser (we highly recommend a Chromium based browser or Firefox) to access the Deephaven IDE. It will bring you to a login screen for which the password has been set as `Cr@shC0urse`. You can change this password by modifying the `docker-compose.yml` file. The password is specified on line 9.

## Crash course notebooks

The crash course notebooks can be found in the top right corner of the UI. The notebooks are designed to be followed in the order they are given. They are interactive and will guide you through the basics of working with Deephaven. We recommend, for new users, to read along as you work through the code blocks.

## Reach out

If you have questions, comments, or concerns, please reach out to us in our [Slack Community](https://deephaven.io/slack), or file a [ticket](https://github.com/deephaven-examples/crash-course/issues/new) in the repository.