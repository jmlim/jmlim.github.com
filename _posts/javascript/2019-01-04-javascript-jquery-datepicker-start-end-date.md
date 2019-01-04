---
layout: post
title:  "jQuery datepicker 사용 시 종료일을 시작일보다 큰 날짜만 지정 가능하도록 설정하기."
date:   2019-01-04 13:50:00 +0900
categories: Javascript
comments: true
tags: [Javascript, JQuery, datepicker]
---

---

### datepicker 관련 Jquery UI load

```html
<link rel="stylesheet" href="//code.jquery.com/ui/1.12.1/themes/base/jquery-ui.css">
<script src="https://code.jquery.com/jquery-1.12.4.js"></script>
<script src="https://code.jquery.com/ui/1.12.1/jquery-ui.js"></script>
```

### 달력을 표시할 startDate, endDate 에 해당하는 inputbox 추가. 

```html
<input type="text" id="startDate"/>
<input type="text" id="endDate"/>
```

### datepicker 적용.

```html
<script type="text/javascript">
$(document).ready(function () {
	$("#startDate").datepicker({
		dateFormat: "yy-mm-dd", // 날짜의 형식
		minDate: 0,
		nextText: ">",
		prevText: "<",
		onSelect: function (date) {
			var endDate = $('#endDate');
			var startDate = $(this).datepicker('getDate');
			var minDate = $(this).datepicker('getDate');
			endDate.datepicker('setDate', minDate);
			startDate.setDate(startDate.getDate() + 30);
			endDate.datepicker('option', 'maxDate', startDate);
			endDate.datepicker('option', 'minDate', minDate);
		}
	});
	$('#endDate').datepicker({
		dateFormat: "yy-mm-dd", // 날짜의 형식
		nextText: ">",
		prevText: "<"
	});
});
</script>
```


[jekyll-docs]: https://jekyllrb.com/docs/home
[jekyll-gh]:   https://github.com/jekyll/jekyll
[jekyll-talk]: https://talk.jekyllrb.com/
---
