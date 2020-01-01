# E-02 用程式回覆-寫入Firebase


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
// 設定自己的firebaseConfig連線資訊.
// 在 "Project Overview" -> "專案設定"中
//-----------------------------------------------------------
var firebase = require('firebase');
 
var firebaseConfig = {
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
    // 處理使用者詢問推薦的意圖
    //------------------------------------	  
    function ask_suggestion(agent) {  
        //取得參數    
        const body = JSON.stringify(request.body);
        const queryText = request.body.queryResult.queryText;
        const action = request.body.queryResult.action;
        //使用者資訊已不支援 
        //const userId = request.body.originalDetectIntentRequest.payload.data.source.userId;
        //const timestamp = request.body.originalDetectIntentRequest.payload.data.timestamp;
        const parameters = request.body.queryResult.parameters;                
        const meal = parameters.meal;
        const food = parameters.food;      
        
        //回應參數   
        agent.add('請求封包內文:' + body);      
        agent.add('傳入訊息:' + queryText);
        agent.add('動作:' + action);
        agent.add('使用者ID:' + userId);
        agent.add('時間戳記:' + timestamp);      
        agent.add('餐別參數:' + meal);      
        agent.add('食物別參數:' + food); 
      
        //寫資料到firebase的資料庫中(自動生成ID)     
        return firebase.database().ref('logs')
            .push({
                recordDatetime:timestamp,
                user:userId,
                meal:meal,
                food:food,
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
    // 設定對話中各個意圖的函式對照
    //------------------------------------	
    let intentMap = new Map();

    intentMap.set("ask suggestion", ask_suggestion); 
  
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
