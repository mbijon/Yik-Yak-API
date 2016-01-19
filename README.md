#YikYak Beta API

##Rate Limiting
The API is rate limited to 30 requests per 60 seconds. Breaching this limit will cause the server to return an `HTTP 429` code, so take it slow.

##Objects


###Yak
JSON object representing a Yak (known as messages in the API)

|Key|Value|Notes
|---|---|---
|`canDownVote`|bool
|`canReply`|bool|
|`canReport`|int|`0` or `1`
|`canUpVote`|bool
|`canVote`|bool
|`comments`|int|Number of comments
|`deliveryID`|int|Inverse position in list (i.e. 0 at bottom)
|`gmt`|float
|`handle`|???|Always seems to be None
|`hidePin`|int|`0` or `1`
|`latitude`|float|Yak latitude to 2dp
|`liked`|int|`0` or `1`
|`location`|dict|Appears to be always empty
|`locationDisplayStyle`|int|Always `0`
|`locationName`|string
|`longitude`|float|Yak long to 2 decimal places
|`message`|string|Yak contents
|`messageID`|string|ID of Yak
|`numberOfLikes`|int|Current Yak score
|`posterID`|string|
|`readOnly`|int|`0` or `1`
|`reyaked`|int|`0` or `1`
|`score`|float
|`time`|timestamp|`YYYY-MM-DD HH:MM:SS`
|`type`|int|`0` = text; `6` = image
|**Image Yaks Only**
|`expandInFeed`|int|Always `1`
|`imageHeight`|int|Pixel height
|`imageWidth`|int|Pixel width
|`thumbNailUrl`|url|Link to thumb
|`url`|url|Link to full size

###Comment
|Key|Value|Notes
|---|---|---
|`backID`|string|Icon background
|`comment`|string|Comment body
|`commentID`|string
|`deliveryID`|int
|`gmt`|float
|`isDeleted`|bool
|`liked`|int|`0` or `1`
|`messageID`|string|ID of parent Yak
|`numberOfLikes`|int
|`overlayID`|string|Icon
|`posterID`|string
|`time`|timestamp|`YYYY-MM-DD HH:MM:SS`

##Authenticating
###Initialise Pairing
`POST https://beta.yikyak.com/api/auth/initPairing`

To login to the web API we need a 6 digit PIN generated by the mobile app. If you know your Yik Yak user ID, we avoid having to manually check the app by automatically retrieving the PIN from the API.

**Request Body**
```
{
    'userID': 'XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX',
}
```

**Response**
```
{
    'pin': '110786'
    'ttl': 60,
}
```

###Pair
`POST https://beta.yikyak.com/api/auth/pair`  

To perform any further actions with the Web Beta API, we need an authentication token which can be retrieved using our country code, phone number and YikYak auth PIN (either from the mobile app, or retrieved from `initPairing`).

**Request Headers**  
```
{
    'Referer': 'https://beta.yikyak.com/'
}
```

**Request Body** (JSON)  
```
{
    'countryCode': "ABC",
    'phoneNumber': "1234567890",
    'pin': "123456",
}
```

**Response** (JSON)  
Authentication token string

##Authenticated Operations

Once we have retrieved the authentication token, we can interact with the API. For all authenticated operations, the following headers must be sent:

```
{
    'Referer': 'https://beta.yikyak.com/',
    'x-access-token': <auth_token>,
}
```
 


###Yakker
`GET https://beta.yikyak.com/api/proxy/v1/yakker`  

**Request Headers**
```
{
    'Referer': 'https://beta.yikyak.com/',
    'x-access-token': <auth_token>,
}
```

**Response**
```
{
    'amplitudeID': <string>,
    'myHerd': {
        'eligible': <bool>,
        'enabled': <bool>,
        'lat': <string>,
        'long': <string>,
        'name': <string>,
    },
    'userID': <string>,
    'verificationStatus': {
        'forceVerification': <bool>,
        'isVerified': <bool>,
    },
    'yakarma': 
}
```


###New / Hot Yak Feed
`GET https://beta.yikyak.com/api/proxy/v1/messages/all/new`  
`GET https://beta.yikyak.com/api/proxy/v1/messages/all/hot`  

**Request Headers**
```
{
    'Referer': 'https://beta.yikyak.com/',
    'x-access-token': <auth_token>,
}
```

**Request Query String**
```
userLat=0
userLong=0
lat=<latitude>
long=<longitude>
myHerd=0
```

`userLat` and `userLong` do not appear to be required. `lat` and `long` is the co-ordinates to retrieve Yaks from.

**Response**
Returns up to 200(?) Yak objects

###Yak Details
`GET https://beta.yikyak.com/api/proxy/v1/messages/<yak_id>`

> **Note**
> The `/` in the Yak's ID must be made URL-safe (i.e. converted to `%2F`)
> `R/xxxxxxxxxxxxxxxxxxxxxxxxxxxxx` → `R%2Fxxxxxxxxxxxxxxxxxxxxxxxxxxxxx`

**Request Query String**  
These values must be sent, but appear to have no effect.  
```
userLat=0.0
userLong=0.0
myHerd=0
```

**Request Headers**  
```
{
    'Referer': 'https://beta.yikyak.com/',
    'x-access-token': <auth_token>,
}
```

**Response**  
Returns a JSON Yak object (see above)


###Upvoting / Downvoting a Yak

`PUT https://beta.yikyak.com/api/proxy/v1/messages/<yak_id>/upvote`  
`PUT https://beta.yikyak.com/api/proxy/v1/messages/<yak_id>/downvote`  

**Request Query String**  
These values must be sent, but appear to have no effect.  
```
userLat=0.0
userLong=0.0
myHerd=0
```

**Request Headers**  
```
{
    'Referer': 'https://beta.yikyak.com/',
    'x-access-token': <auth_token>,
}
```

**Response**  
JSON parse fails on the response. Probably a malformed Yak object.
