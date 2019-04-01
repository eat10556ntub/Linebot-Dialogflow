# 08 將收到的資料寫入Firebase


```
在Firebase的控制台, 選擇Realtime Database的 "查看->", "規則". 修改如下:

{
    "rules": {
        ".read": true,
        ".write": true
    }
}
```

## (1) index.js

```javascript
'use strict';
 
//------------------------------------
// 載入必要模組
//------------------------------------ 
const functions = require('firebase-functions');
const {WebhookClient} = require('dialogflow-fulfillment');
const {Card, Suggestion} = require('dialogflow-fulfillment');

process.env.DEBUG = 'dialogflow:debug';

//-----------------------------------------------------------
// 設定自己的firebase連線, 
//   以下資料在Project Overview的Settings中,
//   選擇"專案設定", 再選"將 Firebase 加入您的網路應用程式",
//   其中將有所有以下的資料. 
//-----------------------------------------------------------
var firebase = require('firebase');
 
var config = {
    apiKey: "(請填自己的firebase設定)",
    authDomain: "(請填自己的firebase設定)",
    databaseURL: "(請填自己的firebase設定)",
    projectId: "(請填自己的firebase設定)",
    storageBucket: "(請填自己的firebase設定)",
    messagingSenderId: "(請填自己的firebase設定)"
};
  
firebase.initializeApp(config);
//-----------------------------------------------------------

 
exports.dialogflowFirebaseFulfillment = functions.https.onRequest((request, response) => {
    //------------------------------------
    // 建立一個對話代理人
    //------------------------------------	
    const agent = new WebhookClient({ request, response });
  
    //------------------------------------
    // 處理使用者提供付款方式的意圖
    //------------------------------------	  
    function providing_payment(agent) {
        // 取得時間戳記
        var id = request.body.originalDetectIntentRequest.payload.data.timestamp;
        agent.add('timestamp:'+id);   
        
        // 取的使用者告訴機器人的參數
        const parameters = request.body.queryResult.parameters;        
        
        var type = parameters.type;
        var number = parameters.number;
        var payment = parameters.payment_type;

        // 回應取得資料
        agent.add('房型:'+type);
        agent.add('人數:'+number);
        agent.add('付款方式:'+payment);
        
        //------------------------------------------------
        // 寫資料到firebase的Realtime Database中
        //------------------------------------------------        
        return firebase.database().ref('students/' + id)
            .set({
                type:type,
                number:number,
                payment:payment
            })
            .then(snapshot => {
                agent.add(`Ok, 新增完成`);
            })
            .catch(error => {
                agent.add(`新增失敗`);
            });
        //------------------------------------------------             
    } 
    

    //------------------------------------
    // 設定對話中各個意圖的函式對照
    //------------------------------------	
    let intentMap = new Map();	
    intentMap.set('providing_payment', providing_payment); 
    agent.handleRequest(intentMap);
    //------------------------------------	
});
```


## (2) package.json

```javascript
{
    "name": "dialogflowFirebaseFulfillment",
    "description": "修改了dialogflow, dialogflow-fulfillment版本, 增加匯入firebase",
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
        "firebase": "^5.5.7"
    }
}
```
