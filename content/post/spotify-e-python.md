+++
title = "Estraiamo i dati da Spotify con Python"
date = "2021-04-27T17:07:30+02:002"
draft = false
author = "Giacomo Lorenzo"
description = "Etl di trasformazione e caricamento su database locale dei dati di spotify"
tags = [
    "Python",
    "Spotify",
    "Etl"
]
categories = [
    "Python"
]
+++

## Mini guida per estrarre i dati delle ultime canzoni ascoltate da spotify

Innanzitutto sarà necessario installare python se non lo si ha nel proprio ambiente installiamolo da qui, [Python]([https://link](https://www.python.org/downloads/))


## Attrezziamoci

### Dipendenze Python

Sarà necessario installare i [Requirement.txt](https://raw.githubusercontent.com/giacomolorenzo/spotify-etl-data/main/requirement.txt) tramite il comando:

`pip install requirements.txt`

### Token Spotify

Adesso sarà necessario reperire un token dalla developer console di spotify [Spotify Console](https://developer.spotify.com/console/get-recently-played/?limit=&after=&before=)
Una volta ottenuto il token possiamo cominciare a scrivere il nostro file python

cominciamo importanto le librerie e dichiarando le costanti:
```
import sqlalchemy
import pandas as pd 
from sqlalchemy.orm import sessionmaker
import requests
import urllib
import json
from datetime import datetime
import datetime
import sqlite3
import os
from dotenv import load_dotenv
load_dotenv(os.path.join("./", '.env'))
DATABASE_LOCATION = "sqlite:///my_played_tracks.sqlite"
TOKEN= os.getenv("TOKEN_SPOTIFY")
```

**Creiamo un file .env** che ci servirà per ottenere il valore del token di spotify qui un esempio:

```
export TOKEN_SPOTIFY=<SPOTIFY_TOKEN>
```
### Interroghiamo spotify

Definiamo un if in modo da eseguire queste azioni solo come main e poi definiamo un oggetto headers in cui all'interno definiamo tutto il necessario per avere una risposta dalle API di spotify
```
if __name__ == "__main__":
    headers = {
        "Accept": "application/json",
        "Content-Type": "application/json",
        "Authorization": "Bearer {token}".format(token=TOKEN)
    }
```
ok adesso sarà necessario capire quale API di Spotify vogliamo usare nel nostro caso useremo **Recentlyplayed** che ci permetterà di vedere le ultime canzoni che abbiamo ascoltato negli ultimi X giorni, ma vediamo come fare.

ok adesso definiamo 3 variabili per gestire il tempo in modo da passarlo alle api di spotify nel modo corretto:

```
    today = datetime.datetime.now()
    yesterday = today - datetime.timedelta(days=60)
    yesterday_unix_timestamp = int(yesterday.timestamp()) * 1000
```
**today** rappresenta la data attuale.

**yesterday** la data dalla quale vogliamo partire nel nostro caso gli ultimi 60 giorni. 

**yesterday_unix_timestamp** è la rappresentazione della data in formato timestamp che è quello che viene 
accettato dalle API di Spotify.

### Facciamo la nostra prima richiesta

```
r = requests.get("https://api.spotify.com/v1/me/player/recently-played?after={time}".format(time=yesterday_unix_timestamp),headers=headers) 
```
Andiamo ad analizzare la richiesta per effettuarla utilizzeremo [Request](https://pypi.org/project/requests/)

Come primo parametro l'url come secondo parametro l'headers che abbiamo precedentemente creato.

questo è un esempio di risposta:

```json
{
  "items" : [ {
    "track" : {
      "album" : {
        "album_type" : "single",
        "artists" : [ {
          "external_urls" : {
            "spotify" : "https://open.spotify.com/artist/6Tu0luJL7EoFv1RsHZP30p"
          },
          "href" : "https://api.spotify.com/v1/artists/6Tu0luJL7EoFv1RsHZP30p",
          "id" : "6Tu0luJL7EoFv1RsHZP30p",
          "name" : "Ranji",
          "type" : "artist",
          "uri" : "spotify:artist:6Tu0luJL7EoFv1RsHZP30p"
        }, {
          "external_urls" : {
            "spotify" : "https://open.spotify.com/artist/6ZJDt01Lh0XOPMMJbUMcUi"
          },
          "href" : "https://api.spotify.com/v1/artists/6ZJDt01Lh0XOPMMJbUMcUi",
          "id" : "6ZJDt01Lh0XOPMMJbUMcUi",
          "name" : "Ghost Rider",
          "type" : "artist",
          "uri" : "spotify:artist:6ZJDt01Lh0XOPMMJbUMcUi"
        } ],
        "available_markets" : [ "AD", "AE", "AG", "AL", "AM", "AO", "AR", "AT", "AU", "AZ", "BA", "BB", "BD", "BE", "BF", "BG", "BH", "BI", "BJ", "BN", "BO", "BR", "BS", "BT", "BW", "BY", "BZ", "CA", "CH", "CI", "CL", "CM", "CO", "CR", "CV", "CW", "CY", "CZ", "DE", "DJ", "DK", "DM", "DO", "DZ", "EC", "EE", "EG", "ES", "FI", "FJ", "FM", "FR", "GA", "GB", "GD", "GE", "GH", "GM", "GN", "GQ", "GR", "GT", "GW", "GY", "HK", "HN", "HR", "HT", "HU", "ID", "IE", "IL", "IN", "IS", "IT", "JM", "JO", "JP", "KE", "KG", "KH", "KI", "KM", "KN", "KR", "KW", "KZ", "LA", "LB", "LC", "LI", "LK", "LR", "LS", "LT", "LU", "LV", "MA", "MC", "MD", "ME", "MG", "MH", "MK", "ML", "MN", "MO", "MR", "MT", "MU", "MV", "MW", "MX", "MY", "MZ", "NA", "NE", "NG", "NI", "NL", "NO", "NP", "NR", "NZ", "OM", "PA", "PE", "PG", "PH", "PK", "PL", "PS", "PT", "PW", "PY", "QA", "RO", "RS", "RU", "RW", "SA", "SB", "SC", "SE", "SG", "SI", "SK", "SL", "SM", "SN", "SR", "ST", "SV", "SZ", "TD", "TG", "TH", "TL", "TN", "TO", "TR", "TT", "TV", "TW", "TZ", "UA", "UG", "US", "UY", "UZ", "VC", "VN", "VU", "WS", "XK", "ZA", "ZM", "ZW" ],
        "external_urls" : {
          "spotify" : "https://open.spotify.com/album/2uKiw3sMzeXdim7hmBp9oF"
        },
        "href" : "https://api.spotify.com/v1/albums/2uKiw3sMzeXdim7hmBp9oF",
        "id" : "2uKiw3sMzeXdim7hmBp9oF",
        "images" : [ {
          "height" : 640,
          "url" : "https://i.scdn.co/image/ab67616d0000b273af959e67cc49b1de59e00ffe",
          "width" : 640
        }, {
          "height" : 300,
          "url" : "https://i.scdn.co/image/ab67616d00001e02af959e67cc49b1de59e00ffe",
          "width" : 300
        }, {
          "height" : 64,
          "url" : "https://i.scdn.co/image/ab67616d00004851af959e67cc49b1de59e00ffe",
          "width" : 64
        } ],
        "name" : "Can't Sleep (Radio Version)",
        "release_date" : "2020-06-26",
        "release_date_precision" : "day",
        "total_tracks" : 1,
        "type" : "album",
        "uri" : "spotify:album:2uKiw3sMzeXdim7hmBp9oF"
      },
      "artists" : [ {
        "external_urls" : {
          "spotify" : "https://open.spotify.com/artist/6Tu0luJL7EoFv1RsHZP30p"
        },
        "href" : "https://api.spotify.com/v1/artists/6Tu0luJL7EoFv1RsHZP30p",
        "id" : "6Tu0luJL7EoFv1RsHZP30p",
        "name" : "Ranji",
        "type" : "artist",
        "uri" : "spotify:artist:6Tu0luJL7EoFv1RsHZP30p"
      }, {
        "external_urls" : {
          "spotify" : "https://open.spotify.com/artist/6ZJDt01Lh0XOPMMJbUMcUi"
        },
        "href" : "https://api.spotify.com/v1/artists/6ZJDt01Lh0XOPMMJbUMcUi",
        "id" : "6ZJDt01Lh0XOPMMJbUMcUi",
        "name" : "Ghost Rider",
        "type" : "artist",
        "uri" : "spotify:artist:6ZJDt01Lh0XOPMMJbUMcUi"
      } ],
      "available_markets" : [ "AD", "AE", "AG", "AL", "AM", "AO", "AR", "AT", "AU", "AZ", "BA", "BB", "BD", "BE", "BF", "BG", "BH", "BI", "BJ", "BN", "BO", "BR", "BS", "BT", "BW", "BY", "BZ", "CA", "CH", "CI", "CL", "CM", "CO", "CR", "CV", "CW", "CY", "CZ", "DE", "DJ", "DK", "DM", "DO", "DZ", "EC", "EE", "EG", "ES", "FI", "FJ", "FM", "FR", "GA", "GB", "GD", "GE", "GH", "GM", "GN", "GQ", "GR", "GT", "GW", "GY", "HK", "HN", "HR", "HT", "HU", "ID", "IE", "IL", "IN", "IS", "IT", "JM", "JO", "JP", "KE", "KG", "KH", "KI", "KM", "KN", "KR", "KW", "KZ", "LA", "LB", "LC", "LI", "LK", "LR", "LS", "LT", "LU", "LV", "MA", "MC", "MD", "ME", "MG", "MH", "MK", "ML", "MN", "MO", "MR", "MT", "MU", "MV", "MW", "MX", "MY", "MZ", "NA", "NE", "NG", "NI", "NL", "NO", "NP", "NR", "NZ", "OM", "PA", "PE", "PG", "PH", "PK", "PL", "PS", "PT", "PW", "PY", "QA", "RO", "RS", "RU", "RW", "SA", "SB", "SC", "SE", "SG", "SI", "SK", "SL", "SM", "SN", "SR", "ST", "SV", "SZ", "TD", "TG", "TH", "TL", "TN", "TO", "TR", "TT", "TV", "TW", "TZ", "UA", "UG", "US", "UY", "UZ", "VC", "VN", "VU", "WS", "XK", "ZA", "ZM", "ZW" ],
      "disc_number" : 1,
      "duration_ms" : 201467,
      "explicit" : false,
      "external_ids" : {
        "isrc" : "DEGA22003204"
      },
      "external_urls" : {
        "spotify" : "https://open.spotify.com/track/1ChzNnL5BTDCDYQHxrT1l9"
      },
      "href" : "https://api.spotify.com/v1/tracks/1ChzNnL5BTDCDYQHxrT1l9",
      "id" : "1ChzNnL5BTDCDYQHxrT1l9",
      "is_local" : false,
      "name" : "Can't Sleep - Radio Version",
      "popularity" : 60,
      "preview_url" : "https://p.scdn.co/mp3-preview/7054cfd91e1804006c4bef0bc72df517a8794a42?cid=774b29d4f13844c495f206cafdad9c86",
      "track_number" : 1,
      "type" : "track",
      "uri" : "spotify:track:1ChzNnL5BTDCDYQHxrT1l9"
    },
    "played_at" : "2021-04-26T12:58:27.757Z",
    "context" : null
  } ],
  "next" : "https://api.spotify.com/v1/me/player/recently-played?before=1619441907757&limit=1",
  "cursors" : {
    "after" : "1619441907757",
    "before" : "1619441907757"
  },
  "limit" : 1,
  "href" : "https://api.spotify.com/v1/me/player/recently-played?limit=1"

  ```
adesso siamo pronti a usare i dizionari e pandas con i [DataFrame](https://pandas.pydata.org/docs/reference/api/pandas.DataFrame.html)

quindi cataloghiamo i nostri dati dal json:

```python
data = r.json()
song_names = []
artist_names = []
played_at_list = []
timestamps = []
print(data)
for song in data["items"]:
    print(song["track"]["name"])
    song_names.append(song["track"]["name"])
    artist_names.append(song["track"]["album"]["artists"][0]["name"])
    played_at_list.append(song["played_at"])
    timestamps.append(song["played_at"][0:10])

song_dict = {
    "song_name" : song_names,
    "artist_name": artist_names,
    "played_at": played_at_list,
    "timestamp": timestamps
}
song_df = pd.DataFrame(song_dict, columns = ["song_name","artist_name","played_at","timestamp"] )

print(song_df)
#Validate
if check_if_valid_data(song_df):
    print("Data valid, proceed to Load stage")
```
analizziamo il codice:

### Creiamo prima degli array che rappresentano:
1. song_names (nomi delle canzoni)
2. artist_names (nome degli artisti)
3. plated_at_list (lista riprodotta)
4. timestamps (tempo nel quale è stato riprodotto il brano)

```python

for song in data["items"]:
    print(song["track"]["name"])
    song_names.append(song["track"]["name"])
    artist_names.append(song["track"]["album"]["artists"][0]["name"])
    played_at_list.append(song["played_at"])
    timestamps.append(song["played_at"][0:10])
```

tutti i contenuti sono dentro items quindi ci posizioniamo li e lo scorriamo e aggiungiamo a ogni array le relative informazioni

### Creiamo un dizionario in python
```python
song_dict = {
    "song_name" : song_names,
    "artist_name": artist_names,
    "played_at": played_at_list,
    "timestamp": timestamps
}
```

### Creiamo un DataFrame rappresentabile con pandas
```python
song_df = pd.DataFrame(song_dict, columns = ["song_name","artist_name","played_at","timestamp"] )
```
Pandas è una libreria che permette la manipolazione dei dati e la rappresentazione tramite dei DataFrame e non solo.

I parametri da passare a pandas saranno i dizionari e le colonne da rappresentare.

esempio di output:
```python
print(song_df)
```

```
                song_name  artist_name                 played_at   timestamp
0                 Ride It       Regard  2021-04-15T14:19:54.032Z  2021-04-15
1                 Be Mine     Ofenbach  2021-04-15T14:16:14.467Z  2021-04-15
2   Roses - Imanbek Remix    SAINt JHN  2021-04-15T14:13:32.656Z  2021-04-15
3                 Ride It       Regard  2021-04-15T14:10:35.325Z  2021-04-15
4                 Ride It       Regard  2021-04-14T12:14:36.069Z  2021-04-14
5                 Ride It       Regard  2021-04-14T12:10:56.966Z  2021-04-14
6   Roses - Imanbek Remix    SAINt JHN  2021-04-14T12:06:49.074Z  2021-04-14
7                 Ride It       Regard  2021-04-14T12:03:51.746Z  2021-04-14
8                 Be Mine     Ofenbach  2021-04-14T08:07:25.552Z  2021-04-14
9   Roses - Imanbek Remix    SAINt JHN  2021-04-14T07:32:04.034Z  2021-04-14
10                Ride It       Regard  2021-04-14T07:28:30.376Z  2021-04-14
11    Che Ne Sanno I 2000  Gabry Ponte  2021-04-14T07:23:34.461Z  2021-04-14
12                Be Mine     Ofenbach  2021-04-14T07:19:39.603Z  2021-04-14
13  Roses - Imanbek Remix    SAINt JHN  2021-04-14T07:16:57.939Z  2021-04-14
14                Ride It       Regard  2021-04-14T07:13:40.048Z  2021-04-14
15  Roses - Imanbek Remix    SAINt JHN  2021-04-13T17:58:10.820Z  2021-04-13
16                Ride It       Regard  2021-04-13T12:37:49.168Z  2021-04-13
17  Roses - Imanbek Remix    SAINt JHN  2021-04-13T12:34:04.503Z  2021-04-13
18                Ride It       Regard  2021-04-13T12:31:07.186Z  2021-04-13
19                Ride It       Regard      2021-04-13T12:27:54Z  2021-04-13
```

