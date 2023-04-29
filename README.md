# Apache Airflow POC
Using this as a POC to futher evaluate Airflow and how it utilizes DAGs in Python for data pipelines. I originally tried to install Airflow manually, but then opted to go the Docker route instead.

## Setup Airflow using Docker

### Install Docker and create a new folder
- Install Docker and Docker-compose
- Open a Linux terminal (I used Ubuntu in WSL)
- Create a new directory using ``mkdir airflow-docker``
- Change into the new directory with ``cd airflow-docker``

### Configure Docker
```
# Download the docker-compose.yaml file
curl -LfO 'https://airflow.apache.org/docs/apache-airflow/stable/docker-compose.yaml'

# Make expected directories and set an expected environment variable
mkdir ./dags ./logs ./plugins
echo -e "AIRFLOW_UID=$(id -u)\nAIRFLOW_GID=0" > .env

# Initialize the database
docker-compose up airflow-init

# Start up all services
docker-compose up
```

### Interact with Airflow
Navigate to http://localhost:8080/ and login with user ``airflow`` and pw ``airflow``

You can also connect to the API
``curl -X GET "http://localhost:8080/api/v1/dags"``

You will likely get an "unathorized" error. To get around this open the docker-compose.yaml file and under the "ENVIRONMENT" variables add a new variable:
``AIRFLOW__API__AUTH_BACKENDS: 'airflow.api.auth.backend.basic_auth'``

Restart the services
``docker-compose down && docker-compose up``

Now try the command again, but this time passing in the user and pw
``curl -X GET --user "airflow:airflow" "http://localhost:8080/api/v1/dags"``

## Setup Airflow the manual (non-Docker) way
I originally went this route but then abandoned it for the Docker version of Airflow

I referenced the [Airflow quick start](https://airflow.apache.org/docs/apache-airflow/stable/start.html) documentation for the initial installation.

### Clone this repo
1) Install [Python](https://www.python.org/downloads/) if you do not already have it
2) Clone this repo - I cloned into an Ubuntu shell in WSL 
3) Create a virtual environment and then activate it
    ```
    python3 -m venv venv
    source venv/bin/activate
    ```
4) Verify your Python version - I am on 3.8.10
    ``python3 -V``
5) Now identify a constraints file from the [Airflow project page](https://github.com/apache/airflow) that you can use for library imports. This should match your installed Python version. The [contraints-main](https://github.com/apache/airflow/tree/constraints-main) branch of their repo has files for multiple versions. Verify which of these files that you need and then plug it into the link below. Plug that link into your browser to be sure it exists and you don't hit a "not found" error:
    https://raw.githubusercontent.com/apache/airflow/constraints-main/constraints-no-providers-3.8.txt

### Install Airflow
```
    export AIRFLOW_HOME=~/airflow
    AIRFLOW_VERSION=2.5.0
    CONSTRAINT_URL="https://raw.githubusercontent.com/apache/airflow/constraints-${AIRFLOW_VERSION}/constraints-${PYTHON_VERSION}.txt"
    pip install "apache-airflow==${AIRFLOW_VERSION}" --constraint "${CONSTRAINT_URL}"
```

### Run Airflow
```
    # Initialize the DB
    airflow db init

    # Run the websever in daemon mode (as a background process)
    airflow webserver --port 8080 -D
    
    # Run the scheduler in daemon mode
    airflow scheduler -D

    # View the daemon processes after they are running
    ps -T # processes associated with this terminal
    ps -r # all running processes
```

### DB Connection
Airflow comes with a SQLLite DB out of the box. This is fine for very light testing and dev, but you would need to move to a more substantial DB for anything heavier.

You can view the current connection string that is being used in SQL Alchemy using ``airflow config get-value database sql_alchemy_conn``

### DAGs
Airflow comes with a bunch of example DAGs out of the box. If you don't want to see these anymore you can actually disable these in the config.

1) Open the config file at ``~/airflow/airflow.cfg`` and set ``load_examples = False``
2) Reset the Airflow DB with ``airflow db reset``
3) Restart the webserver and scheduler as daemons
    ```
    airflow webserver -D
    airflow scheduler -D
    ```

If Terminal tells you Airflow is already running, get the process ID of the task running on port 8080 with ``lsof -i tcp:8080`` and then use ``kill <pid>`` to terminate the process. Now you should be able to run both Webserver and Scheduler.