# D-05 用程式回覆-取出Firebase資料


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
    // 處理使用者詢問今日特餐的意圖
    //------------------------------------	  
    function ask_today_special(agent) {  
        return firebase.database().ref('foods/001').once('value')
            .then(snapshot => {
                if (!snapshot.exists()) {               
                    agent.add('沒有特餐資料');
                }else{
                    // 取出快照內容
                    const data = snapshot.val();

                    agent.add(data.name);
                    agent.add(data.price.toString());  //數字轉為文字輸出
                    agent.add(data.description);
                  
                    //傳送自訂訊息(目前只能有一個自訂訊息)
                    var lineMessage = {
                        "type": "image",
                        "originalContentUrl": data.picture,
                        "previewImageUrl": data.picture
                    };
        
                    var payload = new Payload('LINE', lineMessage, {sendAsMessage: true});
                    agent.add(payload);                                      
                }
            });
    }  

    //------------------------------------
    // 處理使用者詢問主廚推薦的意圖
    //------------------------------------	  
    function ask_chef_recommendation(agent) {      
        return firebase.database().ref('foods').orderByChild('price').once('value')
            .then(snapshot => {
                if (!snapshot.exists()) {               
                    agent.add('沒有主廚推薦資料');
                }else{
                    //取出快照內容
                    let key, data;
                  
                    snapshot.forEach(childSnapshot => {
                        key = childSnapshot.key;
                        data = childSnapshot.val();
                    });
                  
                    agent.add(data.name);
                    agent.add(data.price.toString());  //數字轉為文字輸出
                    agent.add(data.description);
                  
                    //傳送自訂訊息(目前只能有一個自訂訊息)
                    var lineMessage = {
                        "type": "image",
                        "originalContentUrl": data.picture,
                        "previewImageUrl": data.picture
                    };
        
                    var payload = new Payload('LINE', lineMessage, {sendAsMessage: true});
                    agent.add(payload);                                      
                }
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
