Table Of Contents

[TOC]

## 0. 사전자료
[chatIF.md](https://github.com/com2best/dev-guide/blob/master/ChatIF.md)
[dialog.md](https://github.com/com2best/dev-guide/blob/master/dialog.md)
[READMEv1.md](https://github.com/com2best/dev-guide/blob/master/READMEv1.md)

참조 코드의 위치는 bot-web 0.9 기준

## 1. Dialog

### Diadalog = {
     on    ne       et       l 
        inents: []
        te              eence     et   
 task
    utut    
}
```


###  Input

Input은 아래와 같은 종류가 있다. 

* text: 키워드 매칭
* if: "변수명 === true" 같은 조건문과 true/false를 return하는 function도 될 수 있다. 
* intent: 인텐트
* entity: 엔터티
* types: 여러게의 type이 들어갈수 있다. 
  type에 매치가 되면 task 변수에 저장되어 action에서 사용하거나 출력의 치환에 사용할 수 있다. 
* function: true, false를 return 하는 함수가 올 수 있다. 
    
  tuto ft으로 값을 변환 한다. 

* 반복 ## 
* 치환 ++ 
* 예시: [+category+]\n##+index+. +name+ +openStatus+\n#0. 이전\n!. 처음(주문취소)
 
index는 1번부터 시작되는 번호로 자동으로 만든다. 
치환할 변수는 task, context.dialog, context.user 순으로 scope 별로 순차적으로 찾는다. 
치환은 tag를 {}로 변경, {변수명}. 단기적으로는 +변수명+와 {변수명} 모두 되도록.

현재는 processOuput function에 구현되어 있다. 

아래와 같은 형식을 표현해야 한다. 

* im
  aton Actions
Action의 종류

* call
* up: 기본적으로는 상위 Dialog인데, 현재는 이전 Dialog와 혼합해서 사용됨
* back: (
* repet
*eturCal: 다른 Dialog로 이동한후 다시 원래 Dialog로 복귀할 때 
* return: returnDialog로 호출되었을 때 호출한 Dialog로 복귀
* callChild: 하위에 있는 Dialog를 바로 검색
* returnCallChild: 하위 Dialog를 검색하여 returnCall
  {returnCallChild: '메뉴입력', options: {commonCallChild: 1, returnDialog: '주문주소확인'}}  }
    
Action에서는 다음과 같은 option을 사용할 수 있다. 

* output: 이동하면서 표시될 내용 전체를 바꾸는 경우. 형식 String
* prefix: 이동하면서 본래 출력될 내용의 위에 내용을 추가하는 경우. 형식 String
* postfix: 이동하면서 본래 출력될 내용의 아래에 내용을 추가하는 경우. 형식 String
* returnDialog: returnCall과 사용. return시 이동할 dialog를 변경할 경우. 형식 Dialog이름
* page: 이전 다음 페이지로 이동. 
  {repeat: 1, options: {page: "next"}}
  {repeat: 1, options: {page: "pre"}}
* commonCallChild: common Dialog에서 callChild할때 인것 같음
* top: 확인 중
  {call: '배달주문', options: {top: '배달주문'}}
 
### Types
#### Type 정의
아래와 같이 Type의 정의는 JSON으로 하고, typeCheck 함수를 통해 match 여부를 결정한다. 

```
var sampleType = {
	name: 'sampleType',
	typeCheck: function (text, type, dialog, context, callback) {
	  var matched = true; 매치여부
	  callback(text, dialog, matched);
	}
}
```

typeCheck 함수의 파라미터는 다음과 같다.

* text: 사용자에서 입력받은 내용이 넘어온다. NLP 처리가 된 문장이 넘어온다.
* type: type 개체 자체
* dialog: dialog개체로 match된 값은 task.param 저장하여 task 처리에 사용한다.
* context: context 개체
* callback: match처리 후 비동기 방식으로 callback을 호출한다. 


callback 함수 호출 시 파라미터는 다음과 같다.

* text: 입력 paramter를 다시 넘겨 준다. 
* dialog: 처리할 dialog
* matched: 매치 여부

type의 처리는 [dialog.js::executeType](https://github.com/com2best/bot-web/blob/0.9/modules/bot/action/common/dialog.js) function에 구현되어 있다. 

#### type option

* raw: 일반적으로 type 처리하기전에 자연어 처리 후 typeCheck함수를 실행하는데, 원문으로 typeCheck에 넘겨줌. 주소를 원문이 필요한 경우
* context: type이 매칭된 경우 user에 저장. 주소, 휴대폰번호 등. true, false 모두 가능
  {types: [{type : type.mobileType, context: false}]}
* init: 여러단계에 걸처서 Dialog호출이 이루어진경우 처음 진입시의 input을 사용함.
  name: '음식점메뉴단일',
  input: {types: [{type: orderTask.restaurantType, init: true}, orderTasks.menuType],
* save: 매치된 type을 task에만 저장하는 것이  아니라, context.dialog에도 자동 저장하여 사용한다. 
  여러 dialog에서 쓰려고 저장해 놓고 싶은데, 함수로 구현하지 않고 간단히 옵션으로 처리하기 위함
* failSave: save로 저장된 context.dialog의 값을 삭제하고 싶을 때 
  예시: { input: {types: [{type: orderTask.restaurantType, failSave: false}]}, output: {callChild: '음식점입력'}}
  context.dialog[type.name] = null;  //구현코드
  
#### common Type

* listType: 리스트 선택, 번호, 텍스트의 일부로도 선택가능 해야함.
  {types: [{name: 'orderRestaurant', listName: 'restaurant', typeCheck: 'listTypeCheck'}]}
  listName: array를 저장하고 있는 변수. 기본은 task.doc
  name: 선택된 값을 저장할 변수. 기본은 type name
* amount: 금액, 십백천만억원 으로 끝나는 말
* mobile: 휴대폰 번호
* phone: 전화번호
* date: 날짜
* time: 시간
* account: 계좌번호, 숫자와 -로 이루어진 문자열

현재는 type.js에 있음 
 
### DM 고려사항

* 이전으로 갔을 때: 처음 진입시 dialog에 저장한 데이터 그대로 있어야하고, 이전으로 갈때도 task를 실행해야함.
* concept: 동의어 사전 구현 필요.
* type matching시에 한번 matching된것은 재실행 하지 않는다.
* Quibble 구현: 명사, 동사, 문장 등으로 구분. 봇별/글로벌 모두 구현 [global-quibbles.js](https://github.com/com2best/bot-web/blob/0.9/custom_modules/global/global-quibbles.js)
* Global 기능: 공통대화, 퀴블, 동의어 등을 적용 가능하도록 custom_moduels/globals
 
### DM 기능 구현 소스

배달봇
[order/order.dialog.js](https://github.com/com2best/bot-web/blob/0.9/custom_modules/order/order.dialog.js)
[Concept](https://github.com/com2best/bot-web/blob/0.9/custom_modules/order/concepts.js): 동의어

CSDemo
[csdemo/csdemo.graph.js]() github에 없으나 로컬 소스에 있는 지 확인

B2B Demo
[Dialog Graph Demo](https://test.moneybrain.ai/developer/bots/graph-dialog/58bd136d0e8e202503d50587/58d1fb01bc12bd4d4e5360cb)

## 2. Question - Answer (QA)

### QA Options


### QA 고려사항

* DM-DL Hybrid: Intent 분석과 QA를 같은 구조로하고, 더 확률이 높은 것을 선택, 하위 구조에서는 우선권
* context 기능: Context 인식, Context 스위칭, 계층적 Context

### QA 구현 참고
B2B Demo
[실시간DL](https://test.moneybrain.ai/developer/user-bots/context-analytics): QA분석, Context분석


## 3. Task
--

ATHENA의 봇은 사용자의 질문에 대한 정해진 답변만을 출력하는 것에 머물지 않습니다. 사용자의 질문, 요청 등 입력에 대해 태스크(업무)를 수행하고, 이를 답변으로 출력하는 것을 목표로 합니다.  
이때 사용자의 요청에 대한 수행할 업무를 정의한 것을 태스크 라고 합니다. 태스크의 구현은 Javascript 언어(Node.js기반)로 구현하며, Task를 비롯한 자료들은 JSON 형태로 정의합니다.  
Task는 *.js파일에서 정의되며, 대화 그래프 편집창 왼쪽 파일트리 혹은 위쪽 탭에서 들어갈 수 있습니다.  

### 태스크 형식
Task의 기본적인 포맷은 다음과 같이 정의됩니다.  

```javascript
var bot = require(path.resolve('config/lib/bot')).getBot('defaultbot');

var defaultTask = {
    name: 'defaultTask',
    action: function(task, context, callback) {
        callback(task, context);
    }
};
bot.setTask("defaultTask", defaultTask);
```
### Action
Action 함수는 Task 내에서 실제 구현이 들어가 있는 함수로 다음과 같이 정의한다.  

```javascript
function actionName(task, context, callback) {	
	// 여기에 Task에 처리할 내용을 구현한다. 
	callback(task, context);			
}
```

Action 함수는 세가지 파라메터 변수를 받으며, 그 정의는 다음과 같습니다.  
첫번째, 파라미터 변수인 task는 action 함수에서 업무 처리에 필요한 정보를 JSON 형태로 받고, 처리한 결과를 담아서 return 하는 데 사용됩니다. 내부적으로는 action이 정의된 task와 같은 JSON 객체이다.  
두번째, 파라미터 변수인 context는 문맥, 상황을 위한 변수입니다. 스마트한 봇을 구현하기 위해 대화의 문맥, 사용자의 정보 들을 context에 담아서 JSON 형태로 전달합니다.  
세번째, 파라미터 변수인 callback은 결과를 return 하는 역할을 합니다. Action 함수는 비동기(Async) 방식으로 구현합니다. 그러므로 업무 처리가 끝나면 반드시 callback을 호출하여야 합니다.  
Action 함수를 세가지 파라미터 변수로 표준화 함으로써 어떠한 업무를 처리하는 Action 함수도 동일한 구조로 구현할 수 있게 합니다. 다양한 업무처리에 필요한 변수가 달라질 수도 있는 점은 task와 context 변수를 JSON 으로 정의해서 필요한 정보를 JSON에 담아서 보내는 방식으로 처리합니다.

#### Task Parameter
자세한 설명 전에 간단한 Action 함수를 예제로 정의해 봅니다. 아래 함수는 http 요청을 해서 웹에서 데이터를 가지고 옵니다. task는 node 기반 javascript로 작성할 수 있어 node library를 사용할 수 있다. 에러 예외 처리 등은 되어 있지 않는 예시용 함수입니다. 

```javascript
function httpAction(task, context, callback) {
	var request = require('request');
	request({url: task.url}, function (error, response, body) {
	  if (!error && response.statusCode == 200) {
		task.content = body;
	    callback(task, context);
	  }
	})
}
```
위에서 request 모듈은 node에서 많이 쓰이는 http request 라이브러리로 자세한 내용은 https://github.com/request/request 를 참고합니다. 

다음으로 Task 파라미터 변수를 설정하는 방법을 알아봅니다.  
Task를 정의할 때 추가하는 방법입니다. 미리 정의된 변수라고 생각할 수 있습니다.

```javascript
var googleTask = {
	url: 'https://www.google.com',
	action: httpAction,
}
```

위와 같이 정의한 googleTask를 실행하면, task.uri 에 있는 google 주소에 접속에 페이지의 HTML을 읽어서 task.content에 저장합니다.

```javascript
var naverTask = {
	url: 'http://www.naver.com',
	action: httpAction,
}
```
위와 같이 정의한 naverTask를 실행하면, task.uri 에 있는 naver 주소에 접속에 페이지의 HTML을 읽어서 task.content에 저장합니다.

#### Context Parameter

context는 해당 task에 제한되지 않은 여러정보들을 담고 있습니다. 이를 통해 현재 사용자의 입력에 해당되는 정보뿐만 아니라, 사용자 정보 등 다양한 상황에 따른 처리를 할 수 있습니다.  
현재 정의된 상황별 context는 다음과 같습니다.

* context.user: 사용자의 정보를 담고 있다. 
* context.bot: 현재 봇의 정보를 담고 있다. 
* context.botUser: 현재 봇을 사용하는 사용자의 정보를 담고 있다. 
* context.dialog: Dialog가 여러단계로 이루어지는 경우 대화의 문맥을 파악하기 위해 사용 한다. 
	
머니브레인 봇의 모든 데이터는 JSON을 기반으로 하므로, bot 개발시 필요한 경우 각 context 수준에 맞게 추가적인 key를 정의하여 사용할 수 있습니다. 

### Task 결과 후처리
Action 함수에서 처리한 결과를 xpaht 와 regexp로 간편하게 추출하여 사용할 수 있습니다. 또는, 결과값을 http 등으로 받는 html 데이터를 xpath 를 사용하여 저장할 수 있습니다.  
아래의 예시와 같이 하면 html 값에서 <title></title> 사이의 값을 xapth로 읽어서 task.title 에 저장합니다. _text: 'body' 로설정한 것은 task.body 에 있는 값을 사용해서 xpath 검색을 한다는 의미로 http 요청으로 받은 데이터가 task.body 에 저장되어 있기 때문입니다.  

```javascript
var xpathTask = {
	url: 'https://www.google.co.kr/search?q=moneybrain.ai',
	action: httpAction,
	xpath: {
	    _text: 'body',
		title: '//title/text()'
	}
}
```
아래와 같이 하면 html 값에서 검색리스트를 task.doc 에 array 형태로 저장합니다. doc 하부에 _repeat에 있는 xpath로 목록을 가져와서 list와 link 값으로 array를 구성합니다. doc 과 다른 이름으로 추가 xpath를 지정하여 여러게의 list 을 담을 수 있습니다.  

```javascript
var xpathListTask = {
	url: 'https://www.google.co.kr/search?q=moneybrain.ai',
	action: httpAction,
	xpath: {
	    _text: 'body'
		title: '//title/text()',
        doc: {
	      _repeat: '//div[@class="srg"]/div[@class="g"]',
          list: '//a/text()',
          link: '//a/@href',
        }			
	}
}
```
정규식(Regular Expression)을 이용하여 결과값을 task 에 저장할 수 있습니다. 아래 예시에서는 정규식을 사용하여 html에서 title 값을 task.title에 저장합니다.  

```javascript
var xpathTask = {
	url: 'https://www.google.com',
	action: httpAction,
	regexp: {
	    _text: 'body',
		title: /<title>(.*)<\/title>/
	}
}
```
아래와 같이 하면 검색 리스트 텍스트에서 regexp로 match하여 task.doc을 array 형태로 저장합니다. _repeat에 있는 정규식 g flag 로 검색하여 여러개의 매치를 찾고, ( ) 를 사용하여 변수에 저장합니다. link와 list의 숫자는 매치 저장에서 저장되는 순서이며 1번부터 시작합니다.  

```javascript
var xpathTask = {
	url: 'https://www.google.com',
	action: httpAction,
	regexp: {
	    _text: 'body',
		title: /<title>(.*)<\/title>/
		doc: {
			_repeat: /<h class="r"><a.*href="(.*)".*>(.*)<\/a></h>/g
			link: 1,
			list: 2
		}
	}
}
```
### PreCallback,  PostCallback

아래와 같이 task에 preCallback, postCallback 함수를 구현하여 action 함수를 호출하기 전, 후에 처리할 작업을 추가할 수 있습니다.  

```javascript
var sampleTask =
{
  name: 'sample',
  preCallback: function(task, context, callback) {
      // 여기서 Action 함수를 호출하기 전에 수행할 일을 구현
	  callback(task, context);
  },
  action: sampleAction,
  postCallback: function(task, context, callback) {
      // 여기서 Action 함수를 호출한 후에 수행할 일을 구현
	  callback(task, context);
  }
};
```

### Action Parameter Definition

Action 함수에서 사용할 Entity 정보를 paramDefs로 정의할 수 있습니다. Action 함수의 Entity 정보를 정의하는 이유는 Dialog를 통해서 Task 를 사용할 때 어떠한 정보를 넘겨주어야 하는지 파악할 수 있기 때문입니다. 필수적인 정보가 입력된 내용에 없을 경우는 봇이 자동으로 사용자에게 질문하여 정보를 얻을 수 있습니다.  

```javascript
var sampleTask =
{
  name: 'sample',
  paramDefs: [
    {name: '휴대폰', type: mobileType, require: false, dialog: '휴대폰Dialog'},
  ],
  action: sampleAction,
};
```

paramDef 정의에는 다음의 속성들이 사용가능합니다.  

* name: 정의할 엔터티 이름으로 task JSON 개체에 같은 이름으로 저장된다. 
* require: boolean 형으로 필수 Entity 여부. 필수 Entity가 task에 없는 경우 봇이 사용자에게 요청하는 질문을 하여 입력을 받는다. 
* question: String 형으로 봇이 사용자에게 Entity를 물어볼 경우 질문 내용
* dialog: 봇이 사용자에게 Entity를 물어볼 경우 단순한 질문이 아니라, 몇단계의 dialog 가 있을 수 있는데 이를 정의한다. String 값이면 같은 이름의 dialog를 찾고, dialog 개체를 바로 참조할 수도 있다. 

### Task 구조화

Task를 한가지 이상 조합하여 상위 Task를 만들 수 있습니다.  

여러개의 Task를 연결하여 실행할 수 있습니다. 아래의 예시에서는 sequenceTask는 sampleTask1을 실행하고, sampleTask2를 실행합니##Dilaog 개발 가이드


###1. Quick Start

###2. Dialog

####2.1. Dialog 기본

기본 질문 답변 구조이다. "<" 뒤에 사용자가 입력할 문장을 적고, ">"뒤에 봇이 답변할 문장을 적는다. 입력에는 키워드만 넣고, 동사는 기본형으로 적어야 한다. 

	< 찾다
	> 텍스트 내용을 출력합니다.
 
아래와 같이 한줄로 정의할 수도 있다. 마찬가지고 "<" 뒤에서 ">"까지는 입력이고, ">" 이후는 답변이다. 
 
	<찾다> 텍스트내용을 출력합니다. 
	
Concept을 정의하여, 여러가지 단어를 동의어로 인식하여 매칭할 수 있다. 아래 예시에서는  "응" "좋아" 등 여러가지 입력에 대해 동일하게 인식한다.

	concept: 네 [응 좋아 yes ok] 	
	<~네> 동의하고 진행하겠습니다. 

####2.2. Dialog 입력 인식

Dialog에 정의된 입력을 사용자의 질문에 매치하는 것은 기본적으로 자연어 처리에 기반하여 이루어 진다. 아래에서 매치되는 예시를 보여준다. 

	< 계좌 이체 하다
	
* (매칭) 계좌 이체 해줘.  	      // 사용자가 입력한 "하다" 활용형은 기본형 동사로 매치됨
* (매칭) 계좌 이체좀 해줘.  	  // "이체좀"에서 주요단어는 분리되어 "이체"로 매치됨
* (매칭) 계좌 이체를 해주세요 // "이체를"에서 조사는 분리되어 매치됨 
* (매칭) 해줄래~ 계좌 이체     // 현재는 순서와 상관 없이 인식한다.   	

Regular Expression을 입력의 매치에 사용할 수 있다. 일반적인 정규식을 쓰는 방식과 동일하게 사용이 가능하다.

	</정규식/g> RegExp 매치 되면 내용 출력합니다.

if 조건문을 사용하여 특정조건에서 매치되도록 할 수 있다. 

	< {if: "context.bot.botName == 'sample'"}
	> 조건이 true이면 내용을 출력합니다

조건은 함수로 정의하여 true, false 반환하여 매치할 수 있다. 아래 예시 함수처럼 비동기 (async) 방식이며 callback에 true 또는 false로 호출하는 것에 따라 매치 결과가 나뉜다. inRaw는 자연어 처리전 문장, inNLP는 자연어 처리 후 문장이다.

	< {if: function(inRaw, inNLP, dialog, context, callback) { if(inRaw == '함수')callback(true);else callback(false); }}
	> 조건이 true이면 내용을 출력합니다


미리정의된 type을 인식할 수 있다. 아래 내용은 휴대폰번호가 입력 되었을 경우에 반응한다. 아래와 같이 mobile Type을 인식하면 인식된 mobile 값은 mobile 변수에 저장된다. 출력시에 +변수명+ 을 사용하면 저장된 변수를 사용하여 출력할 수 있다.

	< {types: [type.mobileType]}
	> 휴대폰 +mobile+ 를 출력합니다.

이러한 조건은 여러가지를 조합하여 모두 매칭되어야 출력되도록 할 수 있다. 아래 예시에서는 사용자가 "정규식 010-1111-2222"와 같이 regexp와 mobile이 모두 매치되어야 출력된다.

	< {regexp: /정규식/g, type: [type.mobileType], if: function(inRaw, inNLP, dialog, context, callback) {callback(true);}}
	> 모두 매치 되면 내용을 출력합니다.

####2.3. 다중 Dialog
입력 부분을 여러게 정의하면,  여러가지 입력 형태 중에서 하나가 매치되면 출력된다. 이러한 다중 입력을 활용하여 여러가지 형태의 사용자 입력을 반영할 수 있다.

	< 여러
	< /여러/g
	> 여러개 중에 하나 일치 되면 표시합니다.

출력 부분을 여러게 정의하면, 그 중에 하나가 랜덤하게 출력된다. 이 구현은 사용자가 물어볼 때 마다 다르게 답변할 때 사용한다.

	< 랜덤
	> 랜덤 첫번재 경우입니다.
	> 랜덤 두번째 경우입니다.

####2.4. 하위 Dialog

아래와 Dialog의 단계를 두어, 상위 단계의 입력을 받은 후에 해당 상황에서만 매치하도록 정의할 수 있다. 이를 통해 단계별 시나리오를 구성하거나, 메뉴 방식의 시나리오를 정의할 수 있다.

	<하위 질문> 1.메뉴1\n2.메뉴2\n3.메뉴3
	    <1> 메뉴1 내용입니다.
	    <2> 메뉴2 내용입니다.
	    <3> 메뉴3 내용입니다.

여러 단계를 가지는 대화도 구성가능하다.

	<상위 질문> 1.메뉴1\n2.메뉴2,
	    <1> 메뉴1 내용입니다.\n9.이전,
	        <9> {up: 1 }
	    <2> 메뉴2 내용입니다.\n9.이전,
	        <9> {up: 1}

####2.2. Dialog 호출

다른 Dialog를 호출하여 재사용할 수 있다. 호출 할때는 Dialog 이름으로 호출하므로 미리 Dialog 이름을 정의해야 하며, 정의는 Dialog명 뒤에 ":"를 붙여서 정의한다.

	호출1:
	< 호출대기
	> 첫번째 다이얼로그가 호출 되었을 때 출력됩니다.

아래 처럼 "call"을 이용하여 "호출1" Dialog를 호출하면, 위의 "호출1"에 정의된 출력이 표시된다.

	< 호출 하다 1
	> {call: '호출1'}

아래와 같이 입력부를 false로 하면, 사용자 입력에는 매치되지 않는다. Dialog 이름으로 다른 Dialog이 호출에만 사용된다.

	호출2:
	< false
	> 두번째 다이얼로그가 호출되었을 때 출력됩니다.

	< 호출 하다 2
	> {call: '호출2'}

 위의 다른 함수의 호출에는 option을 사용할 수 있다. 이러한 옵션은 아래 나열되는 up, repeat 등 다른 dialog의 호출에 모두 사용된다 .

	<옵션 1> {call: '호출1', options: {output: '표시되는 내용 교체'}}
	<옵션 2> {call: '호출1', options: {prefix: '앞에\n'}}
	<옵션 3> {call: '호출1', options: {postfix: '\n뒤에'}}

* ouput option: 호출은 하되 출력은 정의한 내용으로 바꾸어 출력한다.
* prefix option: 호출하면서 표시되는 출력 앞에 붙여서 출력한다.
* postfix option: 호출하면서 표시되는 출력 뒤에 붙여서 출력한다.

이전 단계로 이동할 때는 "up"을 이용한다. 아래 예시에서 9를 입력하면 상위로 이동한다.

	<상위 질문> 1.메뉴1\n2.메뉴2,
	    <1> 메뉴1 내용입니다.\n9.이전,
	        <9> {up: 1 }

 repeat를 사용하면 다시 이전 질문을 반복할 수 있다. 의도한 답변이 없을 때 사용할 수 있다.

	<하위 질문> 1.메뉴1\n2.메뉴2
	    <1> 메뉴1 안내입니다.
	    <2> 메뉴2 안내입니다.
	    <8> {repeat: 1}

####2.4. 공통 Dialog

사용자가 처음 봇에 접속 했을 때 표시할 내용은 "시작" 이라는 Dialog로 정의한다. 아래와 같이 하면 봇에 처음 접속시에 표시되고, 사용자가 "시작"이라고 입력하여도 매치된다.

	시작:
	<시작> 초기화 메세지 입니다

사용자입력 중에 매치되는 것이 없을 때 표시되는 것으로 아래와 같이 dialog 문서의 마지막에 정의한다. "<>"으로 입력부가 되어 있는 것은, 모든 입력에 매치된다는 것과 같으므로, 위에서 매치되는 문장이 없으면 마지막에 항상 매치되어 표시가 된다. 

	<> 매치되는 문장이 없습니다.                                     

하위 Dialog의 어느 단계에서나 매치될 수 있는 공통 Dialog 는 아래와 같이 입력부 앞에 c를 붙여서 정의한다. 대표적으로 이전으로 가기나, 시작으로 가기는 모든 단계의 dialog에서 계속 활용 되므로 공통 Dialog로 정의해 두고, 하위 Dialog 단계별로 반복적으로 정의하지 않아도 된다. 
    
	c<9> {up: 1}
	c<0> {call: 시작}


Dialog 의 기본 구조는 다음과 같다.  

	{
		input: '주문해줘',
		task : {
			name: 'order',
			action: OrderTask,
		}
		output: '주문이 완료 되었습니다.' 
	}
	
###3. Task

머니브레인의 봇은 사용자의 질문에 대한 정해진 답변만을 출력하는 것에 머물지 않는다. 사용자의 질문, 요청 등 입력에 대해 Task(업무)를 수행하고, 이를 답변으로 출력하는 것을 목표로 한다. 

이때 사용자의 요청에 대한 수행할 업무를 정의한 것을 Task 라고 한다. Task의 구현은 Javascript 언어로 구현하며, Task를 비롯한 자료들은 JSON 형태로 정의한다.  

####3.1. Task 정의

sample.dlg에서 아래와 같이 정의 한다.  

	< Task 실행
	sampleTask
	> Task 처리 결과: +result+

sample.js 에 아래와 같이 정의한다. 
```
var sampleTask =
{
  action: function (task, context, callback) {
    task.result = 'hello world';
    callback(task, context);
  }
};
```
위와 같이 하면, 아래와 같이 입력 출력이 이루어 진다.

	입력> Task 실행해줘.
	Task> helloworldTask의 action 함수를 실행하여 result 값에 "hello world"  저장.
	출력> Task 처리 결과:  hello world.

####3.2. Action 함수 정의

Action 함수는 Task 내에서 실제 구현이 들어가 있는 함수로 다음과 같이 정의한다. 

```javascript	
function actionName(task, context, callback) {	
	// 여기에 Task에 처리할 내용을 구현한다. 
	callback(task, context);			
}
```

Action 함수는 세가지 파라메터 변수를 받으며, 그 정의는 다음과 같다.

첫번째, 파라미터 변수인 task는 action 함수에서 업무 처리에 필요한 정보를 JSON 형태로 받고, 처리한 결과를 담아서 return 하는 데 사용된다. 내부적으로는 action이 정의된 task와 같은 JSON 객체이다.

두번째, 파라미터 변수인 context는 문맥, 상황을 위한 변수이다. 스마트한 봇을 구현하기 위해 대화의 문맥, 사용자의 정보 들을 context에 담아서 JSON 형태로 전달한다.   

세번째, 파라미터 변수인 callback은 결과를 return 하는 역활을 한다. Action 함수는 비동기(Async) 방식으로 구현한다. 그러므로 업무 처리가 끝나면 반드시 callback을 호출하여야 한다. 

Action 함수를 세가지 파라미터 변수로 표준화 함으로써 어떠한 업무를 처리하는 Action 함수도 동일한 구조로 구현할 수 있게 한다. 다양한 업무처리에 필요한 변수가 달라질 수도 있는 점은 task와 context 변수를 JSON 으로 정의해서 필요한 정보를 JSON에 담아서 보내는 방식으로 처리한다.



####3.3. Task 파라미터 변수

자세한 설명 전에 간단한 Action 함수를 예제로 정의해 본다. 아래 함수는 http 요청을 해서 웹에서 데이터를 가지고 온다. task는 node 기반 javascript로 작성할 수 있어 node library를 사용할 수 있다. 에러 예외 처리 등은 되어 있지 않는 예시용 함수 이다. 

```javascript
function httpAction(task, context, callback) {
	var request = require('request');
	request({url: task.url}, function (error, response, body) {
	  if (!error && response.statusCode == 200) {
		task.content = body;
	    callback(task, context);
	  }
	})
}
```
위에서 request 모듈은 node에서 많이 쓰이는 http request 라이브러리로 자세한 내용은 https://github.com/request/request 를 참고한다. 

다음으로 Task 파라미터 변수를 설정하는 방법을 알아본다.

첫째, Task 정의할 때 추가하는 방법이다. 미리 정의된 변수라고 생각할 수 있다.

	var googleTask = {
		url: 'https://www.google.com',
		action: httpAction,
	}


위와 같이 정의한 googleTask를 실행하면, task.uri 에 있는 google 주소에 접속에 페이지의 HTML을 읽어서 task.content 에 저장한다.

	var naverTask = {
		url: 'http://www.naver.com',
		action: httpAction,
	}

위와 같이 정의한 naverTask를 실행하면, task.uri 에 있는 naver 주소에 접속에 페이지의 HTML을 읽어서 task.content 에 저장한다.

둘째, 사용자의 입력을 받아 task 파라메터 변수로 전달하는 방법이다.

정규식을 사용하면 사용자가 입력한 내용 중에 일부를 변수로 사용할 수 있다. 

	< /구글 (.*) 찾다/
	googleTask
	> google 에서 +1+을 검색하였습니다. 

위 구현에서 정규식으로 저장한 1번째 내용이 task.1 에 저장된다.

사용자가 입력한 값을 받아서 Google에서 검색하는 것을 Task로 구현해 보기 위해 googleTask와 httpAction을 아래와 같이 수정한다.

```javascript
var googleTask = {
	url: 'https://www.google.co.kr/search?q=',
	action: googleAction,
}

function googleAction(task, context, callback) {
	var request = require('request');
	
	task.url = task.url + task.1;
	
	request({uri: task.url}, function (error, response, body) {
	  if (!error && response.statusCode == 200) {
		task.content = body;
	    callback(task, context);
	  }
	})
}
```

구글에서 검색하기 위해 url에 /search?q= 를 추가하였다. q= 뒤에 검색어를 넣어서 요청하면 된다. 그리고 googleAction 에서는 task.query 의 값을 q= 뒤에 합칠 수 있도록 구현을 추가 하였다. 

Action 함수에서 task.1 값을 검색 쿼리로 입력하여 google 검색을 요청한다. 

Type  을 사용하여 매치된 값은 type 이름으로 task에 저장되고 Action함수에서 사용할 수 있다.  예를 들어 아래와 같이 mobileType 에 매치되면 task.mobile 에 휴대폰 번호 형식으로 입력한 값이 저장된다.

	< {types: [type: mobileType]}
	> 휴대폰 번호는 +mobile+ 입니다. 


####3.4. Context 파라미터 변수

context	는 해당 task에 제한되지 않은 여러정보들을 담고 있다. 이를 통해 현재 사용자의 입력에 해당 되는 정보 뿐만아니라, 사용자 정보 등 다양한 상황에 따른 처리를 할 수 있다. 

현재 정의된 상황별 context는 다음과 같다.

* context.global: 
* context.user: 사용자의 정보를 담고 있다. 
* context.bot: 현재 봇의 정보를 담고 있다. 
* context.botUser: 현재 봇을 사용하는 사용자의 정보를 담고 있다. 
* context.dialog: Dialog가 여러단계로 이루어지는 경우 대화의 문맥을 파악하기 위해 사용 한다. 
	
머니브레인 봇의 모든 데이터는 JSON을 기반으로 하므로, bot 개발시 필요한 경우 각 context 수준에 맞게 추가적인 key를 정의하여 사용할 수 있다. 


####3.6. Task와 출력

task에 있는 값은 출력에 변수로서 값을 표시할 수 있다. +변수명+ 를 통해서 값을 표시할 수 있다. 

아래와 같이 task.title 에 값이 설정되어 있고, 출력부 +title+ 이 있으면 +title+부분이 task.title로 변경되어 표시 된다. 

	task.title = '머니브레인';
	
	> +title+ 표시합니다. 
	
	출력> 머니브레인 표시합니다. 

task 는 JSON 형태로 되어 있으므로, 하위 개체에 있는 것도 JSON 개체를 참조하는 방식으로 사용할 수 있다. 

	task.item = {};
	task.item.name = '홍길동';

	> +item.name+ 표시합니다. 
	
	출력> 홍길동 표시합니다. 


task에 array값로 정의하고 리스트로 표시할 수 있다.  반복표시할 내용은 #을 사용하여 # task변수값 # 반복표시할내용 # 형식으로 사용할 수 있다. 아래 반복 표시할 변수이름이 doc이면 생략하여 ## 반복표시할 내용 # 와 같이 사용할 수 있다. 

	task.doc = [
		{company: '머니브레인', name: '홍길동'},
		{company: '구글', name: 'Sundar'}
	]

	> #doc#+index+.+company+ +name+\n# 감사합니다. 
	
	출력> 1.머니브레인 홍길동 
	     2.구글 Sundar
         감사합니다.
         
####3.5. Task 결과 후처리

Action 함수에서 처리한 결과를 xpaht 와 reg exp로 간편하게 추출하여 사용할 수 있다. 

결과값을 http 등으로 받는 html 데이터를 xpath 를 사용하여 저장할 수 있다. 

아래의 예시와 같이 하면 html 값에서 <title></title> 사이의 값을 xapth로 읽어서 task.title 에 저장한다. _text: 'body' 로설정한 것은 task.body 에 있는 값을 사용해서 xpath 검색을 한다는 의미로 http 요청으로 받은 데이터가 task.body 에 저장되어 있기 때문이다. 

	var xpathTask = {
		url: 'https://www.google.co.kr/search?q=moneybrain.ai',
		action: httpAction,
		xpath: {
		    _text: 'body',
			title: '//title/text()'
		}
	}

아래와 같이 하면 html 값에서 검색리스트를 task.doc 에 array 형태로 저장한다.  doc 하부에 _repeat에 있는 xpath로 목록을 가져와서 list와 link 값으로 array를 구성한다. doc 과 다른 이름으로 추가 xpath를 지정하여 여러게의 list 을 담을 수 있다. 

	var xpathListTask = {
		url: 'https://www.google.co.kr/search?q=moneybrain.ai',
		action: httpAction,
		xpath: {
		    _text: 'body'
			title: '//title/text()',
	        doc: {
		      _repeat: '//div[@class="srg"]/div[@class="g"]',
	          list: '//a/text()',
	          link: '//a/@href',
	        }			
		}
	}


정규식(Regular Expression)을 이용하여 결과값을 task 에 저장할 수 있다. 아래 예시에서는 정규식을 사용하여 html에서 title 값을 task.title 에 저장한다. 

	var xpathTask = {
		url: 'https://www.google.com',
		action: httpAction,
		regexp: {
		    _text: 'body',
			title: /<title>(.*)<\/title>/
		}
	}


아래와 같이 하면 검색 리스트 텍스트에서 reg exp로 match하여 task.doc을 array 형태로 저장한다. _repeat에 있는 정규식 g flag 로 검색하여 여러개의 매치를 찾고, ( ) 를 사용하여 변수에 저장한다. link 와 list의 숫자는 매치 저장에서 저장되는 순서이며 1번부터 시작한다. 

	var xpathTask = {
		url: 'https://www.google.com',
		action: httpAction,
		regexp: {
		    _text: 'body',
			title: /<title>(.*)<\/title>/
			doc: {
				_repeat: /<h class="r"><a.*href="(.*)".*>(.*)<\/a></h>/g
				link: 1,
				list: 2
			}
		}
	}

####3.5. PreCallback,  PostCallback

아래와 같이 task에 preCallback, postCallback 함수를 구현하여 action 함수를 호출하기 전, 후에 처리할 작업을 추가할 수 있다.

```
var sampleTask =
{
  name: 'sample',
  preCallback: function(task, context, callback) {
      // 여기서 Action 함수를 호출하기 전에 수행할 일을 구현
	  callback(task, context);
  },
  action: sampleAction,
  postCallback: function(task, context, callback) {
      // 여기서 Action 함수를 호출한 후에 수행할 일을 구현
	  callback(task, context);
  }
};
```

####3.4. Action Parameter Definition

Action 함수에서 사용할 parameter 정보를 paramDefs로 정의할 수 있다. Action 함수의 parameter 정보를 정의하는 이유는 Dialog를 통해서 Task 를 사용할 때 어떠한 정보를 넘겨주어야 하는지 파악할 수 있기 때문이다. 필수적인 정보가 입력된 내용에 없을 경우는 봇이 자동으로 사용자에게 질문하여 정보를 얻을 수 있다. 

```javascript
var sampleTask =
{
  name: 'sample',
  paramDefs: [
    {name: '휴대폰', type: mobileType, require: false, dialog: '휴대폰Dialog'},
  ],
  action: sampleAction,
};
```

paramDef 정의에는 다음의 속성들이 사용가능하다

* name: 정의할 파라미터 이름으로 task JSON 개체에 같은 이름으로 저장된다. 
* require: boolean 형으로 필수 파라미터 여부. 필수 파라미터가 task에 없는 경우 봇이 사용자에게 요청하는 질문을 하여 입력을 받는다. 
* question: String 형으로 봇이 사용자에게 파라미터를 물어볼 경우 질문 내용
* dialog: 봇이 사용자에게 파라미터를 물어볼 경우 단순한 질문이 아니라, 몇단계의 dialog 가 있을 수 있는데 이를 정의한다. String 값이면 같은 이름의 dialog를 찾고, dialog 개체를 바로 참조할 수도 있다. 

####3.4. Task 구조화

Task를 한가지 이상 조합하여 상위 Task를 만들 수 있다. 

여러개의 Task를 연결하여 실행할 수 있다. 아래의 예시에서는 sequenceTask는 sampleTask1을 실행하고, sampleTask2를 실행한다. 

```javascript
var sampleTask1 =
{
  name: 'sample1',
  action: function (task, context, callback) {
    task.result = 'sample1';
    callback(task, context);
  }
};

var sampleTask2 =
{
  name: 'sample2',
  action: function (task, context, callback) {
    task.result = 'sample2';
    callback(task, context);
  }
};

var sequenceTask = {
	action: 'sequence'
	tasks: [
		sampleTask1,
		sampleTask2,
	]
}
```

while이나 for 문처럼 특정조건이 true일때 까지 반복해서 Task을 수행하게 할 수 있습니다. 조건이 true인지를 체크하는 방법은 수식이나 boolean을 return 하는 함수로 할 수 있습니다. 아래 예에서는 10번까지 Task를 반복 수행합니다. 한다. 

```
var sequenceTask = {
	action: 'while'
	whileIf: function(task, context) {
		if(!task.count) task.count = 0;
		else task.count++;
		
	    return task.count <= 10;
    }, 
	actions: [
		sampleTask1,
		sampleTask2
	]
}
```

if 문처럼 특정 조건일때만 Task를 실행하게 할 수 있습니다. 조건이 true인지를 판단하는 것은 조건식이나 function 으로 정의가 가능합니하다. 아래 예시에서는 sequence로 여러 task를 실행할 때 각 task if 조건이 맞는 경우에만 실행됩니다. 된다. 

```
var sequenceTask = {
	action: 'sequence'
	tasks: [
		{
		  name: 'sample1',
		  if: function(task, context) {
		    return task.mobile != undefined;
		  },
		  action: function (task, context, callback) {
		    task.result = 'sample1';
		    callback(task, context);
		  }
		},
		{
		  name: 'sample2',
		  if: 'context.user.mobile == undefined',
		  action: function (task, context, callback) {
		    task.result = 'sample2';
		    callback(task, context);
		  }
		}
	]
}
```

이러한 task 구조 상에서 다른 상하 task를 참조할 수 있습니다.  

* task.topTask:  최상위 Task
* task.parentTask: 상위 Task
* task.preTask: 이전 Task

### Task 간 데이터 전달

Task의 연결관계, 상하구조에서 데이터를 전달하기 위해서 기본적으로 preCallback, postCallback을 사용할 수 있습니다.  

아래 예시에서는 preCallback을 통해 sequence 등의 이전 task에서 query 값을 가져와서 task.param으로 사용합니다. postCallback을 통해서는 task에서 처리한 result.query 값을 topTask에 저장해 놓습니다. 

```javascript
var sampleTask =
{
  name: 'sample',
  preCallback: function(task, context, callback) {
	  task.param.query = task.preTask.result.query;
	  callback(task, context);
  },
  action: sampleAction,
  postCallback: function(task, context, callback) {
	  task.topTask.param.query = task.result.query;
	  callback(task, context);
  }
};
```
아래와 같이 task 속성을 통해 데이터를 전달할 수도 있습니다.  

```javscript
var sampleTask = {
  name: 'sample',
  data: {
	set: 'replace',
	context: {
	  user: ['result.doc']
	}, 
	task: {
	  topTask: ['result.title'], 
	  parent: ['param.title']
	}
  },
  action: sampleAction
};

var sampleTask = {
  name: 'sample',
  data: {
	get: 'merge',		
    context: ['bot.botName', 'user.mobile'], 
    task: ['topTask.name', parent.result.doc']
  },
  action: sampleAction
};
```

data 속성 안내는 두가지 key 값으로 구분을 합니다.   

* get: context 나 다른 task에서 값을 가져와 현재 task에 저장한다. 
* set: 현재 task의 결과를 context 나 다른 task에 저장한다. 

데이터를 전달하는데 다음의 옵션들이 있습니다.  

* replace: 기본 옵션으로 같은 key의 데이터가 있으면 대체한다.
* merge: 같은 key의 기존 데이터가 있으면 다른 부분만 새로 겹쳐서 합친다.
* concat: 데이터가 array인 경우 두 array를 합쳐서 저장한다. array concat 함수와 동일

### Template Task 

Template task를 사용해서 기존에 있던 task를 재사용하여 확장할 수 있습니다. task의 template 항목에 상속받을 template task 참조를 넣습니다. 참조의 방법은 일반 task참조와 동일합니다.

아래의 예시에서는 googleTask의 url은 그대로 사용하고, moneybrainTask에서 task.param.q 만 변경하여 요청합니다.

```javascript
var moneybrainTask = {
  name: 'moneybrainTask',
  template: googleTask,
  param: {
	  q: 'moneybrain.ai'
  }
};

var googleTask = {
	url: 'https://www.google.com',
	param: {
		q: ''
	},
	action: httpAction,
}
```
### common Task
* mongo
    * findById
    * findOne
    * find
    * findByIndex
    * selectByIndex
* http
    * json
    * xpathRepeat
    * xpath
    * xpathByIndex
    * selectByIndex
    * plaintext

이 부분은 아래 위치에 정의되어 있다. 
    bot-web/custom-modules/moneybot/template.js 

    var templates = {
      "movies": {
        "module": "http",
        "action": "xpathRepeat",
        "url": "http://movie.naver.com",
        "path": "/movie/running/current.nhn?view=list&tab=normal&order=reserve",
        "save": "true",
        "xpath": {
          "repeat": "//ul[@class='lst_detail_t1']/li",
          "limit": "5",
          "doc": {
            "title": "//dt[@class='tit']/a/text()",
            "path": "//dt[@class='tit']/a/@href",
            "director": "//dl[@class='info_txt1']/dd[2]//a/text()",
            "actors": "//dl[@class='info_txt1']/dd[3]//a/text()"
          }
        }
      },


### Task 고려사항

* Task UI: 위 간단한 task의 입력항목을 ui 로 입력하여 코딩 없이 간단한 업무 개발 가능
* Task Server: DM과 독립적으로 실행하여 Local에서 개발 디버깅 가능하게
* Task Module: 제공되는 module에는 여러 테스크가 있으므로 js파일을 모듈로하고 여러 task를 담을 수 있게
* Task 공유: npm 처럼 개발자가 task, action을 등록하고 다른 사람이 사용 가능하도록 (TOBE)
* Task Graph: sequence, if, while등의 조건을 task graph에서 편집 가능하게, task ui 와 결합하면 코딩없이 task를 개발 (TOBE)

### Task 기능 구현 소스
금융봇
[moneybot/faq.js](https://github.com/com2best/bot-web/blob/0.9/custom_modules/moneybot/faq.js) 게시판 크롤링 테스크
[moneybot/fss-crawling.js](https://github.com/com2best/bot-web/blob/0.9/custom_modules/moneybot/fss-crawling.js) 금융정보 크롤링 테스크
배달봇
[order/franchise/lotteria.js](https://github.com/com2best/bot-web/blob/0.9/custom_modules/order/franchise/lotteria.js)   http 롯데리아 자동주문 테스크 
[order/menuData.js](https://github.com/com2best/bot-web/blob/0.9/custom_modules/order/menuData.js) webdriver 배달의민족, 요기요 데이터 수집 테스크

## 4. Context

context는 해당 task에 제한되지 않은 여러정보들을 담고 있습니다. 이를 통해 현재 사용자의 입력에 해당되는 정보뿐만 아니라, 사용자 정보 등 다양한 상황에 따른 처리를 할 수 있습니다. 
현재 정의된 상황별 context는 다음과 같습니다.

context.user: 사용자의 정보를 담고 있다.
context.bot: 현재 봇의 정보를 담고 있다.
context.botUser: 현재 봇을 사용하는 사용자의 정보를 담고 있다.
context.dialog: Dialog가 여러단계로 이루어지는 경우 대화의 문맥을 파악하기 위해 사용 한다.
머니브레인 봇의 모든 데이터는 JSON을 기반으로 하므로, bot 개발시 필요한 경우 각 context 수준에 맞게 추가적인 key를 정의하여 사용할 수 있습니다.
#3.4. Task 간 데이터 전달

Task의 연결관계, 상하구조에서 데이터를 전달하기 위해서 기본적으로 preCallback, postCallback을 사용할 수 있다.

아래 예시에서는 preCallback을 통해 sequence 등의 이전 task에서 query 값을 가져와서 task.param으로 사용한다. postCallback을 통해서는 task에서 처리한 result.query 값을 topTask에 저장해 놓는다. 

	var sampleTask =
	{
	  name: 'sample',
	  preCallback: function(task, context, callback) {
		  task.param.query = task.preTask.result.query;
		  callback(task, context);
	  },
	  action: sampleAction,
	  postCallback: function(task, context, callback) {
		  task.topTask.param.query = task.result.query;
		  callback(task, context);
	  }
	};

아래와 같이 task 속성을 통해 데이터를 전달할 수도 있다.

	var sampleTask = {
	  name: 'sample',
	  data: {
		set: 'replace',
		context: {
		  user: ['result.doc']
		}, 
		task: {
		  topTask: ['result.title'], 
		  parent: ['param.title']
		}
	  },
	  action: sampleAction
	};

	var sampleTask = {
	  name: 'sample',
	  data: {
		get: 'merge',		
  	    context: ['bot.botName', 'user.mobile'], 
	    task: ['topTask.name', parent.result.doc']
	  },
	  action: sampleAction
	};


data 속성 안내 두가지 key 값으로 구분을 한다 

* get: context 나 다른 task에서 값을 가져와 현재 task에 저장한다. 
* set: 현재 task의 결과를 context 나 다른 task에 저장한다. 

데이터를 전달하는데 다음의 옵션들이 있다. 

* replace: 기본 옵션으로 같은 key의 데이터가 있으면 대체한다.
* merge: 같은 key의 기존 데이터가 있으면 다른 부분만 새로 겹쳐서 합친다.
* concat: 데이터가 array인 경우 두 array를 합쳐서 저장한다. array concat 함수와 동일


####3.4. Task 및 Action 함수 호출과 참조

#####3.4.1. Task 참조

* Dialog의 task 영역에 inline으로 Task를 정의하는 경우

아래와 같이 task를 dialog 입력과  출력사이에 바로 정의하여 사용할 수 있다. 

```
// sample.dlg
< Task 실행
{
  action: function (task, context, callback) {
    task.result = 'hello world';
    callback(task, context);
  }
}
> Task 처리 결과: +result+
```

* Task를 따로 정의하고 dialog 의 task 항목에 참조하는 경우
같은 이름의 dlg 파일과 js 파일은 같은 dlg build 과정을 통해 합쳐지므로, 같은 namespace에 존재한다. 

```
// sample.dlg
< Task 실행
sampleTask
> Task 처리 결과: +result+

// sample.js
var sampleTask =
{
  action: function (task, context, callback) {
    task.result = 'hello world';
    callback(task, context);
  }
};
```

* 다른 곳에 정의한 Action 함수를 사용하는 경우
다른 곳에 정의된  Task를 사용하려면 아래와 같이 String 형식의 Task 이름을 넣는다.

아래 예시처럼 sample.dlg 와 이름이 다른 other.js에 정의된 Task를 사용하려면 String 으로 'sampleTask' 를  넣는다. 다른 곳에서 사용하려면 Task를 등록하는 과정을 거쳐야 한다. 제일 하단의 bot.setTask 을 통하여 등록한다. setTask 통해 등록하는 Task명은 다른 Task와 중복되면 안된다. 

```
// sample.dlg
< Task 실행
'sampleTask'
> Task 처리 결과: +result+

// other.js
var sampleTask =
{
  action: function (task, context, callback) {
    task.result = 'hello world';
    callback(task, context);
  }
};

bot.setTask('sampleTask', sampleTask);
```

기본적으로 task는 nodejs javascript로 구현하므로, node의 require를 통해서 module방식으로 참조할 수도 있다. 

	// sample.dlg
	< Task 실행
	other.sampleTask
	> Task 처리 결과: +result+

	// sample.js
	var path = require('path');
	var other = require(path.resolve('custom_modules/sample/other.js'));

	// other.js
	var sampleTask =
	{
	  action: function (task, context, callback) {
		task.result = 'hello world';
	    callback(task, context);
	  }
	};

#####3.4.2. Action 함수 참조

Action 함수는 Task 정의 내에서 inline function 으로 정의하거나, 별도로 정의하고 참조 할 수 있다. Javascript 에서 JSON과 함수를 참조하는 것과 동일하다.

* Task의 action 항목에 함수를 inline으로 정의하는 경우
```
var sampleTask =
{
  name: 'sample',
  action: function (task, context, callback) {
    task.result = 'hello world';
    callback(task, context);
  }
};
```

* 함수를 따로 정의하고 task 의 action 항목에 참조하는 경우

같은 이름의 dlg 파일과 js 파일은 같은 namespace에 존재하므로 직접 참조가 가능하다. 

```javascript
var sampleTask =
{
  name: 'sample',
  action: sampleAction
};

function sampleAction(task, context, callback) {
  task.result = 'hello world';
  callback(task, context);
}
```

* 다른 곳에 정의한 Action 함수를 사용하는 경우

dlg와 같은이름의 js 파일이 아닌 다른 곳에 정의된  action 함수를 사용하려면 아래와 같이 String 형식의 action 이름을 넣는다.

아래 예시처럼 sample.dlg 와 이름이 다른 other.js에 정의된 action 함수를 사용하려면 String 으로 'sampleAction' 를  넣는다. 다른 곳에서 사용하려면 Action 함수를 등록하는 과정을 거쳐야 한다. 제일 하단의 bot.setAction 을 통하여 등록한다. setAction 통해 등록하는 action명은 다른 action 과 중복되면 안된다. 

```
// sample.dlg
var sampleTask =
{
  name: 'sample',
  action: 'sampleAction'
};

// other.js
function sampleAction(task, context, callback) {
  task.result = 'hello world';
  callback(task, context);
}

bot.setAction('sampleAction', sampleAction);
```

####3.4. Template Task 

Template task를 사용해서 기존에 있던 task를 재사용하여 확장할 수 있다. task의 template 항목에 상속받을 template task 참조를 넣는다. 참조의 방법은 일반 task참조와 동일하다.

아래의 예시에서는 googleTask의 url은 그대로 사용하고, moneybrainTask에서 task.param.q 만 변경하여 요청한다.

	var moneybrainTask = {
	  name: 'moneybrainTask',
	  template: googleTask,
	  param: {
		  q: 'moneybrain.ai'
	  }
	};

	var googleTask = {
		url: 'https://www.google.com',
		param: {
			q: ''
		},
		action: httpAction,
	}


####3.3. Common Task 사용하기



###4. Type

####4.1. 사용자 입력에서 Type 넣기

####4.2. Type 정의

아래와 같이 Type의 정의는 JSON으로 하고, typeCheck 함수를 통해 match 여부를 결정한다. 

```
var sampleType = {
	name: 'sampleType',
	typeCheck: function (text, type, task, context, callback) {
	  var matched = true; 매치여부
	  callback(text, task, matched);
	}
}
```

typeCheck 함수의 파라미터는 다음과 같다.

* text: 사용자에서 입력받은 내용이 넘어온다. NLP 처리가 된 문장이 넘어온다.
* type: type 개체 자체
* task: task개체로 match된 값은 task.param 저장하여 task 처리에 사용한다.
* context: context 개체
* callback: match처리 후 비동기 방식으로 callback을 호출한다. 


callback 함수 호출 시 파라미터는 다음과 같다.

* text: 입력 paramter를 다시 넘겨 준다. 
* task: 처리할 task
* matched: 매치 여부

####4.3. Common Type 사용하기

## 0. 자료보기 순서
chatIF

## 1. Dialog
dialog = {
    orgInput: []
    orgOutput: [] 
    input: {
        text: String
        nlp: []
        intents: []
        entities: []
        types: {}
        sentence
        emotion
    }
    task: 
    output: {
        
    }       
}


###  orgInput
    text
    if
    intent
    entity
    types
    function
    
###  Output
output format
    반복 ##
    치환 ++
image
buttons
list

### Actions
    call
    up
    back ?
    repeat
    returnCall 
    return 
    callChild
    
    options: {output, prefix, postfix, returnDialog}
    
## 2. Context
context = {
    bot: {}
    channel: {}
    user: {}
    session: {}
    topic: {}
}


AS-IS
context =
{
    bot: {
    
      "_id": "59700f7f3080aa1802291ce1",
      "user": "578e5cdc06c7dc1d3e86649a",
      "id": "samplegraph",
      "name": "샘플 그래프",
      "description": "샘플",
      "imageFile": "/files/default.png",
      "created": "2017-07-20T02:03:43.438Z",
      "learning": false,
      "followed": 0,
      "dialogsets": [
        "5a1d306e0e6062d7da0a57ea"
      ],
      "learn": false,
      "public": false,
      "using": false,
      "facebook": false,
      "line": false,
      "kakao": false,
      "language": "ko",
      "dialogs": []
    }  
    channel: {
      "name": "socket"
    }`
    global: {
      "dialogs": {},
      "commonDialogs": {},
      "tasks": {},
      "actions": {},
      "types": {},
      "typeChecks": {},
      "concepts": {},
      "messages": {}
    }
    dialog : {
      "inRaw": "안녕",
      "inNLP": "안녕",
      "typeMatches": {},
      "typeInits": {},
      "isFail": false,
      "typeDoc": null,
      "page": null,
      "numOfPage": null
    }
}


## 5. Bot

bot : {
    id
    name
    description
    use
    public
    user
    language
    
    kakao: true
    facebook: true
    line: true
    topicKeywors: []
    
    startDialog
    noDialog
    dialogs
    commonDialogs
    dialogsets
    dialogsetContexts
    contexts
    types
    typeChecks
    tasks
    actions
    intents
    entities
    concepts
    messages
}

botUser: {
{
  "orgBot": {}
  
}

### Bot 고려사항

* Bot봇 호출: 어떤 봇에서 "ㅇㅇㅇ봇 불러줘" 하면 다른 봇으로 변경됨

### Bot 구현 소스

[Playchat bot](https://github.com/com2best/bot-web/tree/0.9/custom_modules/playchat): 어떤 봇 불러줘, 인기봇 등 찾아서 바로 사용
[Global Dialog](https://github.com/com2best/bot-web/blob/0.9/custom_modules/global/global-dialogs.js): 봇전환 등 기능 있음

## 6. Knowledge

### Knowledge 구현 소스
[지식그래프](https://test.moneybrain.ai/developer/bots/graph-knowledge/58d2734ae10743e31278f86b)



## Task

### basic
task = {
    name:
    preCallback: function(dialog, context, callback)
    action: function(dialog, context, callback) 
    postCallback: function(dialog, context, callback)       
}

action function(dialog, context, callback) {
}

### Task Template(상속)

### Task 조합
    Sequence
    while
    if
    data merge option

## Common Modules
##common Type

* listType
* +amount+: 금액, 십백천만억원 으로 끝나는 말
* +mobile+: 휴대폰 번호
* +phone+: 전화번호
* +date+: 날짜
* +time+: 시간
* +account+: 계좌번호, 숫자와 -로 이루어진 문자열

##common Task
    mongoDB
    http
    xpath
    regexp




<!--stackedit_data:
eyJoaXN0b3J5IjpbLTE0Mjk5MDUwMDhdfQ==
-->