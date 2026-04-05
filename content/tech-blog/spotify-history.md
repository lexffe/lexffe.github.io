+++
title = "Going through my Spotify history"
description = "I take a look at my streaming history, big data style."
date = 2026-04-01
+++

# Premise

I am a giant music nerd and recently I am going through a reminiscing nostalgic trip. I want to know what I used to listen. I found out that Spotify allows you to request a copy of your extended streaming history.

(It is impossible to know my listening history before Spotify. 8 year old me relied on family for music and 12 year old me used to play music on YouTube and Google Music (RIP). I requested a copy of my YouTube data using takeout but none of them are useful (they only preserve 1 year of history it seems.))

# The data

The data came in a zip file. Inside is a README PDF that contains descriptions of the columns.
The actual dataset spans through multiple JSON files, each around 12.8MB big.

An example of a data point may look like this:
```js
[
  {
    "ts": "2020-04-11T00:27:53Z",
    "platform": "iOS 13.4 (iPhone11,8)",
    "ms_played": 174674,
    "conn_country": "SG",
    "ip_addr": "203.0.113.x",
    "master_metadata_track_name": "Jack To The Future (feat. Russoul)",
    "master_metadata_album_artist_name": "Alex Metric",
    "master_metadata_album_album_name": "Jack To The Future (feat. Russoul)",
    "spotify_track_uri": "spotify:track:74cksX4vgcm0cHKaV7XE2L",
    "episode_name": null,
    "episode_show_name": null,
    "spotify_episode_uri": null,
    "audiobook_title": null,
    "audiobook_uri": null,
    "audiobook_chapter_uri": null,
    "audiobook_chapter_title": null,
    "reason_start": "trackdone",
    "reason_end": "trackdone",
    "shuffle": false,
    "skipped": false,
    "offline": false,
    "offline_timestamp": null,
    "incognito_mode": false
  },
  // ...
]
```

A few interesting things:

- The IP address can be IPv4 and IPv6. A potential normalisation might be using IPv6's IPv4-Mapped address prefix like `::ffff:1.2.3.4`. The database may already have types to deal with IP addresses though.
- The IP address may get anonymised at the Spotify side. The last 8-bit of the subnet is replaced with a `x`.
  - You can still tell which ISP you were on by using `bgp.he.net`.
- Spotify is being smart and gave us the album artist's name. Makes sense because there could be a featuring artist on the track, or it could be a compilation album.

# "Quack"

Even though it is being given to me in JSON, this looks like tabular data to me. The only reasonable move
is to put it into a database and query it instead of using grep.

For the data size (around 640k rows), I could have also just used SQLite with a CLI but it really isn't suitable for analytic workloads, especially if I am enriching the data with more metadata from Spotify later, the joins alone in SQLite would slow down the queries a lot.

At $work we use ClickHouse for large scale data analysis. I am very much used to ClickHouse SQL syntax. 
But if this is a personal project, why not use this opportunity to learn a new technology?

I turned to DuckDB as I want a database that's as easy to use as SQLite but can handle the data I gave it.

I ingested the data into DuckDB with a tiny bit of normalisation around the IP addresses and I skipped all the
audiobook stuff because I never used it, as well as the podcast stuff.

## File size

|Format|Size (MB)|
|------|----|
|JSON|513|
|DuckDB|38|
|Arrow|53.2|
|Parquet|34.1|

Even with compression, my dataset still weighs 34.1MB. I did some optimisation such as decoding the spotify track URI's base62 ID component into blobs and store the blobs, and normalisations like storing tracks in a different table and create a view to join the history table with the track table back together. This took the file size down to 21.8MB. I guess I can further normalise by having the user agent in a different table / enum, storing `reason_start` and `reason_end` as enums, but that's an exercise for later.

# Excerpt on what I found

```sql
select 
  distinct artist_name,
  count() as count 
FROM spotify_history_view 
WHERE ts between '2020-01-01' and '2026-04-01' 
GROUP BY artist_name 
ORDER BY count desc 
limit 10;
```

My top 10 artists between 2020 and now are
- deadmau5,
- Flume,
- Skrillex, 
- Bring Me The Horizon, 
- Lana Del Rey, 
- London Grammar, 
- Kendrick Lamar,
- Radiohead,
- ODESZA, and
- The Prodigy.

Not surprising. But when did I actually start listening to them?

```sql
with top_artists AS (
select
  distinct artist_name,
  count() as count,
FROM spotify_history_view
WHERE timestamp between '2020-01-01' and '2026-04-01'
GROUP BY artist_name
ORDER BY count desc
limit 10
)   
SELECT 
artist_name, min(timestamp) 
from spotify_history_view 
JOIN top_artists 
using (artist_name) 
group by artist_name;
```

|     artist_name      |  min("timestamp")   |
|----------------------|---------------------|
| Lana Del Rey         | 2014-03-17 06:44:11 |
| The Prodigy          | 2014-12-20 14:24:23 |
| Bring Me The Horizon | 2015-02-14 14:45:11 |
| London Grammar       | 2014-03-17 20:19:13 |
| deadmau5             | 2014-04-14 21:42:46 |
| Flume                | 2015-01-11 02:35:49 |
| ODESZA               | 2014-12-30 01:52:43 |
| Radiohead            | 2014-05-13 21:03:06 |
| Kendrick Lamar       | 2015-01-17 23:39:01 |
| Skrillex             | 2014-04-12 12:04:00 |

Exciting. I have been listening to these artists for more than 10 years.

How about top albums? This is harder to answer with this dataset because there is no play count for each album so we can only estimate based on play counts for each tracks in each album. You can skew this data if you just play one song multiple times. (There might be a way but the SQL is too annoying to conjure.)

```sql
select date(min(timestamp)) as ts_date, count() as album_count, album_name, artist_name from spotify_history_view group by artist_name, album_name order by album_count desc limit 20;
```

|  ts_date   | album_count |                album_name                |    artist_name    |
|------------|------------:|------------------------------------------|-------------------|
| 2019-03-21 | 1225        | Hi This Is Flume                         | Flume             |
| 2016-12-02 | 1167        | W:/2016ALBUM/                            | deadmau5          |
| 2017-06-04 | 1098        | hopeless fountain kingdom                | Halsey            |
| 2014-03-17 | 885         | If You Wait                              | London Grammar    |
| 2015-09-13 | 885         | Oh Wonder                                | Oh Wonder         |
| 2014-05-01 | 794         | No Mythologies to Follow (Deluxe)        | MØ                |
| 2015-08-03 | 790         | Down to Earth                            | Flight Facilities |
| 2015-09-21 | 787         | BADLANDS                                 | Halsey            |
| 2016-05-19 | 772         | The Life Of Pablo                        | Kanye West        |
| 2017-06-16 | 771         | Melodrama                                | Lorde             |
| 2015-01-20 | 757         | My Beautiful Dark Twisted Fantasy        | Kanye West        |
| 2015-01-30 | 741         | while(1<2)                               | deadmau5          |
| 2014-03-08 | 731         | True                                     | Avicii            |
| 2018-07-13 | 710         | where's the drop?                        | deadmau5          |
| 2014-03-17 | 699         | Settle                                   | Disclosure        |
| 2014-03-31 | 686         | Days Are Gone                            | HAIM              |
| 2019-03-29 | 679         | WHEN WE ALL FALL ASLEEP, WHERE DO WE GO? | Billie Eilish     |
| 2017-06-07 | 677         | I See You                                | The xx            |
| 2014-04-15 | 670         | > Album Title Goes Here <                | deadmau5          |
| 2016-09-16 | 661         | New Age \| Dark Age (Deluxe Version)     | Karma Fields      |


# After words

This dataset is basically Spotify Wrapped but better.

Spotify recently greatly reduced the amount of data they serve over the Web API.
They no longer serve metadata such as genres, BPM, etc.
My original plan was to write a function (either ClickHouse UDF or some JS) to fetch these metadata on-demand but that's no longer viable.

I might jot down the process of getting more metadata out of Spotify, MusicBrainz and Last.fm
in a future post. It is still in progress and involves setting up a MusicBrainz database locally, attempting to pglite and a bit of LLM wrangling. I also want to look into fuzzy text search.

I also have friends asking me how to make this. I might make a little client-side web app for people
to play with. DuckDB has a WASM version so this can all be done without a server.

In the mean time, I'd like to leave a toy app that generates wallpapers from album arts [here](https://album-art-wallpaper-gen.vercel.app/). ([source](https://github.com/lexffe/album-art-wallpaper-gen))


## N.B. I want to share this but nevermind

I had a toy site ready to go, that lets people go and query an anonymised version of the data, but
even then LLMs reminded me that AI giants and scrapers could just download the parquet file and
use it for commercial purposes. :/
