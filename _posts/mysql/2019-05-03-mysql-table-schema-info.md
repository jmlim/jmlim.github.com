---
layout: post
title:  "[MySQL] 테이블 스키마 정보 조회하기."
date:   2019-05-03 16:50:00 +0900
categories: Mysql
comments: true
tags: [Mysql, DB]
---

---


### 테이블 필드정보 1

```roomsql
SELECT
 a.TABLE_NAME '테이블명',
 b.ORDINAL_POSITION '순번',
 lower(b.COLUMN_NAME) '필드명',
 upper(b.DATA_TYPE) 'DATA TYPE',
 b.COLUMN_TYPE '데이터길이',
 b.COLUMN_DEFAULT '디폴트값',
 b.COLUMN_KEY 'KEY',
 b.IS_NULLABLE 'NULL값여부',
 b.EXTRA '자동여부',
 b.COLUMN_COMMENT '필드설명'
from information_schema.TABLES a JOIN information_schema.COLUMNS b on a.TABLE_NAME = b.TABLE_NAME and a.TABLE_SCHEMA = b.TABLE_SCHEMA
where a.TABLE_SCHEMA = '테이블 스키마명' 
ORDER BY
  a.TABLE_NAME, b.ORDINAL_POSITION
;
```


### 테이블 필드정보 2
```roomsql
SELECT 
  b.TABLE_NAME,
  b.ORDINAL_POSITION,
  lower(b.COLUMN_NAME) '필드명',
  upper(b.DATA_TYPE) 'DATA TYPE',
  b.CHARACTER_MAXIMUM_LENGTH,  
  b.COLUMN_DEFAULT,
  CASE WHEN b.IS_NULLABLE = 'NO' THEN 'NOT NULL' ELSE '' END NULLABLE,
  b.COLUMN_COMMENT
FROM information_schema.COLUMNS b
WHERE TABLE_SCHEMA = '테이블 스키마명'
ORDER BY TABLE_NAME, b.ORDINAL_POSITION
;
```


### 인덱스 정보 (프라이머리 제외)
```roomsql
SELECT TABLE_NAME, INDEX_NAME, GROUP_CONCAT(COLUMN_NAME SEPARATOR ','), UNIQ
FROM (
	SELECT TABLE_NAME, SEQ_IN_INDEX, INDEX_NAME, LOWER(COLUMN_NAME) COLUMN_NAME, CASE WHEN NON_UNIQUE = 0 THEN 'UNIQUE' ELSE '' END UNIQ 
	FROM INFORMATION_SCHEMA.STATISTICS
) AS DT
WHERE INDEX_NAME <> 'PRIMARY'
GROUP BY TABLE_NAME, INDEX_NAME, UNIQ
ORDER BY TABLE_NAME
;
```


## 테이블 코멘트 조회
```roomsql
SELECT TABLE_NAME, TABLE_COMMENT from information_schema.TABLES
WHERE TABLE_SCHEMA = '테이블 스키마명'
ORDER BY TABLE_NAME
;
```



[jekyll-docs]: https://jekyllrb.com/docs/home
[jekyll-gh]:   https://github.com/jekyll/jekyll
[jekyll-talk]: https://talk.jekyllrb.com/
---
