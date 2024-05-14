# Reminder_bot
teratailを見てこられた方へ
```
/******/ (() => { // webpackBootstrap
/******/ 	"use strict";
/******/ 	var __webpack_modules__ = ({

/***/ "./src/line.ts":
/*!*********************!*\
  !*** ./src/line.ts ***!
  \*********************/
/***/ ((__unused_webpack_module, __webpack_exports__, __webpack_require__) => {

__webpack_require__.r(__webpack_exports__);
/* harmony export */ __webpack_require__.d(__webpack_exports__, {
/* harmony export */   sendPushMessage: () => (/* binding */ sendPushMessage),
/* harmony export */   sendReplyMessage: () => (/* binding */ sendReplyMessage)
/* harmony export */ });
const prop = PropertiesService.getScriptProperties().getProperties();
const CHANEL_ACCESS_TOKEN = prop.CHANEL_ACCESS_TOKEN;

/**
 * リプライメッセージを送信する
 * @param replyToken
 * @param messages
 */
const sendReplyMessage = (replyToken, messages) => {
  const ENDPOINT_URL = 'https://api.line.me/v2/bot/message/reply';
  const options = {
    headers: {
      'Content-Type': 'application/json; charset=UTF-8',
      Authorization: 'Bearer ' + CHANEL_ACCESS_TOKEN
    },
    method: 'post',
    payload: JSON.stringify({
      replyToken: replyToken,
      messages: messages
    })
  }; // @ts-ignore

  return UrlFetchApp.fetch(ENDPOINT_URL, options);
};
/**
 * プッシュメッセージを送信する
 * @param to
 * @param messages
 */

const sendPushMessage = (to, messages) => {
  const ENDPOINT_URL = 'https://api.line.me/v2/bot/message/push';
  const options = {
    headers: {
      'Content-Type': 'application/json; charset=UTF-8',
      Authorization: 'Bearer ' + CHANEL_ACCESS_TOKEN
    },
    method: 'post',
    payload: JSON.stringify({
      to: to,
      messages: messages
    })
  }; // @ts-ignore

  return UrlFetchApp.fetch(ENDPOINT_URL, options);
};

/***/ }),

/***/ "./src/main.ts":
/*!*********************!*\
  !*** ./src/main.ts ***!
  \*********************/
/***/ ((__unused_webpack_module, __webpack_exports__, __webpack_require__) => {

__webpack_require__.r(__webpack_exports__);
/* harmony export */ __webpack_require__.d(__webpack_exports__, {
/* harmony export */   doPost: () => (/* binding */ doPost),
/* harmony export */   main: () => (/* binding */ main),
/* harmony export */   remind: () => (/* binding */ remind)
/* harmony export */ });
/* harmony import */ var _spreadsheet__WEBPACK_IMPORTED_MODULE_0__ = __webpack_require__(/*! ./spreadsheet */ "./src/spreadsheet.ts");
/* harmony import */ var _line__WEBPACK_IMPORTED_MODULE_1__ = __webpack_require__(/*! ./line */ "./src/line.ts");


const main = () => {
  console.log('🐛 debug : テスト');
};
/**
 * WebhookからのPOSTリクエストを処理する
 * @param e
 */

const doPost = e => {
  const EVENTS = JSON.parse(e.postData.contents).events;

  for (const event of EVENTS) {
    execute(event);
  }
};
/**
 * イベントを処理する
 * @param event
 */

const execute = event => {
  const EVENT_TYPE = event.type;
  const REPLY_TOKEN = event.replyToken;
  const USER_ID = event.source.userId;

  if (EVENT_TYPE === 'message') {
    if (event.message.type === 'text') {
      const text = event.message.text; // 「登録」で始まるメッセージの場合、リマインドメッセージを登録する

      const matchResult = text.match(/^登録/);

      if (matchResult && matchResult.input === text) {
        add(text, REPLY_TOKEN, USER_ID);
      } else {
        sendError(REPLY_TOKEN);
      }
    }
  }
};
/**
 * リマインドメッセージをスプレッドシートに登録する
 */


const add = (text, replyToken, userId) => {
  // 登録 <日付(月/日)> <メッセージ>の形式であることを確認する
  const reg = /^登録 (\d{1,2}\/\d{1,2}) (.+)$/;
  const validate = reg.test(text);

  if (!validate) {
    sendError(replyToken);
    return;
  }

  const match = text.match(reg); // 日付を取得

  const dateStr = match?.[1] ?? '';
  const date = new Date(dateStr); // 有効な日付であることを確認する, 空文字もここで弾けるはず

  if (isNaN(date.getTime())) {
    sendError(replyToken);
    return;
  } // スプレッドシートを開く


  const activeSpreadsheet = SpreadsheetApp.getActiveSpreadsheet();
  const sheet = activeSpreadsheet.getSheetByName('シート1');

  if (!sheet) {
    throw new Error('sheet not found');
  } // 列のインデックスを取得


  const columnIndexMap = (0,_spreadsheet__WEBPACK_IMPORTED_MODULE_0__.getColumnIndexMap)(sheet); // 新しい行を作成して書き込む

  const newRow = Array.from({
    length: _spreadsheet__WEBPACK_IMPORTED_MODULE_0__.columnHeader.length
  }, () => '');
  newRow[columnIndexMap.date] = dateStr;
  newRow[columnIndexMap.message] = match?.[2] ?? '';
  newRow[columnIndexMap.user_id] = userId;
  sheet.appendRow(newRow); // 登録完了メッセージを送信する

  const messages = [{
    type: 'text',
    text: '登録しました'
  }];
  (0,_line__WEBPACK_IMPORTED_MODULE_1__.sendReplyMessage)(replyToken, messages);
};
/**
 * リマインドメッセージを送信する
 * @param replyToken
 */


const sendError = replyToken => {
  const messages = [{
    type: 'text',
    text: '登録 <日付(月/日)> <メッセージ>の形式で入力してください'
  }];
  (0,_line__WEBPACK_IMPORTED_MODULE_1__.sendReplyMessage)(replyToken, messages);
};
/**
 * リマインドメッセージを送信する
 */


const remind = () => {
  // スプレッドシートを開く
  const activeSpreadsheet = SpreadsheetApp.getActiveSpreadsheet();
  const sheet = activeSpreadsheet.getSheetByName('シート1');

  if (!sheet) {
    throw new Error('sheet not found');
  } // 列のインデックスを取得


  const columnIndexMap = (0,_spreadsheet__WEBPACK_IMPORTED_MODULE_0__.getColumnIndexMap)(sheet); // 今日の日付を取得

  const today = new Date();
  const todayMonth = today.getMonth() + 1;
  const todayDate = today.getDate(); // データを取得して、今日の日付のデータを抽出する

  const rows = sheet.getDataRange().getValues();
  // ユーザーごとにメッセージをまとめる
  const userMessagesMap = rows.reduce((acc, row) => {
    const rowDate = row[columnIndexMap.date];
    const rowDateObj = new Date(rowDate); // 今日の日付のデータの場合、メッセージを格納する

    if (rowDateObj.getMonth() + 1 === todayMonth && rowDateObj.getDate() === todayDate) {
      // 既に同じユーザーに対するメッセージの配列がある場合、メッセージを追加する
      if (acc[row[columnIndexMap.user_id]]) {
        acc[row[columnIndexMap.user_id]].push({
          type: 'text',
          text: row[columnIndexMap.message]
        });
      } else {
        // まだ同じユーザーに対するメッセージの配列がない場合、新しくメッセージの配列を作成する
        acc[row[columnIndexMap.user_id]] = [{
          type: 'text',
          text: row[columnIndexMap.message]
        }];
      }
    }

    return acc;
  }, {}); // ユーザーごとにメッセージを送信する

  for (const userId in userMessagesMap) {
    const messages = userMessagesMap[userId];
    (0,_line__WEBPACK_IMPORTED_MODULE_1__.sendPushMessage)(userId, messages);
  }
};

/***/ }),

/***/ "./src/spreadsheet.ts":
/*!****************************!*\
  !*** ./src/spreadsheet.ts ***!
  \****************************/
/***/ ((__unused_webpack_module, __webpack_exports__, __webpack_require__) => {

__webpack_require__.r(__webpack_exports__);
/* harmony export */ __webpack_require__.d(__webpack_exports__, {
/* harmony export */   columnHeader: () => (/* binding */ columnHeader),
/* harmony export */   getColumnIndexMap: () => (/* binding */ getColumnIndexMap)
/* harmony export */ });
// データが格納されているシートのヘッダー情報
const columnHeader = ['date', 'message', 'user_id'];

// 定義されているヘッダー情報と一致するか
const isColumnHeader = item => {
  return columnHeader.some(type => type === item);
};
/**
 * シートのヘッダーの列数情報を取得
 * NOTE: どのヘッダーが何列目にあるのかを取得する
 * @param sheet
 */


const getColumnIndexMap = sheet => {
  const headerValues = sheet.getRange(1, 1, 1, sheet.getLastColumn()).getValues()[0]; // 列数情報を作成

  return headerValues.reduce((acc, item, index) => {
    if (isColumnHeader(item)) {
      acc[item] = index;
    }

    return acc;
  }, {});
}; // シート1行分の情報

/***/ })

/******/ 	});
/************************************************************************/
/******/ 	// The module cache
/******/ 	var __webpack_module_cache__ = {};
/******/ 	
/******/ 	// The require function
/******/ 	function __webpack_require__(moduleId) {
/******/ 		// Check if module is in cache
/******/ 		var cachedModule = __webpack_module_cache__[moduleId];
/******/ 		if (cachedModule !== undefined) {
/******/ 			return cachedModule.exports;
/******/ 		}
/******/ 		// Create a new module (and put it into the cache)
/******/ 		var module = __webpack_module_cache__[moduleId] = {
/******/ 			// no module.id needed
/******/ 			// no module.loaded needed
/******/ 			exports: {}
/******/ 		};
/******/ 	
/******/ 		// Execute the module function
/******/ 		__webpack_modules__[moduleId](module, module.exports, __webpack_require__);
/******/ 	
/******/ 		// Return the exports of the module
/******/ 		return module.exports;
/******/ 	}
/******/ 	
/************************************************************************/
/******/ 	/* webpack/runtime/define property getters */
/******/ 	(() => {
/******/ 		// define getter functions for harmony exports
/******/ 		__webpack_require__.d = (exports, definition) => {
/******/ 			for(var key in definition) {
/******/ 				if(__webpack_require__.o(definition, key) && !__webpack_require__.o(exports, key)) {
/******/ 					Object.defineProperty(exports, key, { enumerable: true, get: definition[key] });
/******/ 				}
/******/ 			}
/******/ 		};
/******/ 	})();
/******/ 	
/******/ 	/* webpack/runtime/global */
/******/ 	(() => {
/******/ 		__webpack_require__.g = (function() {
/******/ 			if (typeof globalThis === 'object') return globalThis;
/******/ 			try {
/******/ 				return this || new Function('return this')();
/******/ 			} catch (e) {
/******/ 				if (typeof window === 'object') return window;
/******/ 			}
/******/ 		})();
/******/ 	})();
/******/ 	
/******/ 	/* webpack/runtime/hasOwnProperty shorthand */
/******/ 	(() => {
/******/ 		__webpack_require__.o = (obj, prop) => (Object.prototype.hasOwnProperty.call(obj, prop))
/******/ 	})();
/******/ 	
/******/ 	/* webpack/runtime/make namespace object */
/******/ 	(() => {
/******/ 		// define __esModule on exports
/******/ 		__webpack_require__.r = (exports) => {
/******/ 			if(typeof Symbol !== 'undefined' && Symbol.toStringTag) {
/******/ 				Object.defineProperty(exports, Symbol.toStringTag, { value: 'Module' });
/******/ 			}
/******/ 			Object.defineProperty(exports, '__esModule', { value: true });
/******/ 		};
/******/ 	})();
/******/ 	
/************************************************************************/
var __webpack_exports__ = {};
// This entry need to be wrapped in an IIFE because it need to be isolated against other modules in the chunk.
(() => {
/*!**********************!*\
  !*** ./src/index.ts ***!
  \**********************/
__webpack_require__.r(__webpack_exports__);
/* harmony import */ var _main__WEBPACK_IMPORTED_MODULE_0__ = __webpack_require__(/*! ./main */ "./src/main.ts");

__webpack_require__.g.main = _main__WEBPACK_IMPORTED_MODULE_0__.main;
__webpack_require__.g.doPost = _main__WEBPACK_IMPORTED_MODULE_0__.doPost;
__webpack_require__.g.remind = _main__WEBPACK_IMPORTED_MODULE_0__.remind;
})();

/******/ })()
;
```
## 手順書 (この方の記事を参考にしました。)
https://minakoph.notion.site/GAS-TS-LINE-Bot-BOT-AWARDS-2024-653b26d50a33430f9283a66839d27704
