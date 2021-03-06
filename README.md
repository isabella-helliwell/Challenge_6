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
  * Google Cloud Platform
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

            # 4b. Drop any empty rows and create a new DataFrame that doesn???t have empty rows.
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
            
            
             As seen in Output 3 there are some cities where hotels could not be allocated. For those rows, we will drop the rows.
            
            # 7. Drop the rows where there is no Hotel Name.
              #dropna() method
              import numpy as np
              import pandas as pd
              
              filter=hotel_df['Hotel Name']!=""
              clean_hotel_df=hotel_df[filter]
              clean_hotel_df.head()
            
            
            # 8a. Create the output File (CSV)
              output_data_file='CSV_Files/WeatherPy_vacation.csv'
              # 8b. Export the City_Data into a csv
              clean_hotel_df.to_csv(output_data_file, index_label="City_ID")

          Next we create a info box for all the cities with hotels, with the following information: City name, Country, weather condition, max temperature.
          
              # 9. Using the template add city name, the country code, the weather description and maximum temperature for the city.
              info_box_template = """
              <dl>
              <dt>City</dt><dd>{City}</dd>
              <dt>Country</dt><dd>{Country}</dd>
              <dt>Current Description</dt><dd>{Current Description}</dd>
              <dt>Max Temp</dt><dd>{Max Temp} ??F</dd>
              </dl>
              """

              # 10a. Get the data from each row and add it to the formatting template and store the data in a list.
              hotel_info = [info_box_template.format(**row) for index, row in clean_hotel_df.iterrows()]

              # 10b. Get the latitude and longitude from each row and store in a new DataFrame.
              locations = clean_hotel_df[["Lat", "Lng"]]          
          
              # 11a. Add a marker layer for each city to the map. 
              marker_layer = gmaps.marker_layer(locations, info_box_content=hotel_info)

              # 11b. Display the figure
              fig = gmaps.figure(center=(30.0, 31.0), zoom_level=2.05)
              fig.add_layer(marker_layer)
              fig
              
Output 4.WeatherPy_vacation_map.png
![image](https://user-images.githubusercontent.com/85843030/128057092-bd052990-9967-46d7-a37b-9e0aefb17f6b.png)



   ### 3.3 Create a Travel Itinerary Map
            Last part of the challenge is to create a route, using the WeatherPy_vacation.csv and show the route. For this
            we will use Google Directions. We will also create a marker layer map with a pop-up marker for each city on the
            itinerary.
            
            # Dependencies and Setup
            import pandas as pd
            import requests
            import gmaps
            import gmaps.datasets

            # Import API key
            from config2 import g_key

            # Configure gmaps
            gmaps.configure(api_key=g_key)

   
           # 1. Read the WeatherPy_vacation.csv into a DataFrame.
            vacation_df = pd.read_csv("Vacation_Search/CSV_Files/WeatherPy_vacation.csv")
            vacation_df.head()

           # 2. Using the template add the city name, the country code, the weather description and maximum temperature for the city.
            info_box_template = """
            <dl>
            <dt>City</dt><dd>{City}</dd>
            <dt>Country</dt><dd>{Country}</dd>
            <dt>Current Description</dt><dd>{Current Description}</dd>
            <dt>Max Temp</dt><dd>{Max Temp} ??F</dd>
            </dl>
            """

            # 3a. Get the data from each row and add it to the formatting template and store the data in a list.
            hotel_info = [info_box_template.format(**row) for index, row in vacation_df.iterrows()]

            # 3b. Get the latitude and longitude from each row and store in a new DataFrame.
            locations = vacation_df[["Lat", "Lng"]]
            
            # 4a. Add a marker layer for each city to the map.
            marker_layer = gmaps.marker_layer(locations, info_box_content=hotel_info)
            # 4b. Display the figure
            fig = gmaps.figure(center=(30.0, 31.0), zoom_level=2.05)
            fig.add_layer(marker_layer)
            fig
            
            # From the map above pick 4 cities and create a vacation itinerary route to travel between the four cities. 
            # 5. Create DataFrames for each city by filtering the 'vacation_df' using the loc method. 
            # Hint: The starting and ending city should be the same city.

            # starting city is Greeneville US
            #stop1 is Jasper US
            #stop2 is Boone, US
            #stop 3 is Hamilton US



            vacation_start = vacation_df.loc[(vacation_df["City"]=="Greeneville")]
            vacation_end = vacation_df.loc[(vacation_df["City"]=="Greeneville")]
            vacation_stop1 = vacation_df.loc[(vacation_df["City"]=="Jasper")]
            vacation_stop2 = vacation_df.loc[(vacation_df["City"]=="Boone")] 
            vacation_stop3 = vacation_df.loc[(vacation_df["City"]=="Hamilton")]
            
            # 6. Get the latitude-longitude pairs as tuples from each city DataFrame using the to_numpy function and list indexing.
              start = vacation_start[["Lat", "Lng"]].to_numpy()
              end = vacation_end[["Lat", "Lng"]].to_numpy()
              stop1 = vacation_stop1[['Lat', 'Lng']].to_numpy() 
              stop2 = vacation_stop2[['Lat','Lng']].to_numpy()
              stop3 = vacation_stop3[['Lat','Lng']].to_numpy()
              
              
            # 7. Create a direction layer map using the start and end latitude-longitude pairs,
            # and stop1, stop2, and stop3 as the waypoints. The travel_mode should be "DRIVING", "BICYCLING", or "WALKING".

            start_coord=(start[0,0], start[0,1])
            end_coord=start_coord
            stop1_coord=(stop1[0,0], stop1[0,1])
            stop2_coord=(stop2[0,0], stop2[0,1])
            stop3_coord=(stop3[0,0], stop3[0,1])
            
            fig = gmaps.figure()
            route = gmaps.directions_layer(start_coord, end_coord, waypoints=[stop1_coord, stop2_coord,stop3_coord], travel_mode='DRIVING')

            fig.add_layer(route)
            fig
        
Output5. WeatherPy_travel_map.png
![image](https://user-images.githubusercontent.com/85843030/128059662-1109e813-65ad-41c4-941d-fb7417deebee.png)

              # 8. To create a marker layer map between the four cities.
              #  Combine the four city DataFrames into one DataFrame using the concat() function.
              itinerary_df = pd.concat([vacation_start,vacation_end,vacation_stop1,vacation_stop2,vacation_stop3],ignore_index=True)
              itinerary_df


Output.6
![image](https://user-images.githubusercontent.com/85843030/128060153-e9a12c21-21cb-4ed3-b27b-d67c1bc3cf5b.png)


              # 9 Using the template add city name, the country code, the weather description and maximum temperature for the city. 
              info_box_template = """
              <dl>
              <dt>Hotel Name</dt><dd>{Hotel Name}</dd>
              <dt>City</dt><dd>{City}</dd>
              <dt>Country</dt><dd>{Country}</dd>
              <dt>Current Description</dt><dd>{Current Description}</dd>
              <dt>Max Temp</dt><dd>{Max Temp} ??F</dd>
              </dl>
              """

              # 10a Get the data from each row and add it to the formatting template and store the data in a list.
              hotel_info = [info_box_template.format(**row) for index, row in itinerary_df.iterrows()]

              # 10b. Get the latitude and longitude from each row and store in a new DataFrame.
              locations = itinerary_df[["Lat", "Lng"]]

              # 11a. Add a marker layer for each city to the map.
              marker_layer = gmaps.marker_layer(locations, info_box_content=hotel_info)
              # 11b. Display the figure
              fig = gmaps.figure(center=(41, -87.0), zoom_level=6)
              fig.add_layer(marker_layer)
              fig




Output7. WeatherPy_travel_map_markers.png
![image](https://user-images.githubusercontent.com/85843030/128059890-a0fb8282-fc11-4bdf-8bfa-651277002cfa.png)



            
