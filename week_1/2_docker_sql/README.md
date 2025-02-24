# ENVIRONMENT SETUP

Using WSL2 on Windows in order to run Docker Desktop. [This guide](https://learn.microsoft.com/en-us/windows/wsl/install) is helpful to get started.

Once initialized, install Anaconda from Ubuntu terminal to get access to and manage Python, Pip, Jupyter, and other Data Engineering tools.



# DOCKER

[Docker Video Lesson](https://www.youtube.com/watch?v=EYNwNlOrpr0&list=PL3MmuxUbc_hJed7dXYoJw8DoCuVHhGEQb&index=5)

Allows you to run images which are packages containing OS, software, libraries, etc.

### BENEFITS

- Reproducable: Can create an image and deploy it to numerous different systems, with all dependancies in place.
- Isolation: If an error occurs, something is misconfigured, etc. in a running instance, no worries! Your original image still exists and you can use it to recover.

# DOCKERFILE

Text file used by Docker to create an image. A simple example:

>	FROM python:3.9
>
>	RUN pip install pandas
>
>	WORKDIR /app
>	COPY pipeline.py pipeline.py
>
>	ENTRYPOINT ["python", "pipeline.py"]

These are command line commands.

- FROM indicates what the base image will be
- RUN is a build command
- WORKDIR makes the default working directory of the image
- COPY adds a file to the working directory. The last arg will always be the destination, all prior args will be copied to that destination (in this case copying our pipeline file into the container's working directory and keeping the same name)
- ENTRYPOINT specifies commands to execute from within the container. In this case, run python, then execute our pipelin script

### TO BUILD CONTAINER

	docker build .

- builds the Docker container according to the Dockerfile
- can add an optional tag with -t for example

	docker build -t test:pandas .

### TO RUN CONTAINER

	docker run -it test:pandas [args]

- runs the container tagged as test:pandas in interactive mode (-it) with some optional args passed in

# POSTGRES

[Postgres Video Lesson](https://www.youtube.com/watch?v=2JM-ziJt0WI&list=PL3MmuxUbc_hJed7dXYoJw8DoCuVHhGEQb&index=7)

Docker has built-in Postgres container. We can run the container to set up a database and populate it with a dataset

### CREATING A DB

Made a bash script to set up the postgres container via Docker

```
docker run -it \
    -e POSTGRES_USER="root" \
    -e POSTGRES_PASSWORD="root" \
    -e POSTGRES_DB="ny_taxi" \
    -v $(pwd)/ny_taxi_postgres_data:/var/lib/postgresql/data \
    -p 5432:5432 postgres:13
```

- -v is for mounting the volume. The host machine will have a dirctory mapped to the directory in the container.
- -p is the port argument. 5432 is the port on both the container and host machine

We can connect to the DB using the PGCLI package. Install with:

```
pip install pgcli
```

then, connect to the DB with 

```
pgcli -h localhost -p 5432 -u root -d ny_taxi
```

and enter the password when prompted (we set it as "root")

### INGESTING DATA

We will be populating our database with data from the NYC Taxi and Limousine Commission. They have data on rides taken via NYC Taxi cabs. We will use the [January 2021 dataset](https://github.com/DataTalksClub/nyc-tlc-data/releases/download/yellow/yellow_tripdata_2021-01.csv.gz).

We will need to take the raw data from the CSV and transform it so we can create a schema and populate it in our Postgres database. To do so, we will use Jupyter notebooks to create a Python script for the task. The script will utilize a data analysis and manipulation library called Pandas to transform the data.

Start by running

```
 jupyter notebook
```

In our notebook, we import the pandas library. Then, we create a pandas dataframe using the csv (limiting to 100 rows as the original data is over 1 million).

```
import pandas as pd

df = pd.read_csv('yellow_tripdata_2021-01.csv',nrows=100)
```

We can use the pandas module io on our dataframe to turn it into DDL (data definition language) which can be used to create our database schema.

```
print(pd.io.sql.get_schema(df, name='yellow_taxi_data'))
```

Unfortunately, pandas didn't recognize pickup and dropoff as the correct type, categorizing it as text rather than timestamp. So before we generate the DDL, let's fix that.

```
df.tpep_pickup_datetime = pd.to_datetime(df.tpep_pickup_datetime)
df.tpep_dropoff_datetime = pd.to_datetime(df.tpep_dropoff_datetime)
```

Now we are going to connect to our database using sqlalchemy.

```
from sqlalchemy import create_engine
engine = create_engine('postgresql://root:root@localhost:5432/ny_taxi')
```

Then create an python iterator so we can add our data to the db in chunks, rather than all 1 million+ simultaneously.

```
df_iter = pd.read_csv('yellow_tripdata_2021-01.csv', iterator=True, chunksize=100000)
```

We load the next 100,000 rows as the current dataframe, and repeat the datatype fix for them.

```
df = next(df_iter)
df.tpep_pickup_datetime = pd.to_datetime(df.tpep_pickup_datetime)
df.tpep_dropoff_datetime = pd.to_datetime(df.tpep_dropoff_datetime)
```

Now lets actually take the dataframe data and load it into our pg database. We will use pandas to_sql to load it, using the connection we created with sqlalchemy. We start by only loading the very first row, which is the column names,by specifying head(0).

```
df.head(0).to_sql(name='yellow_taxi_data', con=engine, if_exists='replace')
```

Finally, we can use some python to loop adding each of the chunks. We can also benchmark how long each chunk took to load.

```
while True:
    t_start = time()
    
    df = next(df_iter)
    df.tpep_pickup_datetime = pd.to_datetime(df.tpep_pickup_datetime)
    df.tpep_dropoff_datetime = pd.to_datetime(df.tpep_dropoff_datetime)

    df.to_sql(name='yellow_taxi_data', con=engine, if_exists='append')

    t_end = time()

    print('inserted another chunk..., took %.3f seconds' % (t_end - t_start))
```
NICE! We can now query some basic info about our database, such as the number of rows, or say the highest cost trip.

```
select count(1) from yellow_taxi_data
```
>+---------+
>| count   |
>|---------|
>| 1369765 |
>+---------+

```
select max(total_amount) from yellow_taxi_data
```
>+---------+
>| max     |
>|---------|
>| 7661.28 |
>+---------+

# PGADMIN

Interacting with the DB through PGCLI is cumbersome. A more versatile method is using pgAdmin, a web based GUI for viewing and manipulating a Postgres DB. Lets spin up a docker image for it.

```
docker run -it \
-e PGADMIN_DEFAULT_EMAIL="admin@admin.com" \
-e PGADMIN_DEFAULT_PASSWORD="root" \
-p 8080:80 \
dpage/pgadmin4
```

This launches Ppgdmin and maps local port 8080 to the container's port 80. So if we visit localhost:8080 in our browser, we can access the GUI.

In the GUI, we can register a new server(right click on Servers) with our Postgres docker's connection information.

> Host name/address: localhost
> Port: 5432
> Username: root
> Password: root

However, Postgres is running on one docker container, while pgAdmin is in another. We need to use Docker network to allow them to communicate.

### Docker Network

We can create a network that our Docker services can communicate over with the following.

```
docker network create pg-network
```

Then, we need to restart our Postgres and pgAdmin with additional parameters to place them on the same docker network.

```
docker run -it \
    -e POSTGRES_USER="root" \
    -e POSTGRES_PASSWORD="root" \
    -e POSTGRES_DB="ny_taxi" \
    -v $(pwd)/ny_taxi_postgres_data:/var/lib/postgresql/data \
    --network=pg-network \
    --name pg-database \
    -p 5432:5432 postgres:13
```

```
docker run -it \
    -e PGADMIN_DEFAULT_EMAIL="admin@admin.com" \
    -e PGADMIN_DEFAULT_PASSWORD="root" \
    -p 8080:80 \
    --network=pg-network \
    --name pgadmin \
dpage/pgadmin4
```

And now we can actually register the database in pgAdmin. Note, the host name needs to match the name given to the Postgres db

> Host name/address: pg-database
> Port: 5432
> Username: root
> Password: root

### CREATING INGEST SCRIPT

We will take our Jupyter notebook and turn it into a python script that can be run from the shell. This works for now, but eventually we will want to use a more versatile orchestration tool like Apache Airflow or Kestra.

In shell, we can use the following to convert the notebook to a script.

```
jupyter nbconvert --to=script upload_data.ipynb
```

We then make some modifications to the script to allow it to be executed from the command line and take in command line arguments such as username and password of the db.

```
if __name__ == '__main__':
    parser = argparse.ArgumentParser(description='Ingest CSV data to Postgres')
    parser.add_argument('user', help='user name for postgres')      
    parser.add_argument('password', help='password for postgres')
    parser.add_argument('host', help='host for postgres')      
    parser.add_argument('port', help='port for postgres')      
    parser.add_argument('db', help='database name for postgres')      
    parser.add_argument('table-name', help='name of the table we will write the results to')      
    parser.add_argument('url', help='url of the csv file')      


    args = parser.parse_args()

    main(args)
```

We will also rename the script to "ingest_data.py", which describes the process of pulling data from a source to the db.

We can call the script to create the db in the following way.

```
URL="https://github.com/DataTalksClub/nyc-tlc-data/releases/download/yellow/yellow_tripdata_2021-01.csv.gz"

python ingest_data.py \
    --user=root \
    --password=root \
    --host=localhost \
    --port=5432 \
    --db=ny_taxi \
    --table_name=yellow_taxi_trips \
    --url=${URL}
```

# DOCKERFILE AGAIN

Now, we can Dockerize the script. We will run our script through our Dockerfile so we can easily and repeatedly ingest our data to a Postgres database. 

After adding some dependencies and changing our entrypoint, our Dockerfile now looks like this.

```
FROM python:3.9

RUN apt-get install wget
RUN pip install pandas sqlalchemy psycopg2

WORKDIR /app
COPY ingest_data.py ingest_data.py

ENTRYPOINT ["python", "ingest_data.py"]
```

We can now build the container with this

```
docker build -t taxi_ingest:v001 .
```

and run it with this (note that we specify the network before the name of the image, because it is a docker parameter rather than an argument being passed into the script).

```
docker run -it --network=pg-network taxi_ingest:v001 \
    --user=root \
    --password=root \
    --host=pg-database \
    --port=5432 \
    --db=ny_taxi \
    --table_name=yellow_taxi_trips \
    --url=${URL}
```
# DOCKER COMPOSE

We now have two containers set up, one for the Postgres DB and another for pgAdmin. The commands to run them in the terminal have a cumbersome syntax, and for this situation, we would always want both running.

A solution to these issues is creating a Docker Compose file. These are used to create multiple docker containers simultaneously. As an added bonus, all containers spun up with the same compose file will automatically be placed in a network, saving us a bit of work.

We create the docker-compose.yaml file and modify our two sets of terminal commands.

```
services:
  pgdatabase:
    image: postgres:13
    environment:
      - POSTGRES_USER=root
      - POSTGRES_PASSWORD=root
      - POSTGRES_DB=ny_taxi
    volumes:
      - "./ny_taxi_postgres_data:/var/lib/postgresql/data:rw"
    ports:
    - "5432:5432"
  pgadmin:
    image: dpage/pgadmin4
    environment:
      - PGADMIN_DEFAULT_EMAIL=admin@admin.com
      - PGADMIN_DEFAULT_PASSWORD=root
    ports:
      - "8080:80"
```

We can then execute the compose file with

```
docker-compose up
```

If you add -d as an argument, you run in detached mode, which gives you back access to the terminal. When finished, you can shutdown your services with

```
docker-compose down
```

# SQL

We are going to do some SQL queries on our DB. Quickly we will add a new table to the DB that will let us map the trips to different areas of NYC.

We can run the following in Jupyter to get the data and put it to our DB.

```
!wget https://d37ci6vzurychx.cloudfront.net/misc/taxi_zone_lookup.csv

df_zones = pd.read_csv('taxi_zone_lookup.csv')

df_zones.to_sql(name='zones', con=engine, if_exists='replace')
```

Now we have added the zone coding as a new table to our ny_taxi DB. In pgAdmin, we can query our trip table and join it to our zones table where the PULocation ID or DOLocationID matches a LocationID.

```
SELECT * 
FROM 
  yellow_taxi_trips t JOIN zones zpu
  ON t."PULocationID" = zpu."LocationID" JOIN zones zdo 
  ON t."DOLocationID" = zdo."LocationID"
WHERE 
	t."PULocationID" = zpu."LocationID" AND
	t."DOLocationID" = zdo."LocationID"
LIMIT 100;
```

This works, but the results are cluttered and way more info than we need. Let's simplify it by replacing the * with the following.

```
tpep_pickup_datetime,
tpep_dropoff_datetime,
t."PULocationID",
t."DOLocationID",
CONCAT(zpu."Borough", '/', zpu."Zone") AS "pickup_location",
CONCAT(zdo."Borough", '/', zdo."Zone") AS "dropoff_location"
```

Let's also verify the quality of our dataset. Are there any entries missing a pickup or dropoff location?

```
SELECT 
"PULocationID",
"DOLocationID"
FROM yellow_taxi_trips t
WHERE "DOLocationID" is NULL
LIMIT 100;
```

How about any trips with a Location ID that doesn't actually correspond to a real zone?

```
SELECT 
"PULocationID",
"DOLocationID"
FROM yellow_taxi_trips t
WHERE "DOLocationID" NOT IN (SELECT "LocationID" FROM zones)
```

In this case, we find no missing values. But if we had some, that may cause our initial query where we joined the two tables to fail. *If for example, there was a LocationID missing from the zone table, values from the trips table with that zone would not appear in our query*. This is a product of doing an **inner join**.

If we wanted all rows to appear from both tables, even if some values would be null, we need to instead perform an **outer join**. In the above scenario, replace JOIN with LEFT JOIN means we will still be left with trip values even if no LocationID can be matched to the zones table.

As an explainer:

>A    B
>-    -
>1    3
>2    4
>3    5
>4    6
>
>select * from a JOIN b on a.a = b.b;
>
>a | b
>--+--
>3 | 3
>4 | 4
>
>select * from a LEFT JOIN b on a.a = b.b;
>
>a |  b
>--+-----
>1 | null
>2 | null
>3 |    3
>4 |    4

Finally, we can do some useful analytics using GROUP BY! Let's say we wanted to know the following:

>Each day, what drop off location had the most trips, and what was the highest fare to it?

The query might look like this.
```
SELECT
	CAST(tpep_dropoff_datetime AS DATE) AS "day",
	"DOLocationID",
	count(1) as "count",
	MAX(total_amount)
FROM 
	yellow_taxi_trips t
GROUP BY
	1, 2
ORDER BY 
	1 ASC, 
	2 ASC;
```

Note: Using 1 and 2 in the ORDER BY and GROUP BY clauses are aliases for the first and second columns listed in the SELECT clause.