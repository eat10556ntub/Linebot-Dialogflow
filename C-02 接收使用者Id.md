# C-02 接收使用者Id


## 在DialogFlow的設定
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
              |
              |__ BASIC AUTH  (Enter username) user    (Enter password) abcdabcdabcd                  
```


## 程式結構
```
 |__ <app>
       |__ index.js
       |__ package.json
```


## 追加auth-header外掛
```
npm install auth-header --save
```

## (1) index.js

``` javascript
"use strict";

const express = require('express');
const authorization = require('auth-header');
const { WebhookClient } = require('dialogflow-fulfillment');
const app = express();


// 如果未通過存在Header中的auth
function fail(res) {
    res.set('WWW-Authenticate', authorization.format('Basic'));
    res.status(401).send();
}

app.post('/dialogflow', express.json(), (req, res) => {
    //------------------------------------
    // 處理Dialogflow傳來的認證
    //------------------------------------      
    // 取得認證資料
    var auth = authorization.parse(req.get('authorization'));
 
    // 如果沒有認證資料
    if (auth.scheme !== 'Basic') {
        return fail(res);
    }
 
    // 取得認證內容
    var [user, password] = Buffer(auth.token, 'base64').toString().split(':', 2);
 
    // 檢查認證內容
    if (user !== 'user' || password !== 'abcdabcdabcd') {
        return fail(res);
    }    
    
    //------------------------------------
    // 處理請求/回覆的Dialogflow代理人
    //------------------------------------  
    const agent = new WebhookClient({request: req, response: res})

    console.log('觀察以下物件********************');
    console.log(req.headers);
    console.log(JSON.stringify(req.body));
    console.log('*******************************');

    //------------------------------------
    // 處理歡迎意圖
    //------------------------------------     
    function welcome(){
        agent.add('歡迎你!!!');

        agent.add('傳入訊息:'+req.body.queryResult.queryText);
        agent.add('傳入參數:'+req.body.queryResult.parameters);
        // 以下payload的內容已經Google Dialogflow已不再傳送
        //agent.add('使用者的LineId:'+req.body.originalDetectIntentRequest.payload.data.source.userId);
        //agent.add('timestamp:'+req.body.originalDetectIntentRequest.payload.data.timestamp);        
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

``` javascript
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



## 上傳至Heroku
```
(1) (網頁)已下載及安裝Node.js
(2) 已安裝Heroku CLI, npm install heroku -g
(3) (網頁https://git-scm.com/downloads)已下載及安裝git CLI
(4) (網頁)已登入Github
(5) (網頁)已登入Line Developer
(6) (網頁)已登入Heroku

(假設程式在D槽)
(7) d:
    cd app
(8) heroku login -i
(9) git config --global user.email "自己在git的email帳號"
(10) git init
(11) heroku git:remote -a [Heroku上的應用程式名稱]
---------------------------------------------------
(12) git add .
(13) git commit -am "myApp"
(14) git push heroku master -f
---------------------------------------------------
(15) 查看heroku終端機畫面
     heroku logs --tail
```
