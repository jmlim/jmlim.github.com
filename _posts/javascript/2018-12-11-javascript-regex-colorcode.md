---
layout: post
title:  "Hex Color 코드 관련 자바스크립트 정규식 및 함수 정리."
date:   2018-12-11 11:56:00 +0900
categories: Javascript
comments: true
tags: [Javascript, Hexcode, Regex, 정규표현식]
---

---


먼저 Hex코드란?  <br/>
--
헥스 코드는 색깔을 #과 뒤에 붙는 여섯 개의 알파벳 + 숫자로 나타낸 것이다. 
숫자는 두 자리씩 끊어서 각각 'R', 'G', 'B'를 나타내며 16진수로 표현되어 00일 때 어둡고 FF일 때 밝다. 
즉 엄밀히 말하면 색이 아니고 빛이다. #FF0000이면 R값이 최고값이고 나머지는 0이므로 순수한 빨간색이 나타난다. #FFFF00이면 R값과 G값이 최고값이고 B값이 0이므로 R과 G의 합성 빛인 노란색이 나타난다.
주로 디지털에서 RGB보다 더 많이 쓰인다. 색상코드 앞엔 반드시 #을 붙여서 하지만 대체로 붙이지 않으면 자동으로 붙여 주는 경우가 많다. 
프로그래머(특히 웹 서버 프로그래머)들은 워낙 이 작업을 매우 많이 해서 어지간한 색들의 색상코드는 거의 다 알고 있다고 해도 과언이 아니다. 
실제로 작업하다보면 자의 반 타의반으로 외우게 된다.

RGB코드는 hex코드를 두자리씩 나눠서 10진수로 변환한 수이다. 
* ex : rgb(255,255,255)

자세한 내용은 나무위키 참조
 - https://namu.wiki/w/%ED%97%A5%EC%8A%A4%20%EC%BD%94%EB%93%9C

hex 코드값 판별하는 정규식. 
--
~~~
/^#[0-9a-f]{3,6}$/i
#abc, #abcd, #abcde, #abcdef 형식과 매칭.

/^#([0-9a-f]{3}|[0-9a-f]{6})$/i
#abc and #abcdef 형식과 매칭. #abcd는 매칭 안됨.

/^#([0-9a-f]{3}){1,2}$/i
#abc and #abcdef 형식과 매칭 #abcd는 매칭안됨.

/^#(?:[0-9a-f]{3}){1,2}$/i
#abc and #abcdef 형식과 매칭 #abcd는 매칭안됨.
~~~
자바 스크립트에서 정규 표현식에 대해 자세한 학습을 원하면 RegExp - MDN을 참고하면 많은 도움이 된다.
- https://developer.mozilla.org/ko/docs/Web/JavaScript/Guide/%EC%A0%95%EA%B7%9C%EC%8B%9D

javascript 에서 정규식 사용하여 유효성 체크를 해보자.
```javascript
var regex = /^#(?:[0-9a-f]{3}){1,2}$/i;
var hex = "qweqwe"; 
//올바른 컬러코드값이 아니므로 아래 메세지 창 실행됨. 
//만약 올바른 컬러코드값 (ex: #ffffff)이 hex 변수에 들어있을 경우 regex.test(hex) => true 이므로 메세지 창 뜨지 않음.
if(!regex.test(hex)) {
  alert("올바른 헥사코드 값이 아닙니다.");
  return false;
}
```

hex코드를 rgb로 변환하는 javascript 함수 만들기.
--
```javascript
function hexToRgb(hex) {
    // Expand shorthand form (e.g. "03F") to full form (e.g. "0033FF")
    var shorthandRegex = /^#?([a-f\d])([a-f\d])([a-f\d])$/i;
    hex = hex.replace(shorthandRegex, function(m, r, g, b) {
        return r + r + g + g + b + b;
    });

    var result = /^#?([a-f\d]{2})([a-f\d]{2})([a-f\d]{2})$/i.exec(hex);
    return result ? {
        r: parseInt(result[1], 16),
        g: parseInt(result[2], 16),
        b: parseInt(result[3], 16)
    } : null;
}
```
실행결과 
> hexToRgb("#ffffff"); //실행. <br>
  {r: 255, g: 255, b: 255} //리턴값.

rgb를 hex코드로 변환하는 javascript 함수만들기.
```javascript
//Function to convert rgb color to hex format
function rgbToHex(rgb) {
	rgb = rgb.match(/^rgb\((\d+),\s*(\d+),\s*(\d+)\)$/);
	return "#" + hex(rgb[1]) + hex(rgb[2]) + hex(rgb[3]);
}
```

실행결과 
> rgbToHex("rgb(255,255,255)"); //실행. <br> #ffffff //리턴값.

참고: 
 * https://stackoverflow.com/questions/9682709/regexp-matching-hex-color-syntax-and-shorten-form
