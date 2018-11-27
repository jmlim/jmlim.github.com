---
layout: post
title:  "Javascript timestamp 컨버터 (로컬시간 기준)"
date:   2018-11-27 18:08:00 +0900
categories: Javascript
comments: true
tags: [Javascript, JQuery]
---

---
### JQuery timestamp 컨버터 (로컬시간 기준)

개인적으로 자주 쓸 것 같아서 보관용으로 포스팅한다.

```html
<!DOCTYPE html>
<html lang="ko">
<head>
<meta charset="utf-8">
<meta http-equiv="X-UA-Compatible" content="IE=edge">
<title>Timestamp Converter</title>
<script
  src="https://code.jquery.com/jquery-1.12.4.js"
  integrity="sha256-Qw82+bXyGq6MydymqBxNPYTaUXXq7c8v3CwiYwLLNXU="
  crossorigin="anonymous"></script>
<body>
	<textarea id="timestamp_list" style="height: 200px"></textarea>
	<button id="time_btn">변환</button>
	<div id="conv_list">
	</div>
	<script>
		$("#time_btn").on("click", function() {
			var $timestamp_list_val = $("#timestamp_list").val();
			var lists = $timestamp_list_val.split("\n");
			var conv_lists = [];
			$.each(lists, function(ind, val) {
				var convDate = new Date(Number(val));
				conv_lists.push(convDate.toLocaleString());
			});
			$("#conv_list").html(conv_lists.join("<br/>"));
		});
	</script>
</body>
</html>
```

[jekyll-docs]: https://jekyllrb.com/docs/home
[jekyll-gh]:   https://github.com/jekyll/jekyll
[jekyll-talk]: https://talk.jekyllrb.com/
---
