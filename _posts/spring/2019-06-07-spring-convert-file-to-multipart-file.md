---
layout: post
title:  "[Spring] 스프링에서 File객체를 MultipartFile로 변환하기."
date:   2019-06-07 19:00:00 +0900
categories: Spring
comments: true
tags: [Spring, Spring boot, MultipartFile, File, Java]
---

---

스프링에서 파일을 업로드 하려고 하는데, MultipartFile로 업로드를 해야만 하고 File로만 받아올 수 있는 상황이 백엔드에서 파일 다루다 보면 종종 발생한다.
그래서 File > MultipartFile 변환을 해 주어야 하는데 방법을 찾기가 쉽지는 않았다.

일반적이진 않지만 몇가지 방법이 있어 소개한다.

## 1. 스프링 테스트의 MockMultipartFile 사용

```java
import org.springframework.mock.web.MockMultipartFile;
....

MultipartFile multipartFile = new MockMultipartFile("test.xlsx", new FileInputStream(new File("/home/admin/test.xlsx")));

```

or

```java
import org.springframework.mock.web.MockMultipartFile;
....
String writerData = "str1,str2,str3,str4";
MultipartFile multipartFile = new MockMultipartFile("files", "파일명.csv", "text/plain", writerData.getBytes(StandardCharsets.UTF_8));

```


## 2. IOUtils 사용

1번 방법이 간편하긴 하나.. 테스트를 runtime에 올려햐 하며 사용이 불가능하다는 커뮤니티 글도 있었다. <br/>
그럴땐 아래와 같은 방법으로 변환이 가능하다.

```java

File file = new File("/path/to/file");
FileItem fileItem = new DiskFileItem("mainFile", Files.probeContentType(file.toPath()), false, file.getName(), (int) file.length(), file.getParentFile());

try {
    InputStream input = new FileInputStream(file);
    OutputStream os = fileItem.getOutputStream();
    IOUtils.copy(input, os);
    // Or faster..
    // IOUtils.copy(new FileInputStream(file), fileItem.getOutputStream());
} catch (IOException ex) {
    // do something.
}

MultipartFile multipartFile = new CommonsMultipartFile(fileItem);

```


### 참고자료
 - https://okky.kr/article/441770
 - https://stackoverflow.com/questions/16648549/converting-file-to-multipartfile

[jekyll-docs]: https://jekyllrb.com/docs/home
[jekyll-gh]:   https://github.com/jekyll/jekyll
[jekyll-talk]: https://talk.jekyllrb.com/

---
