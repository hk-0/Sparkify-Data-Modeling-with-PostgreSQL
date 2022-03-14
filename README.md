# Project Sparkify - Data Modeling with PostgreSQL

## Introduction

A fictional music streaming startup called Sparkify wants to analyze the data they've been collecting on songs and user activity on their new music streaming app. The analytics team is particularly interested in understanding what songs users are listening to. Currently, they don't have an easy way to query their data, which resides in a directory of JSON logs on user activity on the app, as well as a directory with JSON metadata on the songs in their app.
They'd like a data engineer to create a Postgres database with tables designed to optimize queries on song play analysis, and bring you on the project. The data engineers role is to create a database schema and ETL pipeline for this analysis.

## Project Description

Define the fact and dimension tables for a star schema.
Write an ETL pipeline in Python and SQL to read the JSON logs and JSON metadata and upload the data into a PostgreSQL database. The ETL process will extract relevant data from the files in order to allow easy querying of user info , user listening habits, user subscription info, etc. 

## Database Model and Schema

The data model is implemented using a star schema. The schema contains one fact table, `songplays` and four dimension tables: `songs`, `artists`, `users` and `time`.

#### Fact Tables

**`songplay`**
| COLUMN      | TYPE      | CONSTRAINT  |
|-------------|-----------|-------------|
| songplay_id | SERIAL    | PRIMARY KEY |
| start_time  | TIMESTAMP | NOT NULL    |
| user_id     | INT       | NOT NULL    |
| level       | VARCHAR   |             |
| song_id     | VARCHAR   |             |
| artist_id   | VARCHAR   |             |
| session_id  | INT       |             |
| location    | VARCHAR   |             |
| user_agent  | VARCHAR   |             |

#### Dimension Tables

**`users`**
| COLUMN     | TYPE    | CONSTRAINT  |
|------------|---------|-------------|
| user_id    | INT     | PRIMARY KEY |
| first_name | VARCHAR | NOT NULL    |
| last_name  | VARCHAR | NOT NULL    |
| gender     | VARCHAR |             |
| level      | VARCHAR |             |

**`songs`**
| COLUMN    | TYPE    | CONSTRAINT  |
|-----------|---------|-------------|
| song_id   | VARCHAR | PRIMARY KEY |
| title     | VARCHAR | NOT NULL    |
| artist_id | VARCHAR | NOT NULL    |
| year      | INT     | NOT NULL    |
| duration  | NUMERIC | NOT NULL    |

**`artist`**
| COLUMN    | TYPE             | CONSTRAINT  |
|-----------|------------------|-------------|
| artist_id | VARCHAR          | PRIMARY KEY |
| name      | VARCHAR          | NOT NULL    |
| location  | VARCHAR          |             |
| latitude  | DOUBLE PRECISION |             |
| longitude | DOUBLE PRECISION |             |

**`time`**
| COLUMN     | TYPE      | CONSTRAINT  |
|------------|-----------|-------------|
| start_time | TIMESTAMP | PRIMARY KEY |
| hour       | INT       |             |
| day        | INT       |             |
| week       | INT       |             |
| month      | INT       |             |
| year       | INT       |             |
| weekday    | INT       |             |

## File Structure And Description

```.
├── data/
│   ├── log_data/
│   │   └── 2018/
│   │       └── 11/
│   │           └── *.json  
│   └── song_data/
│       └── A/
│           ├── A/
│           │   ├── A/
│           │   │   └── TRAAA*.json
│           │   ├── B/
│           │   │   └── TRAAB*.json
│           │   └── C/
│           │       └── TRAAC*.json
│           └── B/
│               ├── A/
│               │   └── TRABA*.json
│               ├── B/
│               │   └── TRABB*.json
│               └── C/
│                   └── TRABC*.json
├── create_tables.py (create_tables.py contains sql scripts to create the required database tables with the appropriate constraints)
├── etl.ipynb (Jupyter Notebook to test ETL process sqls)
├── etl.py (etl.py iterates the json files in song_data and log_data folders returns data in list format to insert into database tables )
├── README.md
├── sql_queries.py (Contains all the create sql queries needed to create the database tables when calling create_tables.py. Also contains the Inserts sql to insert data returned by etl.py)
└── test.ipynb (Jupyter Notebook to query and test data in database)
```

## Execution

Create the database tables by executing create_tables.py
> python create_tables.py

It will create the requisite tables defined in sql_queries.py

Once the backend tables are created, data can be read and populatedusing the ETL process developed by executing etl.py
> python etl.py

Data is now available in the fact and dimension tables for analysis


## Example Queries

### Example SQL 1 : Count of users by subscription level that listened to music on sparkify in Week 46.
``` select s.level,t.week week_number,count(1) from songplays s join time t on s.start_time = t.start_time where t.week = 46 group by s.level,t.week ```

| week_number | level | count |
|------------:|------:|------:|
|          46 |  free |   302 |
|          46 |  paid |  1656 |

### Example SQL 2 : Count of users online by time of day. Break the day into 4 parts - morning (6 am to 12 pm), afternoon(12 pm to 6 pm), evening (6 pm to 12 am) and night (12 am to 6 am)
``` select 
    case 
        when t.hour >=0 and t.hour <6 then 'Night'
        when t.hour >=6 and t.hour <12 then 'Morning' 
        when t.hour >=12 and t.hour <18 then 'Afternoon' 
        else 'Evening' END as time_of_day
    ,count(distinct user_id) from songplays s join time t on s.start_time = t.start_time group by 1   limit 5;
```
    
| time_of_day | count |
|------------:|------:|
|   Afternoon |    80 |
|     Evening |    62 |
|     Morning |    63 |
|       Night |    63 |