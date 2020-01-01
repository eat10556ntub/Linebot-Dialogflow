# E-03 用程式回覆-多個DB動作


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
        agent.add('時間戳記:' + timestamp); 
        agent.add('使用者ID:' + userId); 
        agent.add('餐別參數:' + meal);      
        agent.add('食物別參數:' + food);           
        agent.add('傳入訊息:' + queryText);     
      
        //寫資料到firebase的資料庫中的函式(自動生成ID)  
        var write_logs = function(timestamp, userId, meal, food, queryText){
            return firebase.database().ref('logs').push({
                recordDatetime:timestamp,
                user:userId,
                meal:meal,
                food:food,
                query:queryText              
            }).once('value');
        };   

        //讀取firebase資料庫中的函式
        var read_food = function(id){
            return firebase.database().ref('foods/' + id).once('value');
        };
      
        //循序執行2個資料庫動作
        return write_logs(timestamp, userId, meal, food, queryText)  //第1個DB動作
        .then((snapshot1) => {                        
            agent.add('已寫入log'); 
            return read_food('005');  //第2個DB動作
        })          
        .then((snapshot2) => {           
            // 取出快照內容
            const data = snapshot2.val();

            agent.add('食物名稱: ' + data.name);
            agent.add('價格: ' + data.price.toString());  //數字轉為文字輸出
            agent.add('描述: ' + data.description);
                  
            //傳送自訂訊息(目前只能有一個自訂訊息)
            var lineMessage = {
                "type": "image",
                "originalContentUrl": data.picture,
                "previewImageUrl": data.picture
            };
        
            var payload = new Payload('LINE', lineMessage, {sendAsMessage: true});
            agent.add(payload); 
        })  
        .catch(error => {
            agent.add('沒有推薦, 動作失敗! ' + error);
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
