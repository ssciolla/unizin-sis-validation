# unizin-validation

## Overview

The unizin-validation project contains a program written in Python that attempts to ensure Canvas data was successfully
loaded each night into Unizin data sources, including the Unizin Data Warehouse (UDW) and the Unizin Data Platform (UDP).
The program does this by running SQL queries against the data sources and performing basic checks on the results to detect
irregularities. The queries and checks used are defined in `dbqueries.py`. CSV files with the query results are generated
as part of the workflow.

## Development

### Pre-requisities

The sections below provide instructions for configuring, installing and using the application.
Depending on the environment you plan to run the application in, you may need to install one of the following:

* [Python 3.8](https://docs.python.org/3/)
* [Docker Desktop](https://www.docker.com/products/docker-desktop)

## Configuration

Configuration variables for the program, `validate.py`, are loaded using a JSON file, typically called `env.json`.
To create your version of this file, make a copy of the `env_sample.json` template from the project's `config` directory;
then, add the connection parameters for each data source in the proper nested JSON object.
To connect to these data sources, you will likely need to use a VPN or Ethernet connection with the necessary permissions.
You can also use the configuration file to set the [Python logging level](https://docs.python.org/3/library/logging.html)
(with `LOG_LEVEL`) and the path CSV files will be written to (with `OUT_DIR`).

## Installation & Usage

### With `virtualenv`

To install and run the validation program using a Python virtual environment, do the following:

1. Place the `env.json` file described in the **Configuration** section (above) in the `config` directory.

2. Create and activate a virtual environment.
    ```sh
    virtualenv venv
    source venv/bin/activate  # for Mac OS
    ```

3. Install the dependencies.
    ```sh
    pip install -r requirements.txt
    ```

4. Run the program.
    ```sh
    python validate.py
    ```

    Optionally, you can specify one of two pre-defined jobs -- `UDW` or `Unizin` -- as an additional parameter.
    The default job value is `UDW`. The `Unizin` job includes an additional query and check against the UDP Context Store.
    **Note**: these paraemters are not yet supported when using Docker (see below).

CSV files containing the query results will be written to the value of the `OUT_DIR` configuration variable
(the default is the `data` directory).

## With Docker

The validation program can also be installed and run with Docker using volume mounts. To do so, perform the following steps:

1. Create directories at `~/secrets/unizin-validation` and `~/data/unizin-validation` on your machine,
where `~` is your user's home directory.

2. Place the `env.json` file described in the **Configuration** section (above) in the `~/secrets/unizin-validation` directory.

3. Build a Docker image for the project.
    ```sh
    docker build -t unizin-validation .
    ```

4. Run a container using the just-created image and two volume mounts for input and output,
replacing `{absolute_path}` with the absolute path to your home directory.
    ```sh
    docker run \
        --mount type=bind,source=/{absolute_path}/secrets/unizin-validation,target=/app/config \
        --mount type=bind,source=/{absolute_path}/data/unizin-validation,target=/app/data \
        unizin-validation
    ```

**Note**: the Docker steps above assume you are have specified the value of `OUT_DIR` as the `data` directory.
If that is properly set,
CSV files containing the query results will be written to the `~/data/unizin-validation` directory on your machine.