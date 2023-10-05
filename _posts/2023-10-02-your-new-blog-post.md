# PROJECT: Building PlantTime.com - a Plant Recommendation System (Using AWS DynamoDB, Cloudformation, Lambda, and a CI/CD Github Workflow)


**This is a WIP**

PlantTime URL: xxx (Front-end not yet finished)

#### Preliminary

First thing's first, what do we want to build? I don't want to spend too much time on this part as the point of this post is to document the building process. 

Myself and my front-end developer friend had a great idea for a website we'd both enjoy wherein a user can enter their location, the date they want to begin to grow a plant and the soil they have. The website will recommend a plant to them based on a multi-dimensional
ML model, built by yours truly, taking into account precipitation, UV index, elevation, soil type, season, and forecasted (by the model) weather! The site will also display a "Plant of the Week" and the back-end data will be used to build 1-2 graphs from the data.

My friend is a very talented front-end dev but he knows nothing about back-end work, so I'll be taking care of anything involving the back-end systems.
This means I'll be **architecting the system design, building the database (with high-availability and scalability), setting up endpoints, and automating API querying!**


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



#### Parsing, Lambda, and DynamoDB

Okay, now I have to quickly explain why I chose DynamoDB over RDS or Redshift. This comes down to exactly one reason: cost.
While RDS and Redshift charge no matter how much you're using the DB, DynamoDB scales down to 0 if no-one is using it.
This comes at a cost of slower response time if it's the first time someone queries the website in a while, but this is a price I'm
willing to pay to avoid the price of RDS and Redshift (get it?).

Let's parse the data. 

First, You'll notice that our "weather_data" dictionary has a key called 'daily', which itself is another dictionary. Within this 'daily' dictionary, there are other keys like 'time', 'weathercode', 'temperature_2m_max', and so on. Each of these keys maps to a list.

The key idea is that these lists are parallel. This means the first element in the 'time' list corresponds to the same day as the first element in the 'weathercode' list, 'temperature_2m_max' list, and so on.

What does this mean? We can use a for loop to parse!
Specifically, if we do:

```
for i in range(len(weather_data['daily']['time'])):
```

This will make it such that we're setting up a loop that iterates once for each element in the 'time' list. The iterator 'i' 
takes on values from '0' to the lenth of the dictionary.

Here's the full for loop:

```
for i in range(len(weather_data['daily']['time'])):
            date = weather_data['daily']['time'][i]
            weathercode = weather_data['daily']['weathercode'][i] or 0
            temperature_max = weather_data['daily']['temperature_2m_max'][i] or 0.0
            temperature_min = weather_data['daily']['temperature_2m_min'][i] or 0.0
            uv_index_max = weather_data['daily']['uv_index_max'][i] or 0.0
            precipitation_sum = weather_data['daily']['precipitation_sum'][i] or 0.0
```



Inside the loop when we access weather_data['daily']['time'][i], we're getting the i-th element of the 'time' list. Since the data is parallel and the one thing that should never be or end early is the date, this is the safest bet to iterate our loop on! So on the first loop iteration (when i is 0), this would get us '2022-09-30'. On the second iteration (when i is 1), it would get us '2022-10-01', and so forth. Also, this gets us the same parallel data per dictionary.

Alright, that's the basics of our Lambda function for parsing the weather data from the API. Of course there's more to the code than this, but that's the meat and potatoes. Now we can Zip this file, upload it into an S3 bucket and build our Cloudformation YAML template. More on that in the next section. **Note:** For the zip file, what I did is use a git environment in my terminal for version control, so that I can keep track of my changes and revert back to an old version if something goes wrong. You don't have to do this, but it helps in case you make a mistake.






