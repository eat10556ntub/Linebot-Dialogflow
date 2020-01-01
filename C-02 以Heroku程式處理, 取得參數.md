# C-02 以Heroku程式處理, 送出學號, 詢問姓名


```
|__ Intents
|      |__ Default Welcome Intent
.      |        .
.      |        |__ Fulfillment
.      |                |__ Enable webhook call for this intent (開啟)
.      .
.      .
.      |__ ask name
.               |__ Training Phrases
.               .       |__ 請問120001的名字 
.               .       |__ 問120001名字 
.               .       |__ 120001名字 
.               .           ------ PARAMETER NAME(stuno)  ENTITY(@sys.any)
.               .
.               |__ Action and Parameter
.               .       |__ REQUIRED(V)    PARAMETER NAME(stuno)    ENTITY(@sys.any)    VALUE($stuno) 
.               .
.               .
.               |__ Fulfillment
.                       |__ Enable webhook call for this intent (開啟)
.
.
|__ Entities
.      |__ name, 名字;姓名
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
       |
       |__ <utility>
               |__ asyncDB.js
               |__ student.js   
```

## 安裝外掛
```
npm install pg --save  
```

## (1) asyncDB.js

```javascript
'use strict';

//-----------------------
// 引用資料庫模組
//-----------------------
const {Client} = require('pg');

//-----------------------
// 自己的資料庫連結位址
//-----------------------
var pgConn = 'postgres://填入自己的資料';


//產生可同步執行query物件的函式
function query(sql, value=null) {
    return new Promise((resolve, reject) => {
        //設定資料庫連線物件
        var client = new Client({
            connectionString: pgConn,
            ssl: true
        })     

        //連結資料庫
        client.connect();

        //回覆查詢結果  
        client.query(sql, value, (err, results) => {                   
            if (err){
                reject(err);
            }else{
                resolve(results);
            }

            //關閉連線
            client.end();
        });
    });
}

//匯出
module.exports = query;
```



## (2) student.js

```javascript
'use strict';

//引用操作資料庫的物件
const query = require('./asyncDB');

//------------------------------------------
// 由學號查詢學生資料
//------------------------------------------
var getStu = async function(stuno){
    //存放結果
    let result;  

    //讀取資料庫
    await query('select * from student where stuno = $1', [stuno])
        .then((data) => {
            if(data.rows.length > 0){
                result = data.rows[0];  //學生資料(物件)
            }else{
                result = -1;  //找不到資料
            }    
        }, (error) => {
            result = -9;  //執行錯誤
        });

    //回傳執行結果
    return result;  
}
//------------------------------------------

//匯出
module.exports = {getStu};
```



## (3) index.js

```javascript
"use strict";

const express = require('express')
const { WebhookClient } = require('dialogflow-fulfillment')
const {Text, Card, Image, Suggestion, Payload} = require('dialogflow-fulfillment'); 
const app = express()

//增加引用函式
const student = require('./utility/student');

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
    // 處理詢問姓名
    //------------------------------------     
    function askName(){
        //取得傳入參數
        var stuno = req.body.queryResult.parameters.stuno;

        //呼叫API取得學生資料
        return student.getStu(stuno).then(data => {  
            if (data == -1){
                agent.add('找不到資料');
            }else if(data == -9){   
                agent.add('執行錯誤');  
            }else{
                agent.add('姓名:'+data.stuname);
                agent.add('性別:'+data.gender);
            }  
        }) 
    }    

    //------------------------------------
    // 設定對話中各個意圖的函式對照
    //------------------------------------
    let intentMap = new Map();
    intentMap.set('Default Welcome Intent', welcome);
    intentMap.set('ask name', askName);
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


## (4) package.json 

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
