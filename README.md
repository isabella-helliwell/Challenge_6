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

        
        Next i will check again to make sure there are cities stored in the city-data list by counting the length of the array:
          # to check there are data stored in the city data
            len(city_data)
       
       Converting the list into a dataframe:
       # Convert the array of dictionaries to a Pandas DataFrame and printing the first 10 values to check the data:
          city_data_df = pd.DataFrame(city_data)
          city_data_df.head(10)

       Lastely we need to re order the columns and save the file 
          new_column_order=['City', 'Country','Lat','Lng','Max Temp', 'Humidity', 'Cloudiness', 'Wind Speed', 'Current Description']
          city_data_df=city_data_df[new_column_order]
          # Create the output file (CSV).
          output_data_file = "Weather_Database/WeatherPy_Database.csv"
          # Export the City_Data into a CSV.
          city_data_df.to_csv(output_data_file, index_label="City_ID")
          
  output1. ![image](https://user-images.githubusercontent.com/85843030/128047924-9934d8df-b45c-4364-b668-43939fb70439.png)
  
  ### 3.2 Create a Customer Travel Destination Map
          The aim here, is to use input statements to retrieve customer weather preferences, and use those prefrences as input to
          identify ptential travel destinations and nearby hotels.
          First we import the dependencies and import the "WeatherPy_Database.csv" file that we created in section 3.1
          Then, we will ask the user to input some temperature ranges that they want, like min and max temperature.
          This input is used to filter the dataframe for the temperature ranges. In this example, we used 80-90 F.
          
            # Dependencies and Setup
            import pandas as pd
            import requests
            import gmaps
            # Import API key
            from config1 import g_key
            # Configure gmaps
            gmaps.configure(api_key=g_key)

            # 1. Import the WeatherPy_database.csv file. 
            city_data_df = pd.read_csv("../Weather_Database/WeatherPy_Database.csv")
            city_data_df.head()
            
            # 2. Prompt the user to enter minimum and maximum temperature criteria..
            min_temp = float(input("What is the minimum temperature you would like for your vacation? "))
            max_temp = float(input("What is the maximum temperature you would like for your vacation? "))


            # 3. Filter the city_data_df DataFrame to find the cities that fit the criteria using the loc method.
            preferred_cities_df = city_data_df.loc[(city_data_df["Max Temp"] <= max_temp) & (city_data_df["Max Temp"] >= min_temp)]
            preferred_cities_df.head(10)


Output 2.
![image](https://user-images.githubusercontent.com/85843030/128054673-343d0bec-af4a-428e-8c4e-96a425e2efdf.png)



            # 4a. Determine if there are any empty rows.
            preferred_cities_df.count()

            # 4b. Drop any empty rows and create a new DataFrame that doesn’t have empty rows.
            clean_travel_cities = preferred_cities_df.dropna()
            #clean_travel_cities.head()
            clean_travel_cities.count()
            #clean_travel_cities.columns
            
            
            # 5a. Create DataFrame called hotel_df to store hotel names along with city, country, max temp, and coordinates.
            hotel_df = clean_travel_cities[["City", "Country", "Max Temp", "Current Description", "Lat", "Lng"]].copy()
            # 5b. Create a new column "Hotel Name".
            hotel_df["Hotel Name"] = ""
            hotel_df.head(10)
            hotel_df.count()
          
          Next, is to create a foor loop that goes trough the hotel_df dataframe, and populates hotels in the hotels column that was
          created in step 5b
          Becasue i encountered many time out error when I was running this code, error's like timeouts. I decided to make the temperature range 80-90 instead
          of 75-90 originally intended, and I also put a timer.sleep to delay the request between each call. These solutions, seem to work and I managed 
          to run the for loop successfully.
          
           import time
            params = {
               "radius": 5000,
               "type": "lodging",
               "key": g_key
            }
            # 6b. Iterate through the hotel DataFrame
            for index, row in hotel_df.iterrows():
               # get lat, lng from df
                lat = row["Lat"]
                lng = row["Lng"]
               # 6c. Get latitude and longitude from DataFrame.
                params["location"] = f"{lat},{lng}"
               # 6d. Set up the base URL for the Google Directions API to get JSON data.
                base_url = "https://maps.googleapis.com/maps/api/place/nearbysearch/json"
               # 6e. Make request and retrieve the JSON data from the search.
                time.sleep(1)
                hotels = requests.get(base_url, params=params).json()

               # 6f. Get the first hotel from the results and store the name, if a hotel isn't found skip the city.
                try:
                    hotel_df.loc[index, "Hotel Name"] = hotels["results"][0]["name"]
                    #print(index)
                except:
                     print("Hotel not found... skipping.")
          
            
Output 3. 
![image](https://user-images.githubusercontent.com/85843030/128056163-42c27628-9c54-4ba1-8e2e-31ad004660a4.png)
            
            
            
            
            
            
            
