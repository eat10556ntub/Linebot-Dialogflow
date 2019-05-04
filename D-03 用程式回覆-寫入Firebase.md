# D-03 用程式回覆-寫入Firebase


## 機器人設計
```
|__ Intents
|      |__ ask today's special
|      |        .
|      |        |__ Fulfillment
|      |                |__ Enable webhook call for this intent (開啟)
|      |
|      |__ ask chef's recommendation
.               .
.               |__ Fulfillment
.                       |__ Enable webhook call for this intent (開啟)
.
.
.
|__ Fulfillment     
       |__ Inline Editor (啟動 ENABLED)    
                |__ index.js      
                |__ package.json  
```

## Firebase設定
```
在Firebase的控制台, 選擇 "Realtime Database"(藍色) 的 "查看->", "規則". 修改如下:

{
    "rules": {
        ".read": true,
        ".write": true
    }
}
```

## (1) index.js

``` js
'use strict';
 
//------------------------------------
// 載入必要模組
//------------------------------------ 
const functions = require('firebase-functions');
const {WebhookClient} = require('dialogflow-fulfillment');
const {Text, Card, Image, Suggestion, Payload} = require('dialogflow-fulfillment'); 

//-----------------------------------------------------------
// *設定自己的firebase連線:
//   1. 以下資料在Project Overview的"Settings"中,
//   2. 選擇"專案設定", 再選"將 Firebase 加入您的網路應用程式":
//        2-1. 將 Firebase 新增至應用程式
//        2-2. 選取平台即可開始使用(Web)
//        2-3. 將得到以下firebaseConfig的資料. 
//-----------------------------------------------------------
var firebase = require('firebase');
 
const firebaseConfig = {
    apiKey: "(填入自己的資料)",
    authDomain: "(填入自己的資料)",
    databaseURL: "(填入自己的資料)",
    projectId: "(填入自己的資料)",
    storageBucket: "(填入自己的資料)",
    messagingSenderId: "(填入自己的資料)",
    appId: "(填入自己的資料)"
};
  
firebase.initializeApp(firebaseConfig);
//-----------------------------------------------------------

process.env.DEBUG = 'dialogflow:debug';
 
exports.dialogflowFirebaseFulfillment = functions.https.onRequest((request, response) => {
    //------------------------------------
    // 建立一個對話代理人
    //------------------------------------	
    const agent = new WebhookClient({ request, response });

    //------------------------------------
    // 處理使用者詢問今日特餐的意圖
    //------------------------------------	  
    function ask_today_special(agent) {  
        //取得參數    
        const body = JSON.stringify(request.body);
        const queryText = request.body.queryResult.queryText;
        const action = request.body.queryResult.action;
        const userId = request.body.originalDetectIntentRequest.payload.data.source.userId;
        const timestamp = request.body.originalDetectIntentRequest.payload.data.timestamp;
        const parameters = request.body.queryResult.parameters;                
        const special = parameters.special;
        
        //回應參數   
        agent.add('請求封包內文:' + body);      
        agent.add('傳入訊息:' + queryText);
        agent.add('動作:' + action);
        agent.add('使用者ID:' + userId);
        agent.add('時間戳記:' + timestamp);      
        agent.add('參數:' + special);
      
        //傳送自訂訊息(目前只能有一個自訂訊息)
        var lineMessage = {
            "type": "sticker",
            "packageId": "1",
            "stickerId": "1"
        };
        
        var payload = new Payload('LINE', lineMessage, {sendAsMessage: true});
        agent.add(payload);    
      
        //------------------------------------------------
        // 寫資料到firebase的資料庫中(自動生成ID)
        //------------------------------------------------        
        return firebase.database().ref('special')
            .push({
                recordDatetime:timestamp,
                user:userId,
                intent:special,
                query:queryText
            })
            .then(snapshot => {
                agent.add('Ok, 新增完成');
            })
            .catch(error => {
                agent.add('新增失敗' + error);
            });
        //------------------------------------------------        
    }  

    //------------------------------------
    // 處理使用者詢問主廚推薦的意圖
    //------------------------------------	  
    function ask_chef_recommendation(agent) {      
        //取得參數    
        const body = JSON.stringify(request.body);
        const queryText = request.body.queryResult.queryText;
        const action = request.body.queryResult.action;
        const userId = request.body.originalDetectIntentRequest.payload.data.source.userId;
        const timestamp = request.body.originalDetectIntentRequest.payload.data.timestamp;
        const parameters = request.body.queryResult.parameters;                
        const recommendation = parameters.recommendation;
        
        //回應參數   
        agent.add('請求封包內文:' + body);      
        agent.add('傳入訊息:' + queryText);
        agent.add('動作:' + action);
        agent.add('使用者ID:' + userId);
        agent.add('時間戳記:' + timestamp);      
        agent.add('參數:' + recommendation);

        //傳送自訂訊息(目前只能有一個自訂訊息)
        var lineMessage = {
            "type": "template",
            "altText": "這是一個Carousel圖文選單樣板",
            "template": {
                "type": "carousel",
                "columns":[{
                    "thumbnailImageUrl": "https://tomlin-app-1.herokuapp.com/imgs/f01.jpg",
                    "imageBackgroundColor": "#FFFFFF",
                    "title": "麵類",
                    "text": "請選擇麵類餐點",
                    "actions": [{
                        "type": "message",
                        "label": "想吃牛肉麵",
                        "text": "牛肉麵"
                    },
                    {
                        "type": "message",
                        "label": "想吃大魯麵",
                        "text": "大魯麵"
                    },
                    {
                        "type": "message",
                        "label": "想吃蕃茄麵",
                        "text": "蕃茄麵"
                    }]
                },
                {
                    "thumbnailImageUrl": "https://tomlin-app-1.herokuapp.com/imgs/f05.jpg",
                    "imageBackgroundColor": "#FFFFFF",
                    "title": "飯類",
                    "text": "請選擇飯類餐點",
                    "actions": [{
                        "type": "message",
                        "label": "想吃蛋炒飯",
                        "text": "蛋炒飯"
                    },
                    {
                        "type": "message",
                        "label": "想吃燴飯",
                        "text": "燴飯"
                    },
                    {
                        "type": "message",
                        "label": "想吃海鮮炒飯",
                        "text": "海鮮炒飯"
                    }]
                }],
                "imageAspectRatio": "square",
                "imageSize": "cover"
            }
        };
        
        var payload = new Payload('LINE', lineMessage, {sendAsMessage: true});
        agent.add(payload);
      
        //------------------------------------------------
        // 寫資料到firebase的資料庫中(自己指定ID)
        //------------------------------------------------        
        return firebase.database().ref('recommendation/' + timestamp)
            .set({
                recordDatetime:timestamp,
                user:userId,
                intent:recommendation,
                query:queryText
            })
            .then(snapshot => {
                agent.add('Ok, 新增完成');
            })
            .catch(error => {
                agent.add('新增失敗' + error);
            });
        //------------------------------------------------           
    } 
  
    //------------------------------------
    // 設定對話中各個意圖的函式對照
    //------------------------------------	
    let intentMap = new Map();

    intentMap.set("ask today's special", ask_today_special); 
    intentMap.set("ask chef's recommendation", ask_chef_recommendation);
  
    agent.handleRequest(intentMap);
    //------------------------------------	
});
```


## (2) package.json

``` json
{
  "name": "dialogflowFirebaseFulfillment",
  "description": "更改了dialogflow-fulfillment版本",
  "version": "0.0.1",
  "private": true,
  "license": "Apache Version 2.0",
  "author": "Google Inc.",
  "engines": {
      "node": "8"
  },
  "scripts": {
      "start": "firebase serve --only functions:dialogflowFirebaseFulfillment",
      "deploy": "firebase deploy --only functions:dialogflowFirebaseFulfillment"
  },
  "dependencies": {
      "actions-on-google": "^2.2.0",
      "firebase-admin": "^5.13.1",
      "firebase-functions": "^2.0.2",
      "dialogflow": "^0.6.0",
      "dialogflow-fulfillment": "^0.6.1",
      "firebase": "^5.11.1"
  }
}
```
