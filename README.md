# PlanMyTrip app
## 1. Project
  The purpose of the project is to create the following:
  - Retrieve Weather Data 
  - Create a Customer Travel Destination Map
  - Create a Travel Iteneray Map 
  
## 2. Resources
  * Data source: Vacation_Search_starter_code.ipynb,  Vacation_Itinerary_starter_code.ipynb
  * Software: Python 3.7.6, Visual Studio Code version 1.58
  * web application: Jupyter Notebook
  * Output Data Files: WeatherPy_Database.csv, cities.csv, WeatherPy_vacation.csv 
  * Output .JPEG Files: WeatherPy_travel_map.png, WeatherPy_travel_map_markers.png, WeatherPy_vacation_map.png
  * Google map data: OpenWeatherMap
## 3. Analysis
### 3.1 Weather Data Retrieval
        We need to generate a set of 2000 random latitudes and longitudes, and then retrieve nearest city.
        We need to create an API key and use the key to perform an API call to the google OpenWeatherMap.
        First we import the dependencies and modules we need. The code and comments are shown below;
        # Import the random module.
          import random
          # Import the dependencies.
          import pandas as pd
          import matplotlib.pyplot as plt
          import numpy as np
          %matplotlib inline
          
          # Create a set of random latitude and longitude combinations.
          #we'll pack the latitudes (lats) and longitudes (lngs) as pairs by zipping them (lat_lngs) with the zip() function.
          lats = np.random.uniform(low=-90.000, high=90.000, size=2000)
          lngs = np.random.uniform(low=-180.000, high=180.000, size=2000)
          lat_lngs = zip(lats, lngs)
          lat_lngs
         
        The zip object packs each pair of lats and lngs having the same index in their respective array into a tuple. 
        If there are 2000 latitudes and longitudes, there will be 2000 tuples of paired latitudes and longitudes, 
        where each latitude and longitude in a tuple can be accessed by the index of 0 and 1, respectively.
        You can only unzip a zipped tuple once before it is removed from the computer's memory.
          
          # unpack the lat_lngs zip object into a list
          # Add the latitudes and longitudes to a list.
          coordinates = list(lat_lngs)
          print(coordinates)
          
        With the list of random latitudes and longitues, we will use the coordinates in our lat_lngs tuple to find the nearest city suing
        Python's citypy module.
          # Use the citipy module to determine city based on latitude and longitude.
          from citipy import citipy
          # Create a list for holding the cities.
          cities = []
          # Identify the nearest city for each latitude and longitude combination.
          for coordinate in coordinates:
              city = citipy.nearest_city(coordinate[0], coordinate[1]).city_name

              # If the city is unique, then we will add it to the cities list.
              if city not in cities:
                  cities.append(city)
          # Print the city count to confirm sufficient count.
          len(cities)
          
        Now we need to import some more modules and call the openweathermap.org website
        
          # Import the datetime module from the datetime library.
          from datetime import datetime
          # Import the requests library.
          import requests
          # Import the API key.
          from config import weather_api_key
          # Starting URL for Weather Map API Call.
          url = "http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=" + weather_api_key
          print(url)
          
        Next, I wanted to see how the infromation received from the website would looke like, so I called information for Boston 
          # To see what the information looks like for a random city#
          import json
          city_url = url + "&q=" + "Boston"
          print(city_url)  
          # Create an endpoint URL for a city.To check what key's and values there are
          city_url = url + "&q=" + "Boston"
          city_weather = requests.get(city_url)
          city_weather
          city_weather.json()
          
        Now that I know what information I need to get from the website and how to get it, I loop through the cities in the list,
        and assign the data I need into a list
        
        # Loop through all the cities in the list.
          city_data = []
          # Print the beginning of the logging.
          print("Beginning Data Retrieval     ")
          print("-----------------------------")
          record_count = 1
          set_count = 1
          for i, city in enumerate(cities):

              # Group cities in sets of 50 for logging purposes.
              if (i % 50 == 0 and i >= 50):
                  set_count += 1
                  record_count = 1
              # Create endpoint URL with each city.
              city_url = url + "&q=" + city.replace(" ","+")

              # Log the URL, record, and set numbers and the city.
              print(f"Processing Record {record_count} of Set {set_count} | {city}")
              # Add 1 to the record count.
              record_count += 1
          # Run an API request for each of the cities.
              try:
                  # Parse the JSON and retrieve data.
                  city_weather = requests.get(city_url).json()
                  # Parse out the needed data.
                  city_lat = city_weather["coord"]["lat"]
                  city_lng = city_weather["coord"]["lon"]
                  city_max_temp = city_weather["main"]["temp_max"]
                  city_humidity = city_weather["main"]["humidity"]
                  city_clouds = city_weather["clouds"]["all"]
                  city_wind = city_weather["wind"]["speed"]
                  city_country = city_weather["sys"]["country"]
                  city_description=city_weather["weather"][0]["description"]
                  # Convert the date to ISO standard.
                  #city_date = datetime.utcfromtimestamp(city_weather["dt"]).strftime('%Y-%m-%d %H:%M:%S')
                  # Append the city information into city_data list.
                  city_data.append({"City": city.title(),
                                    "Lat": city_lat,
                                    "Lng": city_lng,
                                    "Max Temp": city_max_temp,
                                    "Humidity": city_humidity,
                                    "Cloudiness": city_clouds,
                                    "Wind Speed": city_wind,
                                    "Country": city_country,
                                    "Current Description": city_description})

          # If an error is experienced, skip the city.
              except:
                  print("City not found. Skipping...")
                  pass

          # Indicate that Data Loading is complete.
          print("-----------------------------")
          print("Data Retrieval Complete      ")
          print("-----------------------------")

        
        Next i will check again to make sure there are cities stored in the city-data list by counting the length of the list:
          # to check there are data stored in the city data
            len(city_data)
       
       Converting the list into a dataframe:
       # Convert the array of dictionaries to a Pandas DataFrame and printing the first 10 values to check the data:
          city_data_df = pd.DataFrame(city_data)
          city_data_df.head(10)

       
       
