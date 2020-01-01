# D-03 用程式回覆-寫入Firebase


## 機器人設計
```
|__ Intents
|      |__ ask today's special (詢問今日特餐)
|      |        .
|      |        |__ Fulfillment
|      |                |__ Enable webhook call for this intent (開啟)
|      |
|      |__ ask chef's recommendation (詢問主廚推薦)
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

## 取得firebaseConfig
```
在Project Overview的"Settings"中, 選擇"專案設定", 
再選"將 Firebase 加入您的網路應用程式":
 -> 將 Firebase 新增至應用程式
 -> 選取平台即可開始使用(Web)
 -> 得到firebaseConfig資料. 
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
// 設定自己的firebaseConfig連線資訊:
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
        //使用者資訊已不支援 
        //const userId = request.body.originalDetectIntentRequest.payload.data.source.userId;
        //const timestamp = request.body.originalDetectIntentRequest.payload.data.timestamp;
        const parameters = request.body.queryResult.parameters;                
        const special = parameters.special;
        
        //回應參數   
        agent.add('請求封包內文:' + body);      
        agent.add('傳入訊息:' + queryText);
        agent.add('動作:' + action);
        agent.add('使用者ID:' + userId);
        
        //寫資料到firebase的資料庫中(自動生成ID)     
        return firebase.database().ref('special')
            .push({
                recordDatetime:timestamp,
                user:userId,
                intent:special,
                query:queryText
            }).once('value')
            .then(snapshot => {
                agent.add('Ok, 新增完成');
            })
            .catch(error => {
                agent.add('新增失敗' + error);
            });              
    }  

    //------------------------------------
    // 處理使用者詢問主廚推薦的意圖
    //------------------------------------	  
    function ask_chef_recommendation(agent) {      
        //取得參數    
        const body = JSON.stringify(request.body);
        const queryText = request.body.queryResult.queryText;
        const action = request.body.queryResult.action;
        //使用者資訊已不支援 
        //const userId = request.body.originalDetectIntentRequest.payload.data.source.userId;
        //const timestamp = request.body.originalDetectIntentRequest.payload.data.timestamp;
        const parameters = request.body.queryResult.parameters;                
        const recommendation = parameters.recommendation;
        
        //回應參數   
        agent.add('請求封包內文:' + body);      
        agent.add('傳入訊息:' + queryText);
        agent.add('動作:' + action);
        agent.add('參數:' + recommendation);
      
        //寫資料到firebase的資料庫中(自己指定ID)    
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
  "description": "更改了dialogflow-fulfillment版本, 也增加firebase套件",
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
