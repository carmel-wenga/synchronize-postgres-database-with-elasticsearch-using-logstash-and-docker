This repository shows how to synchronize a postgres database with elasticsearch using docker and 
docker-compose.
We use docker-compose to define the following three containers : 
* postgres : it creates table ```users``` into a database ```datasync```, and insert three users into the 
```users``` table. 
* elasticsearch
* logstash

1. Clone the repository 
```commandline
git clone https://github.com/carmel-wenga/synchronize-postgres-database-with-elasticsearch-using-logstash-and-docker.git
```

2. Build and launch the application with the following command

```commandline
cd synchronize-postgres-database-with-elasticsearch-using-logstash-and-docker
docker-compose up -d --build
```
3. Make sure the ```users``` table have been created into the ```datasync``` database and the it contains three rows.

    * Connect to the postgres container
        ```commandline
        docker-compose exec postgres sh
        ```
    * Once in the container, run the following command to connect to postgres
        ```commandline
        psql -U datasync -d datasync
        ```
      You will be connected to the datasync database. You can run ```\l``` to list all the databases and 
      ```\d``` to list all the tables (relations).
    * Then run the sql query below to list all the users created in the ```users``` table
        ```sql
        select * from users;
        ```
      you should have the following result
        ```txt
         user_id | username  |     user_email     | user_first_name | user_last_name | user_birthdate |         created_on         |        last_update         
        ---------+-----------+--------------------+-----------------+----------------+----------------+----------------------------+----------------------------
               1 | cwenga    | cwenga@carml.ai    | carmel          | wenga          | 1990-09-20     | 2023-04-25 14:14:47.283077 | 2023-04-25 14:14:47.283077
               2 | smenguope | smenguope@carml.ai | suzie           | menguope       | 1992-11-13     | 2023-04-25 14:14:47.283077 | 2023-04-25 14:14:47.283077
               3 | cdiogni   | cdiogni@carml.ai   | christian       | diogni         | 1992-10-13     | 2023-04-25 14:14:47.283077 | 2023-04-25 14:14:47.283077
        (3 rows)

        ```
4. Check that the database have been synchronized with elasticsearch

The logstash job runs every 10 seconds to get newly created or updated rows in the 
database and write it in elacticsearch. The elasticsearch service runs on 
```http://localhost:9200```

Use a web browser or postman to see what you get with this url
```commandline
http://localhost:9200/*/_search
```
You should have the following result
```json
{
    "took": 2,
    "timed_out": false,
    "_shards": {
        "total": 1,
        "successful": 1,
        "skipped": 0,
        "failed": 0
    },
    "hits": {
        "total": {
            "value": 3,
            "relation": "eq"
        },
        "max_score": 1.0,
        "hits": [
            {
                "_index": "users",
                "_id": "1",
                "_score": 1.0,
                "_source": {
                    "last_update": "2023-04-25T14:14:47.283077Z",
                    "created_on": "2023-04-25T14:14:47.283077Z",
                    "user_last_name": "wenga",
                    "username": "cwenga",
                    "user_first_name": "carmel",
                    "user_id": 1,
                    "user_email": "cwenga@carml.ai",
                    "user_birthdate": "1990-09-20T00:00:00.000Z"
                }
            },
            {
                "_index": "users",
                "_id": "3",
                "_score": 1.0,
                "_source": {
                    "last_update": "2023-04-25T14:14:47.283077Z",
                    "created_on": "2023-04-25T14:14:47.283077Z",
                    "user_last_name": "diogni",
                    "username": "cdiogni",
                    "user_first_name": "christian",
                    "user_id": 3,
                    "user_email": "cdiogni@carml.ai",
                    "user_birthdate": "1992-10-13T00:00:00.000Z"
                }
            },
            {
                "_index": "users",
                "_id": "2",
                "_score": 1.0,
                "_source": {
                    "last_update": "2023-04-25T14:14:47.283077Z",
                    "created_on": "2023-04-25T14:14:47.283077Z",
                    "user_last_name": "menguope",
                    "username": "smenguope",
                    "user_first_name": "suzie",
                    "user_id": 2,
                    "user_email": "smenguope@carml.ai",
                    "user_birthdate": "1992-11-13T00:00:00.000Z"
                }
            }
        ]
    }
}
```
We have exactly 3 documents in the ```users``` index of elasticsearch.

5. Now is time to see if our database is synced with elacticsearch.

Go back to the postgres container (step 3.) and connect to the datasync database.

Run the following sql query to add a new row in the ```users``` table

```sql
INSERT INTO users(
    username,
    user_email,
    user_first_name,
    user_last_name,
    user_birthdate,
    created_on,
    last_update
)
VALUES 
    ('rsmith','rsmith@carml.ai','rouhan','smith','1991-09-20',CURRENT_TIMESTAMP,CURRENT_TIMESTAMP);
```
Now the database as 4 rows
```sql
select * from users;
```
output
```text
 user_id | username  |     user_email     | user_first_name | user_last_name | user_birthdate |         created_on         |        last_update         
---------+-----------+--------------------+-----------------+----------------+----------------+----------------------------+----------------------------
       1 | cwenga    | cwenga@carml.ai    | carmel          | wenga          | 1990-09-20     | 2023-04-25 14:14:47.283077 | 2023-04-25 14:14:47.283077
       2 | smenguope | smenguope@carml.ai | suzie           | menguope       | 1992-11-13     | 2023-04-25 14:14:47.283077 | 2023-04-25 14:14:47.283077
       3 | cdiogni   | cdiogni@carml.ai   | christian       | diogni         | 1992-10-13     | 2023-04-25 14:14:47.283077 | 2023-04-25 14:14:47.283077
       4 | rsmith    | rsmith@carml.ai    | rouhan          | smith          | 1991-09-20     | 2023-04-25 14:40:48.042288 | 2023-04-25 14:40:48.042288
(4 rows)
```

The ```users``` index in elactisearch should also contain the new user

```json
{
    "took": 564,
    "timed_out": false,
    "_shards": {
        "total": 1,
        "successful": 1,
        "skipped": 0,
        "failed": 0
    },
    "hits": {
        "total": {
            "value": 4,
            "relation": "eq"
        },
        "max_score": 1.0,
        "hits": [
            {
                "_index": "users",
                "_id": "1",
                "_score": 1.0,
                "_source": {
                    "last_update": "2023-04-25T14:14:47.283077Z",
                    "created_on": "2023-04-25T14:14:47.283077Z",
                    "user_last_name": "wenga",
                    "username": "cwenga",
                    "user_first_name": "carmel",
                    "user_id": 1,
                    "user_email": "cwenga@carml.ai",
                    "user_birthdate": "1990-09-20T00:00:00.000Z"
                }
            },
            {
                "_index": "users",
                "_id": "3",
                "_score": 1.0,
                "_source": {
                    "last_update": "2023-04-25T14:14:47.283077Z",
                    "created_on": "2023-04-25T14:14:47.283077Z",
                    "user_last_name": "diogni",
                    "username": "cdiogni",
                    "user_first_name": "christian",
                    "user_id": 3,
                    "user_email": "cdiogni@carml.ai",
                    "user_birthdate": "1992-10-13T00:00:00.000Z"
                }
            },
            {
                "_index": "users",
                "_id": "2",
                "_score": 1.0,
                "_source": {
                    "last_update": "2023-04-25T14:14:47.283077Z",
                    "created_on": "2023-04-25T14:14:47.283077Z",
                    "user_last_name": "menguope",
                    "username": "smenguope",
                    "user_first_name": "suzie",
                    "user_id": 2,
                    "user_email": "smenguope@carml.ai",
                    "user_birthdate": "1992-11-13T00:00:00.000Z"
                }
            },
            {
                "_index": "users",
                "_id": "4",
                "_score": 1.0,
                "_source": {
                    "last_update": "2023-04-25T14:40:48.042288Z",
                    "created_on": "2023-04-25T14:40:48.042288Z",
                    "user_last_name": "smith",
                    "username": "rsmith",
                    "user_first_name": "rouhan",
                    "user_id": 4,
                    "user_email": "rsmith@carml.ai",
                    "user_birthdate": "1991-09-20T00:00:00.000Z"
                }
            }
        ]
    }
}
```

6. Update existing rows in the ```users``` table

Run the update query below 
```sql
UPDATE users 
SET user_first_name='carmel new',
user_last_name='wenga new',
last_update=current_timestamp
WHERE username='cwenga'
```
In elacticsearch, you will still have 4 users, but user with ```username='cwenga'```
will have been updated
```json
{
    "_index": "users",
    "_id": "1",
    "_score": 1.0,
    "_source": {
        "last_update": "2023-04-25T14:50:42.328449Z",
        "created_on": "2023-04-25T14:14:47.283077Z",
        "user_last_name": "wenga new",
        "username": "cwenga",
        "user_first_name": "carmel new",
        "user_id": 1,
        "user_email": "cwenga@carml.ai",
        "user_birthdate": "1990-09-20T00:00:00.000Z"
    }
}
```