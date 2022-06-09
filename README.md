# Get New Recommendations

Returns 25 song recommendations based on the user's music attribute vector. 
 - 10 songs are completely attribute-based and do not account for genre
 - 10 songs use both attributes and the user's favorite genres
 - 5 songs use both attributes and the user's favorite artists

Returns top 25 songs in the United States if there is no logged-in user or if the user has not yet created a post. 

[link to source code](https://github.com/UM-EECS-441/musicsharingnetwork/blob/2fc63963e4739eb09c3650e23561bf5b719711e0/backend/routes/recommendations.py#L11)

Endpoint: `GET /recommendations/`

**Request Parameters**
| Key        | Location | Type   | Description      |
| ---------- | -------- | ------ | ---------------- |
| `username` | Session Cookie| String | Current User |

**Response Codes**
| Code              | Description            |
| ----------------- | ---------------------- |
| `200 OK`     | Success                |
| `400 Bad Request` | Invalid parameters     |

**Returns**

*If no user is logged in or no posts created by user*
| Key        | Location       | Type   | Description  |
| ---------- | -------------- | ------ | ------------ |
| `popular_songs` | JSON | List of Spotify Track IDs | Top 25 songs on Spotify in the United States |

*For logged-in users with 1 or more posts created*
| Key        | Location       | Type   | Description  |
| ---------- | -------------- | ------ | ------------ |
| `attribute_recommendations` | JSON |List of Spotify Track IDs | Attribute-based recommendations (random genres) |
| `genre_recommendations` | JSON | List of Spotify Track IDs | Attribute and genre-based recommendations based on user's favorite genres |
| `artist_recommendations` | JSON | List of Spotify Track ID | Attribute and artist-based recommendations based on user's favorite artists | 
| `attribute_error` | JSON | Dictionary | contains the average error % for each attribute between recommendation and the user's attribute vector. | 

**Example**
~~~ 
curl -b cookies.txt -c cookies.txt -X GET https://OUR_SERVER/recommendations/'

{
    "attribute_error": {
        "acousticness": 0.1342,
        "danceability": 0.2567,
        "energy": 0.1144,
        "instrumentalness": 0.021,
        "liveness": 0.0324,
        "loudness": 0.0084,
        "speechiness": 0.0528,
        "tempo": 0.0446,
        "valence": 0.1538
    },
    "attribute_recommendations": [
        "spotify:track:3joo84oco9CD4dBsKNWRRW",
        "spotify:track:5DxlyLbSTkkKjJPGCoMo1O",
        ...
    ],
    "genre_recommendations": [
        "spotify:track:4MzXwWMhyBbmu6hOcLVD49",
        "spotify:track:0bYg9bo50gSsH3LtXe2SQn",
        ...
    ],
    "artist_recommendations": [
        "spotify:track:2EjXfH91m7f8HiJN1yQg97",
        "spotify:track:5o8EvVZzvB7oTvxeFB55UJ",
        ...
    ],
    "url": "/recommendations/"
}
~~~

## Users API

![Signup](https://eecs441.eecs.umich.edu/img/template/Signup.png)

These endpoints update and retrieve data from the Users collection in our database. The Users API handles:

* account creation [(link to code)](https://github.com/UM-EECS-441/musicsharingnetwork/blob/e6d037f68f07b927f444d800e479d03fe982a5c0/backend/routes/users.py#L15)
* account updates [(link to code)](https://github.com/UM-EECS-441/musicsharingnetwork/blob/e6d037f68f07b927f444d800e479d03fe982a5c0/backend/routes/users.py#L66)
* session authentication [(link to code)](https://github.com/UM-EECS-441/musicsharingnetwork/blob/e6d037f68f07b927f444d800e479d03fe982a5c0/backend/routes/users.py#L91)
* user profile retrieval [(link to code)](https://github.com/UM-EECS-441/musicsharingnetwork/blob/e6d037f68f07b927f444d800e479d03fe982a5c0/backend/routes/users.py#L129)

## Get Recommendations

This endpoint runs our recommendation algorithm based on a user's profile. This section provides a high-level overview of our algorithm. For a detailed explanation of our algorithm, please see [here](https://github.com/UM-EECS-441/musicsharingnetwork/wiki/Recommendation-Algorithm).

In order to run the algorithm, the server retrieves a user's attribute vector, genre vector, and artist vector from the database [(link to code)](https://github.com/UM-EECS-441/musicsharingnetwork/blob/e6d037f68f07b927f444d800e479d03fe982a5c0/backend/routes/recommendations.py#L64).

* Attribute Vector - Vector that tracks a user's sentiment towards specific song attributes (the specific attributes are listed in [Spotify Audio Features](https://github.com/UM-EECS-441/musicsharingnetwork/wiki/Backend-Architecture#spotify-audio-features))
* Genre Vector - Vector that tracks a user's sentiment towards specific genres
* Artist Vector - Vector that tracks a user's sentiment towards specific artists

The server then constructs a query vector that targets the song attributes that the user most prefers. This query vector is then used to call the [Spotify Recommendation Seed API](https://github.com/UM-EECS-441/musicsharingnetwork/wiki/Backend-Architecture#spotify-recommendation-seed) to search Spotify's library for songs that meet our desired characteristics. We query this API three times [(link to code)](https://github.com/UM-EECS-441/musicsharingnetwork/blob/e6d037f68f07b927f444d800e479d03fe982a5c0/backend/routes/recommendations.py#L11):

* Using random genres as seeds
* Using user's favorite genres as seeds
* Using user's favorite artists as seeds

The results of these queries are then returned to the frontend application to display as recommendations to the user. 

## Spotify Recommendation Seed

The Spotify Recommendation Seed API is how we search Spotify's library based on song attributes. The query takes in a list of target attributes (our query vector) as well as seeds to limit or expand the scope of the query. Additional documentation on this API can be found [here](https://developer.spotify.com/documentation/web-api/reference/browse/get-recommendations/).

## Create Posts

This endpoint is responsible for storing a new post in the database as well as classifying the sentiment of that post and updating the user's vectors accordingly. When a post is created, our [sentiment analysis model](https://github.com/UM-EECS-441/musicsharingnetwork/wiki/Backend-Architecture#sentiment-analysis-model) analyzes the content of the post and assigns it a sentiment score between -1 and 1 [(link to code)](https://github.com/UM-EECS-441/musicsharingnetwork/blob/e6d037f68f07b927f444d800e479d03fe982a5c0/backend/routes/posts.py#L34). We then use the [Spotify Audio Features API](https://github.com/UM-EECS-441/musicsharingnetwork/wiki/Backend-Architecture#spotify-audio-features) to get the attributes, genre, and artist of the song. We use this information along with the sentiment score to update the user's attribute vector, genre vector, and artist vector accordingly [(link to code)](https://github.com/UM-EECS-441/musicsharingnetwork/blob/e6d037f68f07b927f444d800e479d03fe982a5c0/backend/routes/posts.py#L145).

## Get/View Posts

This endpoint is used to display posts on a user's timeline or on a user's profile page. The server retrieves relevant posts from the database and returns them in chronological order for the user to view and interact with [(link to code)](https://github.com/UM-EECS-441/musicsharingnetwork/blob/e6d037f68f07b927f444d800e479d03fe982a5c0/backend/routes/posts.py#L58).

## User-to-User Recommendations/Direct Messages API

These endpoints update and retrieve data from the Messages collection in our database. The Messages API handles:

* creating new conversations/sending messages [(link to code)](https://github.com/UM-EECS-441/musicsharingnetwork/blob/e6d037f68f07b927f444d800e479d03fe982a5c0/backend/routes/messages.py#L10)
* viewing list of existing conversations [(link to code)](https://github.com/UM-EECS-441/musicsharingnetwork/blob/e6d037f68f07b927f444d800e479d03fe982a5c0/backend/routes/messages.py#L95)
* viewing an entire conversation [(link to code)](https://github.com/UM-EECS-441/musicsharingnetwork/blob/e6d037f68f07b927f444d800e479d03fe982a5c0/backend/routes/messages.py#L69)
# Third-Party SDKs

Our front end is written entirely in Swift and does not use any third-party SDKs. However, we do use [Spotify's Web API](https://developer.spotify.com/documentation/web-api/).

## Spotify Web API

We use [Spotify's Web API](https://developer.spotify.com/documentation/web-api/) on the front end for two purposes. One is to get information about a song, and the other is to search for songs.

### Authorization

Spotify requires us to obtain authorization before using the Web API. Since we do not interact with users' Spotify accounts, we follow the [Client Credentials Flow](https://developer.spotify.com/documentation/general/guides/authorization-guide/#client-credentials-flow).

Code:
* [Obtain authorization](https://github.com/UM-EECS-441/musicsharingnetwork/blob/e6d037f68f07b927f444d800e479d03fe982a5c0/frontend/music-sharing-network/SpotifyWebAPI.swift#L38)

### Song Information

When a song is displayed in our app, we show the name of the song, the name of the song's artist, the title of the album containing the song, and the image (album cover) for that album. For this, we use Spotify's [Get a Track](https://developer.spotify.com/documentation/web-api/reference/tracks/get-track/) ednpoint.

Code:
* [Get a track](https://github.com/UM-EECS-441/musicsharingnetwork/blob/e6d037f68f07b927f444d800e479d03fe982a5c0/frontend/music-sharing-network/SpotifyWebAPI.swift#L137)
* [Parse JSON response and load image for a single track](https://github.com/UM-EECS-441/musicsharingnetwork/blob/e6d037f68f07b927f444d800e479d03fe982a5c0/frontend/music-sharing-network/SpotifyWebAPI.swift#L92)

### Song Search

When a user wants to make a post in our app, they are prompted to select a song to include in the post. We use Spotify's [Search](https://developer.spotify.com/documentation/web-api/reference/search/search/) endpoint to let the user search for a song.

Code:
* [Search for a song](https://github.com/UM-EECS-441/musicsharingnetwork/blob/e6d037f68f07b927f444d800e479d03fe982a5c0/frontend/music-sharing-network/SpotifyWebAPI.swift#L182)
* [Parse JSON response and load image for a single track](https://github.com/UM-EECS-441/musicsharingnetwork/blob/e6d037f68f07b927f444d800e479d03fe982a5c0/frontend/music-sharing-network/SpotifyWebAPI.swift#L92)
