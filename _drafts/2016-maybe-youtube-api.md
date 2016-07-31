default_scope { order(name: :asc) }
  

	
//채널아이디
  https://www.googleapis.com/youtube/v3/channels?part=statistics&id=UCmvv07u2NeZTTf1yCBos9XQ&key=AIzaSyCX3xVj7kjuXeqPbql40JoFZkJ_qEZS0h4

//유저네임
https://www.googleapis.com/youtube/v3/channels?part=statistics&forUsername=KIMDAXworld&key=AIzaSyCX3xVj7kjuXeqPbql40JoFZkJ_qEZS0h4

  {
  "kind": "youtube#channelListResponse",
  "etag": "\"mPrpS7Nrk6Ggi_P7VJ8-KsEOiIw/3Fsy0eHk2Cyb3vr9UkTt5nha5iY\"",
  "pageInfo": {
    "totalResults": 1,
    "resultsPerPage": 1
  },
  "items": [
    {
      "kind": "youtube#channel",
      "etag": "\"mPrpS7Nrk6Ggi_P7VJ8-KsEOiIw/FvhqnWNFHvbDUnWuKHDmiPBercU\"",
      "id": "UCmvv07u2NeZTTf1yCBos9XQ",
      "statistics": {
        "viewCount": "4515883",
        "commentCount": "0",
        "subscriberCount": "96519",
        "hiddenSubscriberCount": false,
        "videoCount": "60"
      }
    }
  ]

  https://developers.google.com/youtube/v3/docs/channels/list

  https://console.developers.google.com/apis/credentials?project=api-project-953814092259

  