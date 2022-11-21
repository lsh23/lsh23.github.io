---
title: vscode custom snippets 설정
date: 2022-11-11 00:05:00 +0900
categories: [IDE, vscode]
tags: [vscode]
comments: true
---

블로그를 markdown으로 쓸때 여러 md 포맷을 활용해야 하는 경우가 많은데  
해당 포맷들을 vscode에서 custom snippet으로 등록하기


## Snippets 파일 선택
File -> Preferences -> User Snippets -> 원하는 Language 선택
![Desktop View](/posts/20221111/vscode-snippets-1.png){: .normal}

## Snippet 등록 with snippet-generator
prefix가 snippet 자동완성을 불러오는 키워드이고,
body는 자동완성할 body 그 자체이다.  
개행 단위로 끊어서 List 형태로 넣어준다.
description은 해당 snippet에 대한 설명을 의미한다.
```json
	"Todo List": {
		"prefix": "todo",
		"body": [
		  "- [ ] Job",
		  "  + [x] Step 1"
		],
		"description": "ToDO List"
	}
```
{: file='~/.config/Code/User/snippets/markdown.json'}

![Desktop View](/posts/20221111/vscode-snippets-2.png){: .normal}
![Desktop View](/posts/20221111/vscode-snippets-3.png){: .normal}



body 내용이 많아지면 개행 뿐만아니라 공백도 맞춰서 List로 넣어줘야하므로 해당 내용을 직접 만들기 매우 번거로워 진다.  
https://snippet-generator.app 사이트를 이용하면 snippet 설정을 위한 json을 쉽게 만들 수 있다.
![Desktop View](/posts/20221111/vscode-snippets-4.png){: .normal}

## Snippet 적용이 안된다면
만약에 위의 설정을 마쳤는데도 markdown(자기가 추가한 언어)파일에서 snippet 자동완성이 지원이 되지 않는다면
우측하단의 markdown(자기가 추가한 언어) 선택후 Congifrue ~ language based settings... 선택 후

![Desktop View](/posts/20221111/vscode-snippets-5.png){: .normal}
![Desktop View](/posts/20221111/vscode-snippets-6.png){: .normal}

해당 설정 파일에 "editor.quickSuggestions": true 옵션이 있는지 확인하고 넣어준다.

```json
  {
    ...
    "[markdown]": {
        "editor.unicodeHighlight.ambiguousCharacters": false,
        "editor.unicodeHighlight.invisibleCharacters": false,
        "editor.wordWrap": "on",
        "editor.quickSuggestions": true
    },
    ... 
```
{: file='~/.config/Code/User/settings.json'}


## reference
* <https://code.visualstudio.com/docs/editor/userdefinedsnippets#_create-your-own-snippets>
* <https://snippet-generator.app/>