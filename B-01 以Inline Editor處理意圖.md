# B-01 以Inline Editor處理意圖

## (1) 設計一個對話

```
假設這個代理人有3個意圖

Intents
   |__ Default Fallback Intent (機器人不懂使用者的意圖)
   |           |               
   |           |__  Fulfillment
   |                   |__ (啟動) Enable webhook call for this intent    
   |
   |
   |__ Default Welcome Intent (開始對話)
   |           |
   |           |__ Contexts
   |           |       |__ input context
   |           |       |         |__ (無)
   |           |       |
   |           |       |__ output context
   |           |                 |__ asking_name
   |           |
   |           |__ Fulfillment
   |                   |__ (啟動) Enable webhook call for this intent
   |
   |
   |__ Providing_name (使用者提供姓名)
   |           |
   |           |__ Contexts
   |           |       |__ input context
   |           |       |         |__ asking_name
   |           |       |
   |           |       |__ output context
   |           |                 |__ (無)
   |           |
   |           |__ Response
   |           |       |__ (啟動) Set this intent as end of conversation
   |           |                    
   |           |__ Fulfillment
   |                   |__ (啟動) Enable webhook call for this intent
   |		       
   .
   .
   |__ Fulfillment     
          |__ Inline Editor (啟動 ENABLED)    
                   |__ index.js (測試以下程式)	       
```

### (2) index.js

```javascript
'use strict';
 
//------------------------------------
// 載入必要模組
//------------------------------------ 
const functions = require('firebase-functions');
const {WebhookClient} = require('dialogflow-fulfillment');
const {Card, Suggestion} = require('dialogflow-fulfillment');

process.env.DEBUG = 'dialogflow:debug';
 
exports.dialogflowFirebaseFulfillment = functions.https.onRequest((request, response) => {
    //------------------------------------
    // 建立一個對話代理人
    //------------------------------------	
    const agent = new WebhookClient({ request, response });

    //------------------------------------
    // 處理使用者開始對話的意圖
    //------------------------------------		
    function welcome(agent) {
        agent.add('request.body:'+JSON.stringify(request.body));
        
        agent.add('傳入訊息:'+request.body.queryResult.queryText);
        agent.add('action:'+request.body.queryResult.action);
        agent.add('parameters:'+request.body.queryResult.parameters);
	//使用者資訊已不支援
        //agent.add('userId:'+request.body.originalDetectIntentRequest.payload.data.source.userId);
        //agent.add('timestamp:'+request.body.originalDetectIntentRequest.payload.data.timestamp);
        
        agent.add(new Card({
            title: '表頭',
            imageUrl: 'https://某張圖片的位址',
            text: '內文',
            buttonText: '按鈕上的文字',
            buttonUrl: 'https://assistant.google.com/'       
        }));
    }

    //------------------------------------
    // 處理機器人不懂使用者的意圖
    //------------------------------------		
    function fallback(agent) {
        agent.add('我不清楚你的意思');
        agent.add('可以再說一次嗎?');
    }
  
    //------------------------------------
    // 處理使用者提供姓名的意圖
    //------------------------------------	  
    function providing_name(agent) {
        agent.add('request.body:'+JSON.stringify(request.body));      
        agent.add('傳入訊息:'+request.body.queryResult.queryText);
        agent.add('action:'+request.body.queryResult.action);
	//使用者資訊已不支援
        //agent.add('userId:'+request.body.originalDetectIntentRequest.payload.data.source.userId);
        //agent.add('timestamp:'+request.body.originalDetectIntentRequest.payload.data.timestamp);  
        
        agent.add('參數:'+request.body.queryResult.parameters.name);
        agent.add('參數:'+request.body.queryResult.parameters.firstname);
        agent.add('參數:'+request.body.queryResult.parameters.lastname);
    }  

    //------------------------------------
    // 設定對話中各個意圖的函式對照
    //------------------------------------	
    let intentMap = new Map();
	
    intentMap.set('Default Welcome Intent', welcome);
    intentMap.set('Default Fallback Intent', fallback);
    intentMap.set('providing_name', providing_name);
 
    agent.handleRequest(intentMap);
    //------------------------------------	
});
```
