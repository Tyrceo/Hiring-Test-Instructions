# Tyrceo Hiring Test

## Foreword
As you'll probably realize once you're done with this test, the data is as fake as the backstory.

## The Story
We ran a survey in Mallorca amongst 1st Generation Pokemon fans (Pokémon Red, Green and Blue).\
We want to answer the following question:

> Where do people who preferred to start with Bulbasaur, Charmander or Squirtle live?

The result of this survey is stored in a MySQL database.

## Instructions
Your job is:
  - to prepare the data:
    - **query** these results from the database
    - only keep the results with a score **higher or equal to 0.5**
    - **clean-up the data**:
      - the person who inserted the data in the database had either a faulty keyboard or faulty fingers
      - so there are some typos in the name of the pokemons (extra spaces, character swap,...)
      - we want you to **keep those entries** and **correct the typos**
    - **only keep** the points we are interested in (those about Bulbasaur, Charmander and Squirtle)
      - lots of people didn't really understand the question and gave non starter pokemon as answer
      - we really only want Bulbasaur, Charmander and Squirtle
    - export this data as a **geojson**
    - Note: you can visualize the result using [mapshaper](https://mapshaper.org/) or [geojson.io](http://geojson.io/)
  - to visualize the data:
    - build a simple html page with a mapbox map
    - add the geojson you prepared as a source
    - add a layer of type `circle` referencing this source
    - bonus: set a different color depending on the pokemon

## Database Structure
### Host and Credentials
The information to access the database should be included in the mail we sent you.

### Schema
```sql
CREATE TABLE `data` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `score` double NOT NULL,
  `pokemon` varchar(45) NOT NULL,
  PRIMARY KEY (`id`),
  UNIQUE KEY `uq_data_pokemon` (`score`,`pokemon`)
);

CREATE TABLE `coordinates` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `latitude` double NOT NULL,
  `longitude` double NOT NULL,
  PRIMARY KEY (`id`)
);

CREATE TABLE `data_at_coordinate` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `id_coordinate` int(11) NOT NULL,
  `id_data` int(11) NOT NULL,
  PRIMARY KEY (`id`),
  UNIQUE KEY `uq_data_coordinates` (`id_coordinate`,`id_data`),
  KEY `fk_coordinates_idx` (`id_coordinate`),
  KEY `fk_data_idx` (`id_data`),
  CONSTRAINT `fk_coordinates` FOREIGN KEY (`id_coordinate`) REFERENCES `coordinates` (`id`) ON DELETE CASCADE ON UPDATE CASCADE,
  CONSTRAINT `fk_data` FOREIGN KEY (`id_data`) REFERENCES `data` (`id`) ON DELETE CASCADE ON UPDATE CASCADE
);
```

### Explanation
#### Survey results
The result of the survey is a list of survey locations (longitude/latitude) in Mallorca,
along with the pokemon that won, and by how much.\
So a result at a given location is a tuple of `(longitude, latitude, pokemon, score)`.

#### Score
An entry like:
```
longitude: '2.6702862915153283'
latitude: '39.580342784323086',
score: '0.2',
pokemon: 'Bulbasaur'
```
Means that the survey done at (longitude,latitude)
yielded, for any given pokemon, a maximum of 20% for Bulbasaur.\
We are only interested in score **above or equal to 0.5**

#### Database design
Since we had a fair amount of repetitions in the (pokemon, score) pair,
we decided to split the tuple in two to reduce redundancy and save space:
 - The (longitude, latitude) part is in the `coordinates` table
 - The (pokemon, score) part is in the `data` table

And the `data_at_coordinate` table stores the actual result
as a unique pair of foreign keys (`id_coordinate`,`id_data`)
referencing  `coordinate` and `data` respectively.


## Mapbox
We generated a temporary token for you to use mapbox without having to create an account
It should have been included in the mail we sent you.\
Make sure you use [mapbox-gl-js](https://docs.mapbox.com/mapbox-gl-js/api/), and not the older mapbox.js

## Submission
You should at least submit the following files:
  - a Pipefile and Pipfile.lock or requirements.txt for your python dependencies
  - a python script for the data preparation
    - or optionally a jupyter notebook
  - an html file for the data vizualization
    - optionally separated css/js or you can keep everything into one html file
  - a geojson file resulting from executing the python script and accessible from the html

Once you're done with the assignement, the preferred submission method is by
sending us a link to a public repository where you've committed your solution (github, gitlab, bitbucket, etc).\
We also accept the aforementionned files zipped in a mail attachment.\
Send your submission at **desarrollo@tyrceo.com**

Example file structure
```
tyrceo_hiring_test_solution
   ├── data_wrangling
   │  ├── generate_geojson.py or .ipynb
   │  ├── Pipfile
   │  ├── Pipfile.lock
   │  └── points.geojson
   └── visualization
      ├── index.html
      └── points.geojson -> ../data_wrangling/points.geojson
```

## Suggested python version and libraries
You are free to use any libraries you want.\
You are expected to use python 3 syntax (we use 3.7.5 these days).\
You'll probably find the following libraries quite useful:
  - pandas and geopandas
  - pymysql

## Local http server
You'll probably need to run a local server when developping the visualization.\
You are also free to choose any method you want.\
If you're not sure, running the following terminal command,
in the directory of your html file should suffice.
```sh
python -m http.server
```

## Final words
The result of this test should be something totally unrealistic but non random.

The few problems you'll encounter in this test are basic data wrangling and data visualization tasks we regularily have to work on.
So the goal of this test is two-fold:
  1. giving us an idea of how you can deal with technology or problem you're not used to deal with
  2. introducing you to some of the tools you'll be using often when working at Tyrceo (python, pandas, mapbox, database, etc)

If you have any question/doubt don't hesitate to send a mail at desarrollo@tyrceo.com

Good luck and have fun.
