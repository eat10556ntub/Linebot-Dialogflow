# C-01 以Heroku程式處理, 在Dialogflow設定


```
|__ Intents
|      |__ Default Welcome Intent
.               .
.               |__ Fulfillment
.                       |__ Enable webhook call for this intent (開啟)
.
.
|__ Fulfillment     
       |__ Webhook (啟動 ENABLED)    
              |__ URL*  -->  https://(自己的應用程式名稱).herokuapp.com/dialogflow              
```

## 程式結構
```
 |__ <app>
       |__ index.js
       |__ package.json
```

## (1) index.js

```javascript
"use strict";

const express = require('express')
const { WebhookClient } = require('dialogflow-fulfillment')
const app = express()

app.post('/dialogflow', express.json(), (req, res) => {
    //------------------------------------
    // 處理請求/回覆的Dialogflow代理人
    //------------------------------------  
    const agent = new WebhookClient({request: req, response: res})


    //------------------------------------
    // 處理歡迎意圖
    //------------------------------------     
    function welcome(){
        agent.add('歡迎你!!!');
    }

    //------------------------------------
    // 設定對話中各個意圖的函式對照
    //------------------------------------
    let intentMap = new Map();
    intentMap.set('Default Welcome Intent', welcome);
    agent.handleRequest(intentMap);
})


//----------------------------------------
// 監聽3000埠號, 
// 或是監聽Heroku設定的埠號
//----------------------------------------
var server = app.listen(process.env.PORT || 3000, function() {
    const port = server.address().port;
    console.log("正在監聽埠號:", port);
});

```


## (2) package.json 

```javascript
{
  "name": "app10",
  "version": "0.0.0",
  "private": true,
  "main": "index.js",
  "scripts": {
    "start": "node ."
  },
  "dependencies": { 
    "actions-on-google": "^2.4.1",
    "body-parser": "^1.18.3",
    "cookie-parser": "~1.4.3",
    "debug": "~2.6.0",
    "express": "^4.16.4",
    "dialogflow": "^0.6.0",
    "dialogflow-fulfillment": "^0.5.0"
  }
}
```


### 上傳至Heroku
```
(1) (網頁)已下載及安裝Node.js
(2) 已安裝Heroku CLI, npm install heroku -g
(3) (網頁)已下載及安裝git CLI
(4) (網頁)已登入Github
(5) (網頁)已登入Line Developer
(6) (網頁)已登入Heroku
(7) heroku login -i
(8) git config --global user.email "自己在git的email帳號"
(9) git init
(10) heroku git:remote -a [Heroku上的應用程式名稱]
---------------------------------------------------
(11) git add .
(12) git commit -am "myApp"
(13) git push heroku master -f
```
