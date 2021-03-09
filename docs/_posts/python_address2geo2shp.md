---
layout: page
title: "【Python】住所から緯度経度取得してshapeファイルに変換"
date: 2021-03-09
---

# 【Python】住所から緯度経度取得してshapeファイルに変換

geocodingのapiを使用して、住所を緯度経度に変換する。

単純に、住所から緯度経度を取得するだけなら以下のサイトでできる。
https://qiita.com/paulxll/items/7bc4a5b0529a8d784673

```python:住所->緯度経度
import requests 
from bs4 import BeautifulSoup 
import time 
import tqdm 

URL = 'http://www.geocoding.jp/api/' 

def coordinate(address):    
    payload = {'q': address} 
    html = requests.get(URL, params=payload) 
    soup = BeautifulSoup(html.content, "html.parser") 
    if soup.find('error'): 
        raise ValueError(f"Invalid address submitted. {address}") 
    latitude = soup.find('lat').string 
    longitude = soup.find('lng').string 
    return [latitude, longitude] 

def coordinates(addresses, interval=10, progress=True): 
    coordinates = [] 
    for address in progress and tqdm(addresses) or addresses: 
        coordinates.append(coordinate(address)) 
        time.sleep(interval) 
    return coordinates
```

ここで得られた緯度経度の情報（Latitude，Longitude）は文字列データであるので、そのままではエラーが出る。

そのため、関数：gpd.points_from_xy()で、数値型へ変換処理を行っておく。

```python:緯度経度->shpファイル
import geopandas as gpd
import pandas as pd
import numpy as np

def addr2geo(data_path):
     data = pd.read_csv(data_path)
     data['Latitude'] = None
     data['Longitude'] = None
     data['geometry'] = None
     for i in tqdm(range(len(data))):
         data.Latitude[i], data.Longitude[i] = coordinate(data.Address[i])
         time.sleep(10)
     data['geometry'] = gpd.points_from_xy(data.Longitude.astype(np.float64), data.Latitude.astype(np.float64))
     result = gpd.GeoDataFrame(data, geometry=data.geometry)
     return result

 def geo2shp(gdf, result_path):
     gdf.to_file(driver='ESRI Shapefile', filename=result_path)
```