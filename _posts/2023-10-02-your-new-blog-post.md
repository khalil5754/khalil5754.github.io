# PROJECT: Building PlantTime.com - a Plant Recommendation System (Using a Weather API, AWS Aurora Serverless, Cloud-Formation, Lambda, and a CI/CD Github Workflow)

PlantTime URL: xxx (Front-End not yet finished)

##### Preliminary

First thing's first, what do we want to build? I don't want to spend too much time on this part as the point of this post is to document the building process. 

Myself and my Front-end developer friend had a great idea for a website we'd both enjoy wherein a user can enter their location, the date they want to grow a plant and the soil they have. The website will recommend a plant to them based on a multi-dimensional
ML model, built by yours truly, taking into account precipitation, UV index, elevation, soil type, season, and forecasted (by the model) weather! The site will also display a "Plant of the Week" and the back-end data will be used to build 1-2 graphs
from the data.

My friend is a very talented front-end dev but he knows nothing about back-end work, so I'll be taking care of anything involving the back-end systems.
This means I'll be architecting the system design, building the DBs, setting up endpoints and 
Okay, let's get started.


#### Groundwork

To lay the groundwork for this project I'm first going to be using a weather API from [open-meteo.com](https://open-meteo.com). The first time we load this data in to look at it, we simply call the API with Python (you can just use your browser or "curl"
in your terminal, but that's less fun):

```
import requests
url = 'https://api.open-meteo.com/v1/forecast?latitude=52.52&longitude=13.41&daily=weathercode,temperature_2m_max,temperature_2m_min,uv_index_max,precipitation_sum&timezone=America%2FNew_York&start_date=2022-09-30&end_date=2023-10-05'
resonse = requests.get(url)

weather_data = response.json() #Convert the output into a JSON for parsing
print(weather_data)
```
What this returns is a JSON of the weather data, in the form:  

```
{'latitude': 52.52, 'longitude': 13.419998, 'timezone': 'America/New_York', 'timezone_abbreviation': 'EDT', 'elevation': 38.0, 'daily_units':
{'time': 'iso8601', 'weathercode': 'wmo code', 'temperature_2m_max': '°C', 'temperature_2m_min': '°C', 'uv_index_max': '', 'precipitation_sum': 'mm'}, 'daily':
{'time': ['2022-09-30', '2022-10-01', '2022-10-02', '2022-10-03', '2022-10-04] 

'temperature_2m_max': [5.7, 5.6, 6.1, 8.3, 3.8]

'uv_index_max': [2.0, 2.55, 1.75, 2.3, 3.4]

'precipitation_sum': [18.7, 2.9, 1.8, 0.0, 3.6]
```

There is a lot more data being pulled, but that should give you a solid idea. Of course whatever DB we use is not going to take this 
data as it is, so we have to parse it and input it into our database. The plan is to insert the past year's data into our database so
that we can have our graphs show historical weather data for comparison with the current weather. 

We also want to build an ML forecasting model but that will likely be out of the scope of this post, but tune in for a 
future post about it.

#### Parsing, Lambda, and Aurora Serverless

Okay, now I have to quickly explain why I chose Aurora Serverless over RDS or Redshift. This comes down to exactly one reason: cost.
While RDS and Redshift charge no matter how much you're using the DB, Aurora Serverless scales down to 0 if no-one is using it.
This comes at a cost of slower response time if it's the first time someone queries the website in a while, but this is a price I'm
willing to pay to avoid the price of RDS and Redshift (get it?).

Let's parse the data. 
