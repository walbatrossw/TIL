
본 포스팅은 [코드로 배우는스프링 웹프로젝트](http://www.yes24.com/24/goods/19720776?scode=032&OzSrank=1)를 참조하여 작성한 내용입니다. 개인적으로 학습한 내용을 복습하기 위한 내용이기 때문에 내용상 오류가 있을 수 있습니다. 기존의 Spring MVC 관련 포스팅들이 제대로 정리되지 않은 것 같아 처음부터 차분히 정리하면서 포스팅을 진행하고 있습니다.

그리고 본 포스팅의 예제는 STS 또는 Eclipse를 사용하지 않고 IntelliJ를 통해 구현하고 있습니다. 그래서 기존의 STS에서 생성된 Spring 프로젝트의 스프링관련 설정 파일명과 프로젝트 구조가 약간 다를 수 있습니다. [IntelliJ를 통한 Spring MVC 프로젝트 생성](http://doublesprogramming.tistory.com/171?category=667155) 포스팅을 참고해주시면 감사하겠습니다.

포스팅하고 있는 현재 프로젝트의 예제가 혹시 필요하신 분은 깃주소(https://github.com/walbatrossw/spring-mvc-ex)를 통해 얻으실 수 있습니다.

---

#### Spring MVC 기본 개념 및 테스트 정리 포스팅 링크
|순서|포스팅 제목|
|:---:|---|
|1|[IntelliJ에서 Spring MVC Project 생성하기](http://doublesprogramming.tistory.com/171)|
|2|[Spring MVC - MySQL 연결테스트](http://doublesprogramming.tistory.com/172)|
|3|[Spring MVC - MyBatis 설정 및 테스트](http://doublesprogramming.tistory.com/173)|
|4|[Spring MVC 구조](http://doublesprogramming.tistory.com/174)|
|5|[Spring MVC - Controller 작성 연습, WAS없이 Controller 테스트 해보기](http://doublesprogramming.tistory.com/175)|
|6|[Spring MVC + MyBatis](http://doublesprogramming.tistory.com/176)|
|7|[Spring - RestController, ResponseEntity, AJAX](http://doublesprogramming.tistory.com/204)|

#### Spring-MVC 게시판 예제  이전 포스팅 링크
|순서|포스팅 제목|
|:---:|---|
|1|[Intellij를 이용한 Spring-MVC 프로젝트 생성 및 세팅](http://doublesprogramming.tistory.com/177)|
|2|[Bootstrap 템플릿 세팅 (AdminLTE)](http://doublesprogramming.tistory.com/178)|
|3|[기본적인 CRUD 구현 : Persistence, Business 계층](http://doublesprogramming.tistory.com/195)|
|4|[기본적인 CRUD 구현 : Control, Presentation 계층](http://doublesprogramming.tistory.com/196)|
|5|[예외처리](http://doublesprogramming.tistory.com/197)|
|6|[페이징처리 : Persistence, Business 계층](http://doublesprogramming.tistory.com/198)|
|7|[페이징처리 : Control, Presentation 계층](http://doublesprogramming.tistory.com/199)|
|8|[페이징처리 개선, 목록페이지 정보 유지](http://doublesprogramming.tistory.com/200)|
|9|[프로젝트 구조 변경 및 수정사항](http://doublesprogramming.tistory.com/201)|
|10|[검색처리, 동적 SQL](http://doublesprogramming.tistory.com/202)|
|11|[AJAX 댓글처리 : Persistence, Business, Control 계층](http://doublesprogramming.tistory.com/205)|
|12|[AJAX 댓글처리 : Presentation 계층](http://doublesprogramming.tistory.com/206)|
|13|[AOP를 이용한 LogAdvice 작성](http://doublesprogramming.tistory.com/207)|
|14|[댓글 갯수, 게시글 조회수 구현, 트랜잭션처리](http://doublesprogramming.tistory.com/208)|
---

# Spring MVC 게시판 예제15 - 게시판 파일 첨부 기능 구현, AJAX방식의 업로드처리

게시판에 파일 업로드 기능을 추가해보자. 파일 업로드 방식에는 여러가지 방식이 있지만 크게 두 가지로 나누어 본다면 아래와 같다.

- **`<form>`태그와 같이 전송하는 방식** : `<input type="file">`태그를 통해 파일을 선택하고, `<form>`태그의 데이터를 전송할 때 파일 데이터도 함께 전송한다.
- **`<form>`태그의 데이터와 파일 데이터를 구분해서 전송하는 방식** : 파일 데이터를 완전히 별도로 전송하고, `<form>`태그의 데이터는 나중에 전송한다.

위의 두 가지 방식의 차이점은 한번에 `<form>`태그 데이터와 파일 데이터를 함께 전송하는지 아니면 파일 데이터를 따로 전송하는가이다. 보통 따로 전송할 경우에는 AJAX방식을 통해 파일 데이터를 전송하게된다.

## 1. 파일 업로드를 위한 준비
우선 파일 업로드를 처리하기 위해서는 몇 가지 설정이 필요하다. 가장 먼저 아래와 같이 파일 업로드 관련 라이브러리를 추가해주고, 파일 업로드 `MultipartResolver`설정을 해준다.

#### # 파일 업로드 관련 라이브러리 추가 : `pom.xml`
```xml
<!--파일 업로드 라이브러리-->
<dependency>
    <groupId>commons-fileupload</groupId>
    <artifactId>commons-fileupload</artifactId>
    <version>1.3.2</version>
</dependency>
```
`imgscalr`라이브러리는 큰 이미지 파일을 고정된 크기로 변환할 때 사용한다.
```xml
<!--이미지 축소 라이브러리-->
<dependency>
    <groupId>org.imgscalr</groupId>
    <artifactId>imgscalr-lib</artifactId>
    <version>4.2</version>
</dependency>
```

#### # 파일업로드 Bean 설정 : `dispatcher-servlet.xml`
스프링에서 파일 업로드를 처리하기 위해서는 파일 업로드로 들어오는 데이터를 처리하는 객체가 필요하기 때문에 아래와 같이 설정을 해야한다. 스프링에서 `multipartResolver`라고 하는 이 객체의 설정은 웹과 관련 있기 때문에 `dispatcher-servlet.xml`에서 설정해준다.
```xml
<!--파일 업로드 MultipartResolver 설정-->
<bean id="multipartResolver" class="org.springframework.web.multipart.commons.CommonsMultipartResolver">
    <property name="maxUploadSize" value="10485760"/>
</bean>
```
`property`중에서 `maxUploadSize`를 통해 파일의 최대 크기를 10MB정도로 제한해준다.

#### # 한글파일명 필터 설정 : `web.xml`
한글 이름으로 된 파일을 업로드했을 때 파일명이 깨져서 나오는 것을 방지하기 위해 아래와 같이 필터 설정이 되어 있는지 확인한다.
```xml
<!--한글 인코딩 설정-->
<filter>
    <filter-name>encoding</filter-name>
    <filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>
    <init-param>
        <param-name>encoding</param-name>
        <param-value>UTF-8</param-value>
    </init-param>
</filter>
<filter-mapping>
    <filter-name>encoding</filter-name>
    <url-pattern>/*</url-pattern>
</filter-mapping>
```

## 2. 파일 업로드 컨트롤러 작성 : `ArticleFileController`
AJAX 방식으로 파일을 업로드 처리하기 위해 `@RestController`애너테이션을 사용한 컨트롤러를 `/src/main/java/기본클래스/upload`패키지에 아래와 같이 컨트롤러 클래스를 생성하고, 코드를 작성해준다. 이 컨트롤러에는 앞으로 파일을 업로드, 파일출력, 파일목록, 파일삭제 메서드등이 추가될 것이다.
```java
@RestController
@RequestMapping("/article/file")
public class ArticleFileController {
    // 파일 업로드 처리 : /article/file/upload
    // 파일 출력 : /article/file/display
    // 파일 목록 : /article/file/list
    // 파일 삭제1(게시글 입력페이지) : /article/file/delete
    // 파일 삭제2(게시글 수정페이지) : /article/file/delete/{articleNo}
    // 파일 삭제3(게시글 삭제처리) : /article/file/deleteAll
}
```

## 3. 파일 업로드를 위한 유틸 클래스 작성 : `UploadFileUtils`, `MediaUtils`

서버에 파일을 업로드 처리할 때 발생할 수 있는 문제점과 그에 대한 해결 방법은 다음과 같다.

- **파일명의 중복저장** : UUID를 이용하여 고유 값을 생성하고, 파일명 앞에 붙여 중복 문제를 해결한다.
- **파일의 단일 저장경로** : 단일 경로에 계속 파일을 저장하게 되면 나중에 파일을 검색하고 찾는데 속도 문제가 발생할 수 있기 때문에 파일을 날짜별 폴더를 생성해 관리한다.
- **이미지 파일인 경우 브라우저에 출력할 크기** : 업로드한 이미지 파일이 크기가 클 경우 서버에서 브라우저에 많은 양의 데이터를 전송하게 되므로 이미지 파일의 축소본, 즉 썸네일을 생성해서 최소한의 데이터를 전송하도록 한다.

위에서 살펴본 문제점과 해결 방안을 고려하여 파일 업로드를 위한 유틸 클래스 2개를 작성할 것인데 그 클래스들의 역할은 다음과 같다.

- `UploadFileUtils` : 파일 업로드/삭제/전송, 폴더생성, 파일명 중복방지 등의 기능을 처리할 메서드들을 가진 유틸 클래스
- `MediaUtils` : 파일의 타입이 이미지인지를 판별하는 메서드를 가진 유틸 클래스

#### # `UploadFileUtils` 클래스 작성
`/src/main/java/기본패키지/commons/util`패키지에 파일 업로드와 관련된 기능을 처리할 `UploadFileUtils`클래스를 아래와 같이 작성해준다.
유틸 클래스로 사용하기 때문에 기본적으로 클래스 내부의 모든 메서드는 `static`메서드로 선언하여 인스턴스 생성 없이 바로 사용이 가능하도록 하였다. 그리고 파일 업로드 처리, 삭제, 출력을 위한 httpHeader설정, 기본경로 추출 메서드는 `public`메서드로 선언하고 파일 업로드 컨트롤러에서 바로 접근하여 사용이 가능하게 했다.
```java
public class UploadFileUtils {

    private static final Logger logger = LoggerFactory.getLogger(UploadFileUtils.class);

    // 파일 업로드 처리 메서드
    public static String uploadFile(String originalFileName, byte[] fileData, HttpServletRequest request) throws Exception {

        // 1. 파일명 중복 방지
        String uuidFileName = getUuidFileName(originalFileName);

        // 2. 파일 업로드 경로 설정
        String rootPath = getRootPath(originalFileName, request); // 기본경로 추출(이미지 or 일반파일)
        String datePath = getDatePath(rootPath); // 날짜 경로 추출, 날짜 폴더 생성

        // 3. 서버에 파일 저장
        File target = new File(rootPath + datePath, uuidFileName); // 파일 객체 생성
        FileCopyUtils.copy(fileData, target); // 파일 객체에 파일 데이터 복사

        // 4. 이미지 파일인 경우 썸네일이미지 생성
        if (MediaUtils.getMediaType(getFormatName(originalFileName)) != null) {
            uuidFileName = makeThumbnail(rootPath, datePath, uuidFileName);
        }

        // 5. 파일 저장 경로 치환
        String savedFilePath = replaceSavedFilePath(datePath, uuidFileName);

        return savedFilePath;
    }

    // 파일 삭제
    public static void deleteFile(String fileName, HttpServletRequest request) {

        String rootPath = getRootPath(fileName, request); // 기본경로 추출(이미지 or 일반파일)

        // 1. 원본 이미지 파일 삭제
        MediaType mediaType = MediaUtils.getMediaType(getFormatName(fileName));
        if (mediaType != null) {
            String originalImg = fileName.substring(0, 12) + fileName.substring(14);
            new File(rootPath + originalImg.replace('/', File.separatorChar)).delete();
        }

        // 2. 파일 삭제(썸네일이미지 or 일반파일)
        new File(rootPath + fileName.replace('/', File.separatorChar)).delete();
    }

    // 파일 출력을 위한 HttpHeader 설정
    public static HttpHeaders getHttpHeaders(String fileName) throws Exception {

        MediaType mediaType = MediaUtils.getMediaType(getFormatName(fileName)); // 파일타입 확인
        HttpHeaders httpHeaders = new HttpHeaders(); // HttpHeder객체 생성

        if (mediaType != null) { // 이미지 파일 O
            httpHeaders.setContentType(mediaType);
        } else { // 이미지 파일 X
            fileName = fileName.substring(fileName.indexOf("_") + 1); // UUID 제거
            httpHeaders.setContentType(MediaType.APPLICATION_OCTET_STREAM); // 다운로드 MIME 타입 설정
            // 파일명 한글 인코딩처리
            httpHeaders.add("Content-Disposition", "attachment; filename=\"" +
                    new String(fileName.getBytes("UTF-8"), "ISO-8859-1")+"\"");
        }

        return httpHeaders;
    }

    // 기본 경로 추출
    public static String getRootPath(String fileName, HttpServletRequest request) {

        String rootPath = "/resources/upload";
        MediaType mediaType = MediaUtils.getMediaType(getFormatName(fileName)); // 파일타입 확인
        if (mediaType != null)
            return request.getSession().getServletContext().getRealPath(rootPath + "/images"); // 이미지 파일 경로

        return request.getSession().getServletContext().getRealPath(rootPath + "/files"); // 일반파일 경로
    }

    // 날짜 폴더명 추출
    private static String getDatePath(String uploadPath) {

        Calendar calendar = Calendar.getInstance();
        String yearPath = File.separator + calendar.get(Calendar.YEAR);
        String monthPath = yearPath + File.separator + new DecimalFormat("00").format(calendar.get(Calendar.MONTH) + 1);
        String datePath = monthPath + File.separator + new DecimalFormat("00").format(calendar.get(Calendar.DATE));

        makeDateDir(uploadPath, yearPath, monthPath, datePath);

        return datePath;
    }

    // 날짜별 폴더 생성
    private static void makeDateDir(String uploadPath, String... paths) {

        // 날짜별 폴더가 이미 존재하면 메서드 종료
        if (new File(uploadPath + paths[paths.length - 1]).exists())
            return;

        for (String path :  paths) {
            File dirPath = new File(uploadPath + path);
            if (!dirPath.exists())
                dirPath.mkdir();

        }
    }

    // 파일 저장 경로 치환
    private static String replaceSavedFilePath(String datePath, String fileName) throws Exception {
        String savedFilePath = datePath + File.separator + fileName;
        return savedFilePath.replace(File.separatorChar, '/');
    }

    // 파일명 중복방지
    private static String getUuidFileName(String originalFileName) {
        return UUID.randomUUID().toString() + "_" + originalFileName;
    }

    // 파일 확장자 추출
    private static String getFormatName(String fileName) {
        return fileName.substring(fileName.lastIndexOf(".") + 1);
    }

    // 썸네일 이미지 생성
    private static String makeThumbnail(String uploadRootPath, String datePath, String fileName) throws Exception {

        BufferedImage originalImg = ImageIO.read(new File(uploadRootPath + datePath, fileName)); // 원본이미지를 메모리상에 로딩
        BufferedImage thumbnailImg = Scalr.resize(originalImg, Scalr.Method.AUTOMATIC, Scalr.Mode.FIT_TO_HEIGHT, 100); // 원본이미지를 축소
        String thumbnailImgName = "s_" + fileName; // 썸네일 파일명 설정
        String fullPath = uploadRootPath + datePath + File.separator + thumbnailImgName; // 썸네일 업로드 경로
        File newFile = new File(fullPath); // 썸네일 파일 객체생성

        String formatName = getFormatName(fileName); // 썸네일 파일 확장자 추출
        ImageIO.write(thumbnailImg, formatName.toUpperCase(), newFile); // 썸네일 파일 저장

        return thumbnailImgName;
    }

}
```


#### # `MediaUtils`
`UploadFileUtils`클래스와 마찬가지로 `/src/main/java/기본패키지/commons/util`패키지에 파일 타입이 이미지인지를 판별해주는 `MediaUtils`클래스를 아래와 같이 작성했다.
```java
public class MediaUtils {

    private static Map<String, MediaType> mediaTypeMap;

    // 클래스 초기화 블럭
    static {
        mediaTypeMap = new HashMap<>();
        mediaTypeMap.put("JPG", MediaType.IMAGE_JPEG);
        mediaTypeMap.put("GIF", MediaType.IMAGE_GIF);
        mediaTypeMap.put("PNG", MediaType.IMAGE_PNG);
    }

    // 파일 타입
    public static MediaType getMediaType(String formatName) {
        return mediaTypeMap.get(formatName.toUpperCase());
    }

}
```

## 4. 파일 업로드 컨트롤러 메서드 작성

#### # 파일 업로드 처리 : 게시글 입력페이지, 수정페이지에서 사용

```java
// 게시글 파일 업로드
@RequestMapping(value = "/upload", method = RequestMethod.POST, produces = "text/plain;charset=UTF-8")
public ResponseEntity<String> uploadFile(MultipartFile file, HttpServletRequest request) {
    ResponseEntity<String> entity = null;
    try {
        String savedFilePath = UploadFileUtils.uploadFile(file.getOriginalFilename(), file.getBytes(), request);
        entity = new ResponseEntity<>(savedFilePath, HttpStatus.CREATED);
    } catch (Exception e) {
        e.printStackTrace();
        entity = new ResponseEntity<>(HttpStatus.BAD_REQUEST);
    }
    return entity;
}
```

파일업로드 처리 메서드는 `UploadFileUtils`클래스의 `uploadFile()`메서드에서 파일을 업로드 처리하고 난 뒤 리턴 받은 값, 즉 첨부파일명("/년/월/일/파일명")을 호출한 곳으로 리턴해준다. 이 과정에서 주목할 점은 `@RequestMapping`애너테이션의 `produces`속성인데 `"text/plain;charset=UTF-8"`으로 지정한 것인데 이것은 한국어를 정상적으로 전송하기 위한 설정으로 리턴값이 한글일 경우 정상적으로 한글을 전달해주기 위한 것이다.

#### # 파일 출력 : 게시글 입력페이지, 조회페이지, 수정페이지에서 사용

```java
// 게시글 파일 출력
@RequestMapping(value = "/display", method = RequestMethod.GET)
public ResponseEntity<byte[]> displayFile(String fileName, HttpServletRequest request) throws Exception {

    HttpHeaders httpHeaders = UploadFileUtils.getHttpHeaders(fileName); // Http 헤더 설정 가져오기
    String rootPath = UploadFileUtils.getRootPath(fileName, request); // 업로드 기본경로 경로

    ResponseEntity<byte[]> entity = null;

    // 파일데이터, HttpHeader 전송
    try (InputStream inputStream = new FileInputStream(rootPath + fileName)) {
        entity = new ResponseEntity<>(IOUtils.toByteArray(inputStream), httpHeaders, HttpStatus.CREATED);
    } catch (Exception e) {
        e.printStackTrace();
        entity = new ResponseEntity<>(HttpStatus.BAD_REQUEST);
    }
    return entity;
}
```
AJAX를 통해 파일업로드 처리가 완료되고, 파일을 화면에 출력해주기 위한 파일 출력 메서드를 위와 같이 작성해준다.


#### # 파일 목록 : 게시글 조회페이지에서 사용
```java

```
#### # 파일 삭제 처리1 : 게시글 입력페이지에서 사용

#### # 파일 삭제 처리2 : 게시글 수정페이지에서 사용

#### # 파일 전체 삭제 처리 : 게시글 삭제처리시
