
# Analysis


```python
# 0/ It was found necessary to first plot an additional graph (long vs lat) to locate the cities and better understand if the random selection
# was accounting for most of the surface of the earth, which seems to be the case.

# 1/ Most of the data are located on the right part of the plots, on the positive axis which correlates directly
# with the amount of apparent and emerged land area on earth suggesting that there will be a higher number of weather stations on the north hemisphere.
# This note is important to explain with the data can potentially be skewed.

# 2/ Most of the highest temperature peak at the equator (latitude 0) level following a bell curve shape. The equatorial cities are more exposed to the sun.

# 3/ However City temperature and Humidity don't seem to have a strong correlation.

# 4/ Most of the wind speed concentration is located between 0 to 10 mph and no strong correlation exist
# with higher wind speeds closer to the equator.
```

# Cities List generation


```python
# Import dependencies
import numpy as no
import pandas as pd
import matplotlib.pyplot as plt
import seaborn as sns
from citipy import citipy
import random
import requests
from config import api_key
import time
```


```python
# List of latitudes and longitudes
latitudes = range (-90,90)
longitudes = range (-180,180)
city_list = []

for lat in latitudes:
    for lng in longitudes:
        city = citipy.nearest_city(lat, lng)
        city_name = city.city_name
        city_list.append(city_name)

#remove duplicates
city_dataframe=pd.DataFrame(city_list)
clean_city_dataframe = city_dataframe.drop_duplicates()

clean_city_dataframe.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>0</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>vaini</td>
    </tr>
    <tr>
      <th>12</th>
      <td>mataura</td>
    </tr>
    <tr>
      <th>39</th>
      <td>rikitea</td>
    </tr>
    <tr>
      <th>53</th>
      <td>punta arenas</td>
    </tr>
    <tr>
      <th>88</th>
      <td>ushuaia</td>
    </tr>
  </tbody>
</table>
</div>




```python
clean_city_dataframe.count()
```




    0    7957
    dtype: int64




```python
#generate cities sample
cities_sample = clean_city_dataframe.sample(600)
#generate the cities index from 1 to 600
cities_sample = cities_sample.reset_index(drop = True)
#rename the cities column by "City"
cities_sample.columns = ["City"]
#start the index from 1
cities_sample.index+= 1
#print the sampled cities
cities_sample.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>City</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>1</th>
      <td>kiunga</td>
    </tr>
    <tr>
      <th>2</th>
      <td>xinqing</td>
    </tr>
    <tr>
      <th>3</th>
      <td>orlovskiy</td>
    </tr>
    <tr>
      <th>4</th>
      <td>maldonado</td>
    </tr>
    <tr>
      <th>5</th>
      <td>farrukhnagar</td>
    </tr>
  </tbody>
</table>
</div>




```python
#create the additional columns to hold the informations
cities_sample["Lat"]=""
cities_sample["Lng"]=""
cities_sample["Country"]=""
cities_sample["Date"]=""
cities_sample["Max temp"]=""
cities_sample["Humidity"]=""
cities_sample["Cloudiness"]=""
cities_sample["Wind speed"]=""
cities_sample.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>City</th>
      <th>Lat</th>
      <th>Lng</th>
      <th>Country</th>
      <th>Date</th>
      <th>Max temp</th>
      <th>Humidity</th>
      <th>Cloudiness</th>
      <th>Wind speed</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>1</th>
      <td>kiunga</td>
      <td></td>
      <td></td>
      <td></td>
      <td></td>
      <td></td>
      <td></td>
      <td></td>
      <td></td>
    </tr>
    <tr>
      <th>2</th>
      <td>xinqing</td>
      <td></td>
      <td></td>
      <td></td>
      <td></td>
      <td></td>
      <td></td>
      <td></td>
      <td></td>
    </tr>
    <tr>
      <th>3</th>
      <td>orlovskiy</td>
      <td></td>
      <td></td>
      <td></td>
      <td></td>
      <td></td>
      <td></td>
      <td></td>
      <td></td>
    </tr>
    <tr>
      <th>4</th>
      <td>maldonado</td>
      <td></td>
      <td></td>
      <td></td>
      <td></td>
      <td></td>
      <td></td>
      <td></td>
      <td></td>
    </tr>
    <tr>
      <th>5</th>
      <td>farrukhnagar</td>
      <td></td>
      <td></td>
      <td></td>
      <td></td>
      <td></td>
      <td></td>
      <td></td>
      <td></td>
    </tr>
  </tbody>
</table>
</div>



# Retrieve data from OpenWeatherMap Api


```python
print("Data retrieval beginning")
print("------------------------")

#Open weather API key
api_key = "25bc90a1196e6f153eece0bc0b0fc9eb"
#define the units as imperial to retrieve temperature Farenheit and wind speed miles per hour
units = "Imperial"
#display url
url = "http://api.openweathermap.org/data/2.5/weather?"

for index,row in cities_sample.iterrows():
    city_name=row["City"]
    city_name_url= city_name.replace("", "%20")
    #query url
    query_url = url + "appid=" + api_key + "&units=" + units + "&q=" + row["City"]
    # Get weather data
    weather_response = requests.get(query_url)
    weather_json = weather_response.json()
#     print(weather_json)
    cities_sample.set_value(index,"Lat", weather_json.get("coord",{}).get("lat"))
    cities_sample.set_value(index,"Lng", weather_json.get("coord",{}).get("lon"))
    cities_sample.set_value(index,"Country", weather_json.get("sys",{}).get("country"))
    cities_sample.set_value(index,"Date", weather_json.get("dt",{}))
    cities_sample.set_value(index,"Max temp", weather_json.get("main",{}).get("temp_max"))
    cities_sample.set_value(index,"Humidity", weather_json.get("main",{}).get("humidity"))
    cities_sample.set_value(index,"Cloudiness", weather_json.get("clouds",{}).get("all"))
    cities_sample.set_value(index,"Wind speed", weather_json.get("wind",{}).get("speed"))   
    print("Processing city #" + str(index) + " out of 600, name: " + str(city_name))
    query_url
    print(query_url)

print("------------------------")
print("Data retrieval done")

# Save config information

```

    Data retrieval beginning
    ------------------------
    

    C:\ProgramData\Anaconda3\envs\PythonData\lib\site-packages\ipykernel\__main__.py:20: FutureWarning: set_value is deprecated and will be removed in a future release. Please use .at[] or .iat[] accessors instead
    C:\ProgramData\Anaconda3\envs\PythonData\lib\site-packages\ipykernel\__main__.py:21: FutureWarning: set_value is deprecated and will be removed in a future release. Please use .at[] or .iat[] accessors instead
    C:\ProgramData\Anaconda3\envs\PythonData\lib\site-packages\ipykernel\__main__.py:22: FutureWarning: set_value is deprecated and will be removed in a future release. Please use .at[] or .iat[] accessors instead
    C:\ProgramData\Anaconda3\envs\PythonData\lib\site-packages\ipykernel\__main__.py:23: FutureWarning: set_value is deprecated and will be removed in a future release. Please use .at[] or .iat[] accessors instead
    C:\ProgramData\Anaconda3\envs\PythonData\lib\site-packages\ipykernel\__main__.py:24: FutureWarning: set_value is deprecated and will be removed in a future release. Please use .at[] or .iat[] accessors instead
    C:\ProgramData\Anaconda3\envs\PythonData\lib\site-packages\ipykernel\__main__.py:25: FutureWarning: set_value is deprecated and will be removed in a future release. Please use .at[] or .iat[] accessors instead
    C:\ProgramData\Anaconda3\envs\PythonData\lib\site-packages\ipykernel\__main__.py:26: FutureWarning: set_value is deprecated and will be removed in a future release. Please use .at[] or .iat[] accessors instead
    C:\ProgramData\Anaconda3\envs\PythonData\lib\site-packages\ipykernel\__main__.py:27: FutureWarning: set_value is deprecated and will be removed in a future release. Please use .at[] or .iat[] accessors instead
    

    Processing city #1 out of 600, name: kiunga
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=kiunga
    Processing city #2 out of 600, name: xinqing
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=xinqing
    Processing city #3 out of 600, name: orlovskiy
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=orlovskiy
    Processing city #4 out of 600, name: maldonado
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=maldonado
    Processing city #5 out of 600, name: farrukhnagar
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=farrukhnagar
    Processing city #6 out of 600, name: kirkland lake
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=kirkland lake
    Processing city #7 out of 600, name: cockburn town
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=cockburn town
    Processing city #8 out of 600, name: abu kamal
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=abu kamal
    Processing city #9 out of 600, name: paso de los toros
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=paso de los toros
    Processing city #10 out of 600, name: aliaga
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=aliaga
    Processing city #11 out of 600, name: north bend
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=north bend
    Processing city #12 out of 600, name: ajaccio
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=ajaccio
    Processing city #13 out of 600, name: balezino
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=balezino
    Processing city #14 out of 600, name: sasaram
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=sasaram
    Processing city #15 out of 600, name: lidkoping
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=lidkoping
    Processing city #16 out of 600, name: abeche
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=abeche
    Processing city #17 out of 600, name: along
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=along
    Processing city #18 out of 600, name: asadabad
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=asadabad
    Processing city #19 out of 600, name: valley
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=valley
    Processing city #20 out of 600, name: la palma
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=la palma
    Processing city #21 out of 600, name: torbat-e jam
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=torbat-e jam
    Processing city #22 out of 600, name: evensk
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=evensk
    Processing city #23 out of 600, name: espinosa
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=espinosa
    Processing city #24 out of 600, name: urulga
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=urulga
    Processing city #25 out of 600, name: barahan
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=barahan
    Processing city #26 out of 600, name: sao jose da coroa grande
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=sao jose da coroa grande
    Processing city #27 out of 600, name: kalavrita
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=kalavrita
    Processing city #28 out of 600, name: kamphaeng phet
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=kamphaeng phet
    Processing city #29 out of 600, name: nkongsamba
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=nkongsamba
    Processing city #30 out of 600, name: onokhoy
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=onokhoy
    Processing city #31 out of 600, name: alihe
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=alihe
    Processing city #32 out of 600, name: jinan
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=jinan
    Processing city #33 out of 600, name: kargasok
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=kargasok
    Processing city #34 out of 600, name: shihezi
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=shihezi
    Processing city #35 out of 600, name: cullman
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=cullman
    Processing city #36 out of 600, name: lapua
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=lapua
    Processing city #37 out of 600, name: madingou
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=madingou
    Processing city #38 out of 600, name: stupino
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=stupino
    Processing city #39 out of 600, name: zhigalovo
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=zhigalovo
    Processing city #40 out of 600, name: bochil
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=bochil
    Processing city #41 out of 600, name: shangrao
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=shangrao
    Processing city #42 out of 600, name: price
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=price
    Processing city #43 out of 600, name: gonbad-e qabus
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=gonbad-e qabus
    Processing city #44 out of 600, name: blagoveshchensk
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=blagoveshchensk
    Processing city #45 out of 600, name: williamsburg
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=williamsburg
    Processing city #46 out of 600, name: yertsevo
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=yertsevo
    Processing city #47 out of 600, name: sennoy
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=sennoy
    Processing city #48 out of 600, name: navrongo
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=navrongo
    Processing city #49 out of 600, name: chitungwiza
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=chitungwiza
    Processing city #50 out of 600, name: laiagam
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=laiagam
    Processing city #51 out of 600, name: bobo dioulasso
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=bobo dioulasso
    Processing city #52 out of 600, name: samarai
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=samarai
    Processing city #53 out of 600, name: kopavogur
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=kopavogur
    Processing city #54 out of 600, name: kasempa
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=kasempa
    Processing city #55 out of 600, name: iwaki
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=iwaki
    Processing city #56 out of 600, name: visby
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=visby
    Processing city #57 out of 600, name: jiangyou
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=jiangyou
    Processing city #58 out of 600, name: rosita
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=rosita
    Processing city #59 out of 600, name: arcachon
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=arcachon
    Processing city #60 out of 600, name: jiayuguan
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=jiayuguan
    Processing city #61 out of 600, name: jamame
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=jamame
    Processing city #62 out of 600, name: kaduqli
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=kaduqli
    Processing city #63 out of 600, name: timbiqui
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=timbiqui
    Processing city #64 out of 600, name: ajoloapan
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=ajoloapan
    Processing city #65 out of 600, name: agogo
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=agogo
    Processing city #66 out of 600, name: katra
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=katra
    Processing city #67 out of 600, name: gweta
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=gweta
    Processing city #68 out of 600, name: sibut
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=sibut
    Processing city #69 out of 600, name: sukhoy log
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=sukhoy log
    Processing city #70 out of 600, name: horasan
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=horasan
    Processing city #71 out of 600, name: yauya
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=yauya
    Processing city #72 out of 600, name: nome
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=nome
    Processing city #73 out of 600, name: mujiayingzi
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=mujiayingzi
    Processing city #74 out of 600, name: tatawin
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=tatawin
    Processing city #75 out of 600, name: junin
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=junin
    Processing city #76 out of 600, name: pointe michel
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=pointe michel
    Processing city #77 out of 600, name: kuandian
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=kuandian
    Processing city #78 out of 600, name: mushie
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=mushie
    Processing city #79 out of 600, name: vitorino freire
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=vitorino freire
    Processing city #80 out of 600, name: tierralta
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=tierralta
    Processing city #81 out of 600, name: ozark
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=ozark
    Processing city #82 out of 600, name: morehead
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=morehead
    Processing city #83 out of 600, name: sonoita
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=sonoita
    Processing city #84 out of 600, name: inuvik
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=inuvik
    Processing city #85 out of 600, name: mugumu
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=mugumu
    Processing city #86 out of 600, name: omboue
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=omboue
    Processing city #87 out of 600, name: narsaq
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=narsaq
    Processing city #88 out of 600, name: gulfport
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=gulfport
    Processing city #89 out of 600, name: northam
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=northam
    Processing city #90 out of 600, name: karatuzskoye
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=karatuzskoye
    Processing city #91 out of 600, name: shchelyayur
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=shchelyayur
    Processing city #92 out of 600, name: leh
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=leh
    Processing city #93 out of 600, name: kolpashevo
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=kolpashevo
    Processing city #94 out of 600, name: masumbwe
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=masumbwe
    Processing city #95 out of 600, name: capinota
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=capinota
    Processing city #96 out of 600, name: novyy nekouz
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=novyy nekouz
    Processing city #97 out of 600, name: taguatinga
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=taguatinga
    Processing city #98 out of 600, name: cutro
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=cutro
    Processing city #99 out of 600, name: saint-pierre
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=saint-pierre
    Processing city #100 out of 600, name: acapetahua
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=acapetahua
    Processing city #101 out of 600, name: alamos
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=alamos
    Processing city #102 out of 600, name: bisert
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=bisert
    Processing city #103 out of 600, name: palanga
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=palanga
    Processing city #104 out of 600, name: vikulovo
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=vikulovo
    Processing city #105 out of 600, name: eureka
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=eureka
    Processing city #106 out of 600, name: purpe
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=purpe
    Processing city #107 out of 600, name: inyonga
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=inyonga
    Processing city #108 out of 600, name: sentyabrskiy
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=sentyabrskiy
    Processing city #109 out of 600, name: asau
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=asau
    Processing city #110 out of 600, name: osuna
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=osuna
    Processing city #111 out of 600, name: wangou
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=wangou
    Processing city #112 out of 600, name: were ilu
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=were ilu
    Processing city #113 out of 600, name: makakilo city
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=makakilo city
    Processing city #114 out of 600, name: manaia
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=manaia
    Processing city #115 out of 600, name: pszczyna
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=pszczyna
    Processing city #116 out of 600, name: gonda
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=gonda
    Processing city #117 out of 600, name: gaspar
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=gaspar
    Processing city #118 out of 600, name: kayes
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=kayes
    Processing city #119 out of 600, name: blensong
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=blensong
    Processing city #120 out of 600, name: owatonna
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=owatonna
    Processing city #121 out of 600, name: gurgan
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=gurgan
    Processing city #122 out of 600, name: lander
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=lander
    Processing city #123 out of 600, name: sofiysk
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=sofiysk
    Processing city #124 out of 600, name: tonneins
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=tonneins
    Processing city #125 out of 600, name: enumclaw
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=enumclaw
    Processing city #126 out of 600, name: genhe
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=genhe
    Processing city #127 out of 600, name: ryotsu
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=ryotsu
    Processing city #128 out of 600, name: mbacke
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=mbacke
    Processing city #129 out of 600, name: yurimaguas
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=yurimaguas
    Processing city #130 out of 600, name: jarjis
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=jarjis
    Processing city #131 out of 600, name: matam
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=matam
    Processing city #132 out of 600, name: tucano
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=tucano
    Processing city #133 out of 600, name: tiruchchendur
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=tiruchchendur
    Processing city #134 out of 600, name: san jose de guanipa
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=san jose de guanipa
    Processing city #135 out of 600, name: bo
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=bo
    Processing city #136 out of 600, name: caconda
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=caconda
    Processing city #137 out of 600, name: deep river
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=deep river
    Processing city #138 out of 600, name: najran
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=najran
    Processing city #139 out of 600, name: wenzhou
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=wenzhou
    Processing city #140 out of 600, name: lichinga
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=lichinga
    Processing city #141 out of 600, name: cienfuegos
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=cienfuegos
    Processing city #142 out of 600, name: irece
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=irece
    Processing city #143 out of 600, name: ishim
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=ishim
    Processing city #144 out of 600, name: gizo
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=gizo
    Processing city #145 out of 600, name: plaridel
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=plaridel
    Processing city #146 out of 600, name: lufkin
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=lufkin
    Processing city #147 out of 600, name: ulu-telyak
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=ulu-telyak
    Processing city #148 out of 600, name: toba
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=toba
    Processing city #149 out of 600, name: praxedis guerrero
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=praxedis guerrero
    Processing city #150 out of 600, name: bolshaya chernigovka
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=bolshaya chernigovka
    Processing city #151 out of 600, name: borgarnes
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=borgarnes
    Processing city #152 out of 600, name: smidovich
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=smidovich
    Processing city #153 out of 600, name: dzhebariki-khaya
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=dzhebariki-khaya
    Processing city #154 out of 600, name: balikpapan
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=balikpapan
    Processing city #155 out of 600, name: te anau
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=te anau
    Processing city #156 out of 600, name: udachnyy
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=udachnyy
    Processing city #157 out of 600, name: phuntsholing
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=phuntsholing
    Processing city #158 out of 600, name: maun
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=maun
    Processing city #159 out of 600, name: sao francisco
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=sao francisco
    Processing city #160 out of 600, name: regen
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=regen
    Processing city #161 out of 600, name: cidreira
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=cidreira
    Processing city #162 out of 600, name: camaragibe
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=camaragibe
    Processing city #163 out of 600, name: agdam
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=agdam
    Processing city #164 out of 600, name: tulua
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=tulua
    Processing city #165 out of 600, name: neftcala
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=neftcala
    Processing city #166 out of 600, name: trincomalee
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=trincomalee
    Processing city #167 out of 600, name: boke
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=boke
    Processing city #168 out of 600, name: cayhagan
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=cayhagan
    Processing city #169 out of 600, name: novikovo
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=novikovo
    Processing city #170 out of 600, name: nalgonda
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=nalgonda
    Processing city #171 out of 600, name: aripuana
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=aripuana
    Processing city #172 out of 600, name: noblesville
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=noblesville
    Processing city #173 out of 600, name: bjelovar
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=bjelovar
    Processing city #174 out of 600, name: shakiso
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=shakiso
    Processing city #175 out of 600, name: bloemfontein
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=bloemfontein
    Processing city #176 out of 600, name: barentu
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=barentu
    Processing city #177 out of 600, name: kolda
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=kolda
    Processing city #178 out of 600, name: vrangel
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=vrangel
    Processing city #179 out of 600, name: devils lake
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=devils lake
    Processing city #180 out of 600, name: staryy saltiv
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=staryy saltiv
    Processing city #181 out of 600, name: outjo
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=outjo
    Processing city #182 out of 600, name: muzhi
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=muzhi
    Processing city #183 out of 600, name: tarakan
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=tarakan
    Processing city #184 out of 600, name: areia branca
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=areia branca
    Processing city #185 out of 600, name: kaduy
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=kaduy
    Processing city #186 out of 600, name: talesh
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=talesh
    Processing city #187 out of 600, name: jinchengjiang
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=jinchengjiang
    Processing city #188 out of 600, name: maturin
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=maturin
    Processing city #189 out of 600, name: caravelas
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=caravelas
    Processing city #190 out of 600, name: conde
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=conde
    Processing city #191 out of 600, name: armidale
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=armidale
    Processing city #192 out of 600, name: mundgod
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=mundgod
    Processing city #193 out of 600, name: suzun
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=suzun
    Processing city #194 out of 600, name: yuncheng
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=yuncheng
    Processing city #195 out of 600, name: cagayan de tawi-tawi
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=cagayan de tawi-tawi
    Processing city #196 out of 600, name: valdobbiadene
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=valdobbiadene
    Processing city #197 out of 600, name: lakhdenpokhya
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=lakhdenpokhya
    Processing city #198 out of 600, name: montes altos
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=montes altos
    Processing city #199 out of 600, name: xiongyue
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=xiongyue
    Processing city #200 out of 600, name: montego bay
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=montego bay
    Processing city #201 out of 600, name: bossangoa
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=bossangoa
    Processing city #202 out of 600, name: bakel
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=bakel
    Processing city #203 out of 600, name: shirokiy
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=shirokiy
    Processing city #204 out of 600, name: bad wurzach
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=bad wurzach
    Processing city #205 out of 600, name: soure
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=soure
    Processing city #206 out of 600, name: makurdi
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=makurdi
    Processing city #207 out of 600, name: bolshiye klyuchishchi
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=bolshiye klyuchishchi
    Processing city #208 out of 600, name: thika
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=thika
    Processing city #209 out of 600, name: choucheng
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=choucheng
    Processing city #210 out of 600, name: estelle
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=estelle
    Processing city #211 out of 600, name: porto velho
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=porto velho
    Processing city #212 out of 600, name: lubango
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=lubango
    Processing city #213 out of 600, name: gualeguay
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=gualeguay
    Processing city #214 out of 600, name: moroni
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=moroni
    Processing city #215 out of 600, name: yarmouth
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=yarmouth
    Processing city #216 out of 600, name: bandarbeyla
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=bandarbeyla
    Processing city #217 out of 600, name: ghauspur
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=ghauspur
    Processing city #218 out of 600, name: kirovsk
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=kirovsk
    Processing city #219 out of 600, name: tupancireta
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=tupancireta
    Processing city #220 out of 600, name: bang len
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=bang len
    Processing city #221 out of 600, name: nivala
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=nivala
    Processing city #222 out of 600, name: barroquinha
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=barroquinha
    Processing city #223 out of 600, name: springdale
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=springdale
    Processing city #224 out of 600, name: tyup
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=tyup
    Processing city #225 out of 600, name: kiruna
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=kiruna
    Processing city #226 out of 600, name: sinaloa
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=sinaloa
    Processing city #227 out of 600, name: ust-tsilma
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=ust-tsilma
    Processing city #228 out of 600, name: chapleau
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=chapleau
    Processing city #229 out of 600, name: gornoye loo
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=gornoye loo
    Processing city #230 out of 600, name: aloleng
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=aloleng
    Processing city #231 out of 600, name: forio
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=forio
    Processing city #232 out of 600, name: sedelnikovo
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=sedelnikovo
    Processing city #233 out of 600, name: key largo
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=key largo
    Processing city #234 out of 600, name: pilar
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=pilar
    Processing city #235 out of 600, name: chakwal
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=chakwal
    Processing city #236 out of 600, name: challans
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=challans
    Processing city #237 out of 600, name: la baneza
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=la baneza
    Processing city #238 out of 600, name: werneck
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=werneck
    Processing city #239 out of 600, name: udimskiy
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=udimskiy
    Processing city #240 out of 600, name: nagai
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=nagai
    Processing city #241 out of 600, name: baneh
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=baneh
    Processing city #242 out of 600, name: zelenoborsk
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=zelenoborsk
    Processing city #243 out of 600, name: arrecife
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=arrecife
    Processing city #244 out of 600, name: akkermanovka
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=akkermanovka
    Processing city #245 out of 600, name: norman wells
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=norman wells
    Processing city #246 out of 600, name: kannauj
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=kannauj
    Processing city #247 out of 600, name: karatau
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=karatau
    Processing city #248 out of 600, name: maxixe
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=maxixe
    Processing city #249 out of 600, name: osa
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=osa
    Processing city #250 out of 600, name: pirgos
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=pirgos
    Processing city #251 out of 600, name: kirchzarten
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=kirchzarten
    Processing city #252 out of 600, name: ahumada
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=ahumada
    Processing city #253 out of 600, name: haflong
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=haflong
    Processing city #254 out of 600, name: kotido
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=kotido
    Processing city #255 out of 600, name: frederiksvaerk
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=frederiksvaerk
    Processing city #256 out of 600, name: sidi bu zayd
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=sidi bu zayd
    Processing city #257 out of 600, name: endicott
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=endicott
    Processing city #258 out of 600, name: ruidoso
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=ruidoso
    Processing city #259 out of 600, name: shimanovsk
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=shimanovsk
    Processing city #260 out of 600, name: sao joao do paraiso
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=sao joao do paraiso
    Processing city #261 out of 600, name: pervomayskoye
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=pervomayskoye
    Processing city #262 out of 600, name: lilongwe
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=lilongwe
    Processing city #263 out of 600, name: putyatino
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=putyatino
    Processing city #264 out of 600, name: boulder
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=boulder
    Processing city #265 out of 600, name: lugoba
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=lugoba
    Processing city #266 out of 600, name: basco
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=basco
    Processing city #267 out of 600, name: fethiye
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=fethiye
    Processing city #268 out of 600, name: zhuanghe
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=zhuanghe
    Processing city #269 out of 600, name: sumbe
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=sumbe
    Processing city #270 out of 600, name: salalah
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=salalah
    Processing city #271 out of 600, name: nova olinda do norte
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=nova olinda do norte
    Processing city #272 out of 600, name: middelburg
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=middelburg
    Processing city #273 out of 600, name: karamken
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=karamken
    Processing city #274 out of 600, name: menongue
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=menongue
    Processing city #275 out of 600, name: formosa do rio preto
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=formosa do rio preto
    Processing city #276 out of 600, name: nazarovo
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=nazarovo
    Processing city #277 out of 600, name: buloh kasap
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=buloh kasap
    Processing city #278 out of 600, name: kamyshla
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=kamyshla
    Processing city #279 out of 600, name: ajdabiya
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=ajdabiya
    Processing city #280 out of 600, name: kargopol
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=kargopol
    Processing city #281 out of 600, name: swellendam
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=swellendam
    Processing city #282 out of 600, name: tingi
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=tingi
    Processing city #283 out of 600, name: mongo
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=mongo
    Processing city #284 out of 600, name: petukhovo
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=petukhovo
    Processing city #285 out of 600, name: dapdap
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=dapdap
    Processing city #286 out of 600, name: lyskovo
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=lyskovo
    Processing city #287 out of 600, name: paidha
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=paidha
    Processing city #288 out of 600, name: hailey
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=hailey
    Processing city #289 out of 600, name: chiusi
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=chiusi
    Processing city #290 out of 600, name: mercedes
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=mercedes
    Processing city #291 out of 600, name: moreira sales
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=moreira sales
    Processing city #292 out of 600, name: chissamba
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=chissamba
    Processing city #293 out of 600, name: mocajuba
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=mocajuba
    Processing city #294 out of 600, name: odienne
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=odienne
    Processing city #295 out of 600, name: tuy hoa
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=tuy hoa
    Processing city #296 out of 600, name: morwa
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=morwa
    Processing city #297 out of 600, name: sucua
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=sucua
    Processing city #298 out of 600, name: tanout
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=tanout
    Processing city #299 out of 600, name: atikokan
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=atikokan
    Processing city #300 out of 600, name: zyryanskoye
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=zyryanskoye
    Processing city #301 out of 600, name: mar del plata
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=mar del plata
    Processing city #302 out of 600, name: omaruru
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=omaruru
    Processing city #303 out of 600, name: glenwood springs
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=glenwood springs
    Processing city #304 out of 600, name: mahbubabad
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=mahbubabad
    Processing city #305 out of 600, name: borujan
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=borujan
    Processing city #306 out of 600, name: verkhnyaya inta
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=verkhnyaya inta
    Processing city #307 out of 600, name: sechenovo
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=sechenovo
    Processing city #308 out of 600, name: barawe
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=barawe
    Processing city #309 out of 600, name: mirnyy
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=mirnyy
    Processing city #310 out of 600, name: godinesti
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=godinesti
    Processing city #311 out of 600, name: kendari
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=kendari
    Processing city #312 out of 600, name: ourossogui
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=ourossogui
    Processing city #313 out of 600, name: buta
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=buta
    Processing city #314 out of 600, name: antofagasta
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=antofagasta
    Processing city #315 out of 600, name: boshnyakovo
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=boshnyakovo
    Processing city #316 out of 600, name: shevchenkove
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=shevchenkove
    Processing city #317 out of 600, name: domoni
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=domoni
    Processing city #318 out of 600, name: zavetnoye
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=zavetnoye
    Processing city #319 out of 600, name: bogatynia
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=bogatynia
    Processing city #320 out of 600, name: rio tercero
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=rio tercero
    Processing city #321 out of 600, name: tocopilla
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=tocopilla
    Processing city #322 out of 600, name: rudbar
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=rudbar
    Processing city #323 out of 600, name: tungor
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=tungor
    Processing city #324 out of 600, name: chicontepec
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=chicontepec
    Processing city #325 out of 600, name: bacolod
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=bacolod
    Processing city #326 out of 600, name: khor
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=khor
    Processing city #327 out of 600, name: mucurapo
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=mucurapo
    Processing city #328 out of 600, name: louga
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=louga
    Processing city #329 out of 600, name: baraki barak
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=baraki barak
    Processing city #330 out of 600, name: opobo
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=opobo
    Processing city #331 out of 600, name: nobres
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=nobres
    Processing city #332 out of 600, name: maniwaki
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=maniwaki
    Processing city #333 out of 600, name: makaryev
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=makaryev
    Processing city #334 out of 600, name: yondo
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=yondo
    Processing city #335 out of 600, name: crixas
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=crixas
    Processing city #336 out of 600, name: henties bay
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=henties bay
    Processing city #337 out of 600, name: datong
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=datong
    Processing city #338 out of 600, name: nyrob
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=nyrob
    Processing city #339 out of 600, name: cocobeach
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=cocobeach
    Processing city #340 out of 600, name: ocosingo
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=ocosingo
    Processing city #341 out of 600, name: bolshoy uluy
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=bolshoy uluy
    Processing city #342 out of 600, name: ugoofaaru
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=ugoofaaru
    Processing city #343 out of 600, name: qorveh
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=qorveh
    Processing city #344 out of 600, name: placido de castro
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=placido de castro
    Processing city #345 out of 600, name: vera cruz
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=vera cruz
    Processing city #346 out of 600, name: honolulu
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=honolulu
    Processing city #347 out of 600, name: chernaya kholunitsa
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=chernaya kholunitsa
    Processing city #348 out of 600, name: liuhe
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=liuhe
    Processing city #349 out of 600, name: nola
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=nola
    Processing city #350 out of 600, name: kermanshah
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=kermanshah
    Processing city #351 out of 600, name: north augusta
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=north augusta
    Processing city #352 out of 600, name: gollere
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=gollere
    Processing city #353 out of 600, name: sheridan
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=sheridan
    Processing city #354 out of 600, name: zemio
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=zemio
    Processing city #355 out of 600, name: betlitsa
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=betlitsa
    Processing city #356 out of 600, name: tongren
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=tongren
    Processing city #357 out of 600, name: la rioja
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=la rioja
    Processing city #358 out of 600, name: don benito
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=don benito
    Processing city #359 out of 600, name: goya
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=goya
    Processing city #360 out of 600, name: petropavl
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=petropavl
    Processing city #361 out of 600, name: larsnes
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=larsnes
    Processing city #362 out of 600, name: chik
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=chik
    Processing city #363 out of 600, name: poum
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=poum
    Processing city #364 out of 600, name: camargo
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=camargo
    Processing city #365 out of 600, name: tallahassee
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=tallahassee
    Processing city #366 out of 600, name: fiche
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=fiche
    Processing city #367 out of 600, name: san pablo
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=san pablo
    Processing city #368 out of 600, name: iacu
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=iacu
    Processing city #369 out of 600, name: enterprise
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=enterprise
    Processing city #370 out of 600, name: boden
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=boden
    Processing city #371 out of 600, name: kathmandu
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=kathmandu
    Processing city #372 out of 600, name: mitchell
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=mitchell
    Processing city #373 out of 600, name: shilka
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=shilka
    Processing city #374 out of 600, name: aleksandrovsk-sakhalinskiy
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=aleksandrovsk-sakhalinskiy
    Processing city #375 out of 600, name: soria
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=soria
    Processing city #376 out of 600, name: fez
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=fez
    Processing city #377 out of 600, name: dong xoai
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=dong xoai
    Processing city #378 out of 600, name: adeje
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=adeje
    Processing city #379 out of 600, name: quthing
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=quthing
    Processing city #380 out of 600, name: mbanza-ngungu
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=mbanza-ngungu
    Processing city #381 out of 600, name: tambo
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=tambo
    Processing city #382 out of 600, name: golden
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=golden
    Processing city #383 out of 600, name: rocky mount
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=rocky mount
    Processing city #384 out of 600, name: forest
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=forest
    Processing city #385 out of 600, name: mineiros
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=mineiros
    Processing city #386 out of 600, name: tapes
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=tapes
    Processing city #387 out of 600, name: tignere
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=tignere
    Processing city #388 out of 600, name: orshanka
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=orshanka
    Processing city #389 out of 600, name: bikaner
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=bikaner
    Processing city #390 out of 600, name: santa josefa
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=santa josefa
    Processing city #391 out of 600, name: florence
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=florence
    Processing city #392 out of 600, name: mtwara
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=mtwara
    Processing city #393 out of 600, name: barnbach
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=barnbach
    Processing city #394 out of 600, name: martyush
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=martyush
    Processing city #395 out of 600, name: medak
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=medak
    Processing city #396 out of 600, name: bandar
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=bandar
    Processing city #397 out of 600, name: kalanwali
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=kalanwali
    Processing city #398 out of 600, name: ushumun
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=ushumun
    Processing city #399 out of 600, name: abancay
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=abancay
    Processing city #400 out of 600, name: sharlyk
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=sharlyk
    Processing city #401 out of 600, name: kazachinskoye
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=kazachinskoye
    Processing city #402 out of 600, name: dokka
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=dokka
    Processing city #403 out of 600, name: kimberley
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=kimberley
    Processing city #404 out of 600, name: agva
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=agva
    Processing city #405 out of 600, name: juchitlan
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=juchitlan
    Processing city #406 out of 600, name: ohaba lunga
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=ohaba lunga
    Processing city #407 out of 600, name: kindersley
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=kindersley
    Processing city #408 out of 600, name: pingshan
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=pingshan
    Processing city #409 out of 600, name: yarensk
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=yarensk
    Processing city #410 out of 600, name: george town
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=george town
    Processing city #411 out of 600, name: aiquile
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=aiquile
    Processing city #412 out of 600, name: catalao
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=catalao
    Processing city #413 out of 600, name: aybak
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=aybak
    Processing city #414 out of 600, name: san gabriel
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=san gabriel
    Processing city #415 out of 600, name: jever
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=jever
    Processing city #416 out of 600, name: srisailam
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=srisailam
    Processing city #417 out of 600, name: halberstadt
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=halberstadt
    Processing city #418 out of 600, name: lambarene
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=lambarene
    Processing city #419 out of 600, name: khangarh
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=khangarh
    Processing city #420 out of 600, name: diego de almagro
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=diego de almagro
    Processing city #421 out of 600, name: asenovgrad
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=asenovgrad
    Processing city #422 out of 600, name: shaoguan
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=shaoguan
    Processing city #423 out of 600, name: damphu
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=damphu
    Processing city #424 out of 600, name: arteaga
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=arteaga
    Processing city #425 out of 600, name: durres
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=durres
    Processing city #426 out of 600, name: muriwai beach
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=muriwai beach
    Processing city #427 out of 600, name: poronaysk
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=poronaysk
    Processing city #428 out of 600, name: canakkale
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=canakkale
    Processing city #429 out of 600, name: magugu
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=magugu
    Processing city #430 out of 600, name: novobiryusinskiy
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=novobiryusinskiy
    Processing city #431 out of 600, name: shaartuz
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=shaartuz
    Processing city #432 out of 600, name: amberley
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=amberley
    Processing city #433 out of 600, name: aracaju
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=aracaju
    Processing city #434 out of 600, name: parichhatgarh
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=parichhatgarh
    Processing city #435 out of 600, name: meridian
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=meridian
    Processing city #436 out of 600, name: ambikapur
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=ambikapur
    Processing city #437 out of 600, name: karur
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=karur
    Processing city #438 out of 600, name: svobodnyy
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=svobodnyy
    Processing city #439 out of 600, name: carnot
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=carnot
    Processing city #440 out of 600, name: tsabong
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=tsabong
    Processing city #441 out of 600, name: cockburn harbour
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=cockburn harbour
    Processing city #442 out of 600, name: volchansk
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=volchansk
    Processing city #443 out of 600, name: mortka
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=mortka
    Processing city #444 out of 600, name: rio gallegos
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=rio gallegos
    Processing city #445 out of 600, name: chulym
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=chulym
    Processing city #446 out of 600, name: narasannapeta
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=narasannapeta
    Processing city #447 out of 600, name: hervey bay
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=hervey bay
    Processing city #448 out of 600, name: nagato
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=nagato
    Processing city #449 out of 600, name: guangyuan
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=guangyuan
    Processing city #450 out of 600, name: yanan
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=yanan
    Processing city #451 out of 600, name: subtanjalla
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=subtanjalla
    Processing city #452 out of 600, name: guane
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=guane
    Processing city #453 out of 600, name: oistins
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=oistins
    Processing city #454 out of 600, name: muisne
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=muisne
    Processing city #455 out of 600, name: kristiansund
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=kristiansund
    Processing city #456 out of 600, name: priargunsk
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=priargunsk
    Processing city #457 out of 600, name: bogande
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=bogande
    Processing city #458 out of 600, name: whitley bay
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=whitley bay
    Processing city #459 out of 600, name: saltpond
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=saltpond
    Processing city #460 out of 600, name: yendi
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=yendi
    Processing city #461 out of 600, name: lahad datu
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=lahad datu
    Processing city #462 out of 600, name: melfi
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=melfi
    Processing city #463 out of 600, name: qaanaaq
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=qaanaaq
    Processing city #464 out of 600, name: angangxi
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=angangxi
    Processing city #465 out of 600, name: hvide sande
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=hvide sande
    Processing city #466 out of 600, name: youghal
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=youghal
    Processing city #467 out of 600, name: vorchdorf
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=vorchdorf
    Processing city #468 out of 600, name: thyboron
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=thyboron
    Processing city #469 out of 600, name: blonduos
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=blonduos
    Processing city #470 out of 600, name: tynda
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=tynda
    Processing city #471 out of 600, name: meadow lake
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=meadow lake
    Processing city #472 out of 600, name: camabatela
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=camabatela
    Processing city #473 out of 600, name: huallanca
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=huallanca
    Processing city #474 out of 600, name: wellington
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=wellington
    Processing city #475 out of 600, name: caxito
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=caxito
    Processing city #476 out of 600, name: gazli
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=gazli
    Processing city #477 out of 600, name: beaupre
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=beaupre
    Processing city #478 out of 600, name: fuyang
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=fuyang
    Processing city #479 out of 600, name: bandar-e lengeh
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=bandar-e lengeh
    Processing city #480 out of 600, name: stulovo
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=stulovo
    Processing city #481 out of 600, name: kalevala
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=kalevala
    Processing city #482 out of 600, name: barda
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=barda
    Processing city #483 out of 600, name: cotonou
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=cotonou
    Processing city #484 out of 600, name: khajuraho
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=khajuraho
    Processing city #485 out of 600, name: kovylkino
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=kovylkino
    Processing city #486 out of 600, name: calatayud
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=calatayud
    Processing city #487 out of 600, name: dargaville
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=dargaville
    Processing city #488 out of 600, name: brooks
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=brooks
    Processing city #489 out of 600, name: harrismith
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=harrismith
    Processing city #490 out of 600, name: bulungu
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=bulungu
    Processing city #491 out of 600, name: qunduz
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=qunduz
    Processing city #492 out of 600, name: motygino
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=motygino
    Processing city #493 out of 600, name: muros
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=muros
    Processing city #494 out of 600, name: villa guerrero
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=villa guerrero
    Processing city #495 out of 600, name: bay roberts
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=bay roberts
    Processing city #496 out of 600, name: lyaskelya
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=lyaskelya
    Processing city #497 out of 600, name: dongying
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=dongying
    Processing city #498 out of 600, name: mayor pablo lagerenza
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=mayor pablo lagerenza
    Processing city #499 out of 600, name: dongli
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=dongli
    Processing city #500 out of 600, name: dharur
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=dharur
    Processing city #501 out of 600, name: gurskoye
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=gurskoye
    Processing city #502 out of 600, name: stolin
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=stolin
    Processing city #503 out of 600, name: villa bruzual
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=villa bruzual
    Processing city #504 out of 600, name: ponnampet
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=ponnampet
    Processing city #505 out of 600, name: minas
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=minas
    Processing city #506 out of 600, name: kolondieba
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=kolondieba
    Processing city #507 out of 600, name: manama
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=manama
    Processing city #508 out of 600, name: araguacu
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=araguacu
    Processing city #509 out of 600, name: si sa ket
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=si sa ket
    Processing city #510 out of 600, name: port harcourt
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=port harcourt
    Processing city #511 out of 600, name: catia la mar
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=catia la mar
    Processing city #512 out of 600, name: igurubi
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=igurubi
    Processing city #513 out of 600, name: gavle
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=gavle
    Processing city #514 out of 600, name: visnes
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=visnes
    Processing city #515 out of 600, name: isla vista
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=isla vista
    Processing city #516 out of 600, name: zakamensk
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=zakamensk
    Processing city #517 out of 600, name: kahone
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=kahone
    Processing city #518 out of 600, name: selaphum
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=selaphum
    Processing city #519 out of 600, name: dondo
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=dondo
    Processing city #520 out of 600, name: cherdyn
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=cherdyn
    Processing city #521 out of 600, name: krasnokholmskiy
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=krasnokholmskiy
    Processing city #522 out of 600, name: moche
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=moche
    Processing city #523 out of 600, name: sohag
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=sohag
    Processing city #524 out of 600, name: copiapo
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=copiapo
    Processing city #525 out of 600, name: kahului
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=kahului
    Processing city #526 out of 600, name: abiy adi
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=abiy adi
    Processing city #527 out of 600, name: selenduma
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=selenduma
    Processing city #528 out of 600, name: sembe
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=sembe
    Processing city #529 out of 600, name: lichtenfels
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=lichtenfels
    Processing city #530 out of 600, name: anton lizardo
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=anton lizardo
    Processing city #531 out of 600, name: kharp
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=kharp
    Processing city #532 out of 600, name: ceahlau
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=ceahlau
    Processing city #533 out of 600, name: khovu-aksy
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=khovu-aksy
    Processing city #534 out of 600, name: amarpur
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=amarpur
    Processing city #535 out of 600, name: novoorsk
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=novoorsk
    Processing city #536 out of 600, name: el wasta
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=el wasta
    Processing city #537 out of 600, name: savinskiy
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=savinskiy
    Processing city #538 out of 600, name: moramanga
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=moramanga
    Processing city #539 out of 600, name: supe
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=supe
    Processing city #540 out of 600, name: guatire
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=guatire
    Processing city #541 out of 600, name: saint-paul
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=saint-paul
    Processing city #542 out of 600, name: king city
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=king city
    Processing city #543 out of 600, name: ust-maya
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=ust-maya
    Processing city #544 out of 600, name: utiroa
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=utiroa
    Processing city #545 out of 600, name: papillion
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=papillion
    Processing city #546 out of 600, name: kamuli
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=kamuli
    Processing city #547 out of 600, name: villa literno
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=villa literno
    Processing city #548 out of 600, name: wazzan
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=wazzan
    Processing city #549 out of 600, name: magadan
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=magadan
    Processing city #550 out of 600, name: yanggu
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=yanggu
    Processing city #551 out of 600, name: meyzieu
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=meyzieu
    Processing city #552 out of 600, name: corinto
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=corinto
    Processing city #553 out of 600, name: nampula
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=nampula
    Processing city #554 out of 600, name: kirzhach
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=kirzhach
    Processing city #555 out of 600, name: aldama
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=aldama
    Processing city #556 out of 600, name: vieste
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=vieste
    Processing city #557 out of 600, name: inderborskiy
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=inderborskiy
    Processing city #558 out of 600, name: eston
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=eston
    Processing city #559 out of 600, name: shaoyang
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=shaoyang
    Processing city #560 out of 600, name: tarsus
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=tarsus
    Processing city #561 out of 600, name: hecelchakan
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=hecelchakan
    Processing city #562 out of 600, name: bambous virieux
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=bambous virieux
    Processing city #563 out of 600, name: cassilandia
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=cassilandia
    Processing city #564 out of 600, name: putina
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=putina
    Processing city #565 out of 600, name: palembang
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=palembang
    Processing city #566 out of 600, name: dunmore town
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=dunmore town
    Processing city #567 out of 600, name: pailon
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=pailon
    Processing city #568 out of 600, name: pizarro
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=pizarro
    Processing city #569 out of 600, name: shellbrook
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=shellbrook
    Processing city #570 out of 600, name: kalyazin
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=kalyazin
    Processing city #571 out of 600, name: nizwa
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=nizwa
    Processing city #572 out of 600, name: grand baie
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=grand baie
    Processing city #573 out of 600, name: takayama
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=takayama
    Processing city #574 out of 600, name: balaguer
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=balaguer
    Processing city #575 out of 600, name: tres passos
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=tres passos
    Processing city #576 out of 600, name: mokhotlong
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=mokhotlong
    Processing city #577 out of 600, name: kuah
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=kuah
    Processing city #578 out of 600, name: sveti nikole
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=sveti nikole
    Processing city #579 out of 600, name: nizamabad
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=nizamabad
    Processing city #580 out of 600, name: naples
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=naples
    Processing city #581 out of 600, name: verona
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=verona
    Processing city #582 out of 600, name: thurso
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=thurso
    Processing city #583 out of 600, name: tygda
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=tygda
    Processing city #584 out of 600, name: ushibuka
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=ushibuka
    Processing city #585 out of 600, name: virden
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=virden
    Processing city #586 out of 600, name: verkhoshizhemye
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=verkhoshizhemye
    Processing city #587 out of 600, name: thompson
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=thompson
    Processing city #588 out of 600, name: chaa-khol
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=chaa-khol
    Processing city #589 out of 600, name: sharan
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=sharan
    Processing city #590 out of 600, name: odemis
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=odemis
    Processing city #591 out of 600, name: oneonta
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=oneonta
    Processing city #592 out of 600, name: tigzirt
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=tigzirt
    Processing city #593 out of 600, name: vredendal
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=vredendal
    Processing city #594 out of 600, name: cananea
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=cananea
    Processing city #595 out of 600, name: dieppe
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=dieppe
    Processing city #596 out of 600, name: barmer
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=barmer
    Processing city #597 out of 600, name: marv dasht
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=marv dasht
    Processing city #598 out of 600, name: juba
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=juba
    Processing city #599 out of 600, name: det udom
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=det udom
    Processing city #600 out of 600, name: mopti
    http://api.openweathermap.org/data/2.5/weather?appid=25bc90a1196e6f153eece0bc0b0fc9eb&units=Imperial&q=mopti
    ------------------------
    Data retrieval done
    


```python
#clean the data and export to a csv file
cities_sample = cities_sample.dropna()
cities_sample.to_csv("export_Weatherpy_data.csv")
# print the number of cities exported
cities_sample.count()
```




    City          542
    Lat           542
    Lng           542
    Country       542
    Date          542
    Max temp      542
    Humidity      542
    Cloudiness    542
    Wind speed    542
    dtype: int64




```python
cities_sample.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>City</th>
      <th>Lat</th>
      <th>Lng</th>
      <th>Country</th>
      <th>Date</th>
      <th>Max temp</th>
      <th>Humidity</th>
      <th>Cloudiness</th>
      <th>Wind speed</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>1</th>
      <td>kiunga</td>
      <td>-6.12</td>
      <td>141.3</td>
      <td>PG</td>
      <td>1522877935</td>
      <td>71.85</td>
      <td>96</td>
      <td>20</td>
      <td>1.95</td>
    </tr>
    <tr>
      <th>2</th>
      <td>xinqing</td>
      <td>48.28</td>
      <td>129.53</td>
      <td>CN</td>
      <td>1522877944</td>
      <td>18.12</td>
      <td>32</td>
      <td>0</td>
      <td>5.97</td>
    </tr>
    <tr>
      <th>3</th>
      <td>orlovskiy</td>
      <td>46.87</td>
      <td>42.05</td>
      <td>RU</td>
      <td>1522877936</td>
      <td>45.21</td>
      <td>88</td>
      <td>88</td>
      <td>8.55</td>
    </tr>
    <tr>
      <th>4</th>
      <td>maldonado</td>
      <td>-34.91</td>
      <td>-54.96</td>
      <td>UY</td>
      <td>1522875600</td>
      <td>73.4</td>
      <td>73</td>
      <td>20</td>
      <td>6.93</td>
    </tr>
    <tr>
      <th>5</th>
      <td>farrukhnagar</td>
      <td>28.45</td>
      <td>76.82</td>
      <td>IN</td>
      <td>1522877400</td>
      <td>82.4</td>
      <td>54</td>
      <td>24</td>
      <td>3.4</td>
    </tr>
  </tbody>
</table>
</div>



# Temperature (F) vs Latitude


```python
# plot scatterplot graph - City latitude vs temperature
date = time.strftime("%m/%d/%Y")
plt.scatter(cities_sample['Lat'],cities_sample['Max temp'], color="orange", edgecolor ="black", s = 30, label = "data", linewidth = 0.5)

plt.title(f"City Temperature (F) vs Latitude {date}")
plt.xlabel("Latitude")
plt.ylabel("Max Temperature (F)")
plt.grid(True)
plt.xlim(-80,80)
plt.ylim(-30, 100)
plt.savefig("Temperature.png")
plt.show()
```


![png](output_13_0.png)


# Humidity (%) vs Latitude


```python
# plot scatterplot graph - City latitude vs humidity
date = time.strftime("%m/%d/%Y")
plt.scatter(cities_sample['Lat'],cities_sample['Humidity'], color="lightblue", edgecolor ="black", s = 30, label = "data", linewidth = 0.5)

plt.title(f"City Humidity (%) vs Latitude {date}")
plt.xlabel("Latitude")
plt.ylabel("Humidity (%))")
plt.grid(True)
plt.xlim(-80,80)
plt.ylim(-30, 110)
plt.savefig("Humidity.png")
plt.show()
```


![png](output_15_0.png)


# Cloudiness (%) vs Latitude


```python
# plot scatterplot graph - City Latitude vs cloudiness
date = time.strftime("%m/%d/%Y")
plt.scatter(cities_sample['Lat'],cities_sample['Cloudiness'], color="grey", edgecolor ="black", s = 30, label = "data", linewidth = 0.5)

plt.title(f"City Cloudiness (%) vs Latitude {date}")
plt.xlabel("Latitude")
plt.ylabel("Cloudiness (%))")
plt.grid(True)
plt.xlim(-80,80)
plt.ylim(-30, 110)
plt.savefig("Cloudiness.png")
plt.show()
```


![png](output_17_0.png)


# Wind speed (mph) vs Latitude


```python
# plot scatterplot graph - City Latitude vs wind speed
date = time.strftime("%m/%d/%Y")
plt.scatter(cities_sample['Lat'],cities_sample['Wind speed'], color="green", edgecolor ="black", s = 30, label = "data", linewidth = 0.5)

plt.title(f"City Wind speed (mph) vs Latitude {date}")
plt.xlabel("Latitude")
plt.ylabel("Wind speed (mph))")
plt.grid(True)
plt.xlim(-80,80)
plt.ylim(-10, 40)
plt.savefig("Wind_speed.png")
plt.show()
```


![png](output_19_0.png)


# Cities location - Latitude vs Longitude


```python
# plot scatterplot graph - City Latitude vs wind speed
date = time.strftime("%m/%d/%Y")
plt.scatter(cities_sample['Lng'], cities_sample['Lat'], color="red", edgecolor ="black", s = 30, label = "data", linewidth = 0.5)

plt.title(f"City Long vs Latitude {date}")
plt.xlabel("Longitude")
plt.ylabel("Latitude")
plt.grid(True)
plt.xlim(-180,180)
plt.ylim(-90, 90)
plt.savefig("Latitude_vs_Longitude.png")
plt.show()
```


![png](output_21_0.png)

