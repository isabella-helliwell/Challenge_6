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
        
          
          
