## 升级闹铃

### 天气预报
喜马拉雅有北京的天气预报的主播，可以通过拉取喜马拉雅的数据，实现每天早上，闹铃自动播放天气预报。

获取播放列表：

https://www.ximalaya.com/revision/play/album?albumId=22689810&pageNum=1&sort=-1&pageSize=1


接口数据如下：

```JSON
{
    "ret": 200,
    "msg": "声音播放数据",
    "data": {
        "uid": 0,
        "albumId": 22689810,
        "sort": 1,
        "pageNum": 1,
        "pageSize": 5,
        "tracksAudioPlay": [
            {
                "index": 5,
                "trackId": 179407457,
                "trackName": "20190427早间 北京-降雨降温大风 晓丹",
                "trackUrl": "/toutiao/22689810/179407457",
                "trackCoverPath": "//imagev2.xmcdn.com/group58/M0B/3E/85/wKgLc1yvEhvQMF_OAB79PHJwOoQ209.jpg",
                "albumId": 22689810,
                "albumName": "中国天气show|北京天气",
                "albumUrl": "/toutiao/22689810/",
                "anchorId": 7396493,
                "canPlay": true,
                "isBaiduMusic": false,
                "isPaid": false,
                "duration": 60,
                "src": "http://aod.tx.xmcdn.com/group58/M07/DE/13/wKgLc1zDj93Dvc4jAAdzvJEZxrM644.m4a",
                "hasBuy": true,
                "albumIsSample": false,
                "sampleDuration": 0,
                "updateTime": "8小时前",
                "createTime": "9小时前",
                "isLike": false,
                "isCopyright": true,
                "firstPlayStatus": true
            },
            {
                // ...
            },
            {
                // ...
            }
        ],
        "hasMore": true
    }
}
```

其中，`data.tracksAudioPlay[0].src` 便是我们需要的音频。
http://audio.xmcdn.com/group58/M07/DE/13/wKgLc1zDj93Dvc4jAAdzvJEZxrM644.m4a

