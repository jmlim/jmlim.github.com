---
layout: post
title:  "Spring boot 환경에서 Amazon s3에 이미지파일 업로드하기."
date:   2018-12-06 16:59:00 +0900
categories: Spring
comments: true
tags: [Spring, Java]
---

---

Spring boot 환경에서 Amazon s3에 이미지파일 업로드하기.
--

amazon s3 인증 받기 및 설정 관련 내용은 생략하였다.

프로젝트 구성을 위해 의존성을 추가. 
--
* pom.xml

```xml
....
<dependency>
	<groupId>com.amazonaws</groupId>
	<artifactId>aws-java-sdk</artifactId>
	<version>1.11.313</version>
</dependency>
....
```

* application.yml 설정파일.

```yaml
spring
 aws:
  access-key: 발급받은 accessKey
  secret-key: 발급받은 secretKey
  region: 지역
  bucket: s3 버킷이름
  url: S3 url
``` 

* AWS properties Configuration 추가.

``` java
@Configuration
@ConfigurationProperties(prefix="spring.aws")
public static class AwsProperties{
	private String accessKey;
	private String secretKey;
	private String region;
	private String bucket;
	private String url;

	//<editor-fold desc="getter setter">
	public void setAccessKey(String accessKey){
		this.accessKey=accessKey;
	}

	public void setSecretKey(String secretKey){
		this.secretKey=secretKey;
	}

	public void setRegion(String region){
		this.region=region;
	}

	public String getBucket(){
		return bucket;
	}

	public void setBucket(String bucket){
		this.bucket=bucket;
	}

	public String getUrl(){
		return url;
	}

	public void setUrl(String url){
		this.url=url;
	}
}

```

* AmazonS3Client 빈 등록
 ```java
  
  import com.amazonaws.auth.AWSStaticCredentialsProvider;
  import com.amazonaws.auth.BasicAWSCredentials;
  import com.amazonaws.regions.Regions;
  import com.amazonaws.services.s3.AmazonS3;
  import com.amazonaws.services.s3.AmazonS3ClientBuilder;
  .....  
  @Bean
  public AmazonS3 amazonS3Client(AwsProperties awsProperties){
  	return AmazonS3ClientBuilder.standard().withCredentials(new AWSStaticCredentialsProvider(new BasicAWSCredentials(awsProperties.accessKey, awsProperties.secretKey))).withRegion(Regions.fromName(awsProperties.region)).build();
  }
  ...
 ```

* Controller에서 이미지 업로드 호출 부분.

```java

@Controller
public class UploadController {

	@Autowired 
	private UploadService uploadService
	....
	@PostMapping("/uploadImage")
	public void background(
			@RequestParam("userImage") MultipartFile multipartFile
	){
		String uploadUrl = "업로드 url 정보";
		uploadService.updateBackgroundImage(uploadUrl, multipartFile);
	}
	....
	
}
```

* Service 에서 AmazonS3 에 이미지 업로드 하는 부분.


```java

@Service
public class UploadService {

	@Autowired
	private AmazonS3 amazonS3;
	
	@Autowired
	private AwsProperties awsProperties;
	
	....
	// controller 에서 실행한 부분.
	public void updateBackgroundImage(String url, MultipartFile multipartFile){
        upload(url, multipartFile);
    }
	
	private void upload(String url, MultipartFile file){
        try{
            String uploadFileName = file.getOriginalFilename();
            String fileExtension = uploadFileName.substring(uploadFileName.lastIndexOf(".") + 1, uploadFileName.length());

            String contentTypeTail = "jpeg";
            if (fileExtension.toLowerCase().equals("gif")) contentTypeTail = "gif";
            else if (fileExtension.toLowerCase().equals("png")) contentTypeTail = "png";

            byte[] bytes = file.getBytes();
            upload(url, new ByteArrayInputStream(bytes), contentTypeTail);
        }catch(Throwable t){
            throw new JmlimException(t.getMessage(), HttpStatus.BAD_REQUEST, t);
        }
    }
	
	private void upload(String key, InputStream inputStream, String imageType){
        ObjectMetadata metadata = new ObjectMetadata();
        if(imageType != null) {
            metadata.addUserMetadata("Content-Type", "image/"+imageType);
        }

        PutObjectRequest putObjectRequest = new PutObjectRequest(awsProperties.getBucket(), key, inputStream, metadata);
		// 퍼블릭으로 공개하여 올림.
        putObjectRequest.setCannedAcl(CannedAccessControlList.PublicRead);
        amazonS3.putObject(putObjectRequest);
    }
}
```


[jekyll-docs]: https://jekyllrb.com/docs/home
[jekyll-gh]:   https://github.com/jekyll/jekyll
[jekyll-talk]: https://talk.jekyllrb.com/
---
