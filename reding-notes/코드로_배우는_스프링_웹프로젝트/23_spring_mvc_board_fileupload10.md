
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

# Spring MVC 게시판 예제15 - AJAX방식의 게시판 첨부파일 기능 구현

이번에는 게시판에 AJAX방식으로 파일첨부기능 구현을 정리해보자.

## 1. 파일 업로드 방식 정리
파일 업로드를 처리하는 방식을 아래와 같이 2가지 방식으로 나눌 수 있다.

- **파일데이터와 `<form>`태그의 데이터를 함께 전송하는 방식** : `<input type="file">`태그를 사용하여 파일은 선택하고, `<form>`데이터와 함께 전송
- **파일데이터와 `<form>`태그의 데이터를 구분해서 전송하는 방식** : AJAX통신을 통해 파일 데이터를 미리 서버에 저장한 뒤 `<form>`데이터를 전송

본 예제에서는 첨부파일 기능 구현을 AJAX방식으로 파일데이터와 `<form>`데이터를 구분해서 전송하는 방식으로 처리하는데 아래와 같은 과정을 거치게 된다.
1. 게시글 제목, 내용, 작성자를 입력
2. 첨부파일이 필요할 경우 탐색기에서 drag & drop방식으로 파일을 끌어다 놓으면 서버에 업로드 처리가 미리 됨
3. 게시글 등록 버튼을 클릭하면 게시글의 정보와 첨부파일의 정보가 함께 DB에 저장됨


## 2. Spring 파일 업로드 설정 추가
스프링에서 파일 업로드를 처리하기 위해서는 몇 가지 설정이 필요하다. 아래와 같이 파일 업로드 관련 라이브러리를 추가해주고, 파일 업로드 `MultipartResolver`설정을 해준다. 그리고 한글파일명이 깨지는 것을 방지하기 위해 필터설정이 되있는지 확인해준다.

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
<!--이미지 크기 변환 라이브러리-->
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
업로드했을 때 한글 파일명이 깨져서 나오는 것을 방지하기 위해 아래와 같이 필터 설정이 되어 있는지 확인한다.
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

## 3. 첨부파일 테이블 추가 및 도메인 클래스 변경
#### # 게시글 첨부파일 테이블 생성
게시글의 첨부파일명을 저장하기 위한 `tbl_article_file`테이블을 생성해준다. 첨부파일의 이름(`file_name`)은 고유한 속성을 가지도록 처리할 예정이기 때문에 기본키로 설정하였고, 참조키는 게시글 번호(`article_no`)로 설정해주었다. 그리고 게시글 테이블(`tbl_article`)에는 게시글 목록페이지에서 첨부파일의 갯수를 출력해주기 위해 칼럼을 아래와 같이 추가해주었다.
```sql
-- 게시글 첨부파일 테이블 생성
CREATE TABLE tbl_article_file (
  file_name VARCHAR(150) NOT NULL ,
  article_no INT NOT NULL ,
  reg_date TIMESTAMP DEFAULT NOW(),
  PRIMARY KEY (file_name)
);

-- 게시글 첨부파일 테이블 참조키 설정
ALTER TABLE tbl_article_file ADD CONSTRAINT fk_article_file
FOREIGN KEY (article_no) REFERENCES tbl_article (article_no);

-- 게시글 테이블 첨부파일 갯수 칼럼 추가
ALTER TABLE tbl_article ADD COLUMN file_cnt INT DEFAULT 0;
```

#### # 게시글 도메인 클래스의 수정 : `ArticleVO`
데이터베이스의 테이블이 변경되었기 때문에 도메인 클래스에도 추가적인 데이터가 필요하게 되었다. 그래서 `ArticleVO`클래스에 파일들의 이름을 의미하는 String배열 타입 변수 `files`와 파일 개수를 의미하는 변수 `fileCnt`를 선언해준다. 그리고 게시글이 입력/수정될 때 생성되는 `ArticleVO`의 인스턴스가 스스로 게시글의 첨부파일 갯수를 상태를 가질 수 있게 `setFiles()`메서드를 아래와 같이 작성해준다. 이렇게 되면 외부에서 `setFiles()`를 호출하여 게시글의 첨부파일 갯수를 넣어주지 않아도 된다.
```java
public class ArticleVO {

    private Integer articleNo;
    private String title;
    private String content;
    private String writer;
    private Date regDate;
    private int viewCnt;
    private int replyCnt;

    private String[] files;
    private int fileCnt;

    // ... getter, setter

    public void setFiles(String[] files) {
        this.files = files;
        setFileCnt(files.length);
    }

    private void setFileCnt(int fileCnt) {
        this.fileCnt = fileCnt;
    }

    // ... toString()
}
```

## 4. 파일 업로드 유틸 클래스 작성

#### # 서버에 파일 업로드할 때 고려사항
서버에 파일을 저장할 때 발생할 수 있는 문제점과 그 해결방안을 정리하면 아래와 같다.

- **동일한 파일명이 저장되는 경우** : UUID를 이용하여 고유 값을 생성하고, 파일명 앞에 붙여 중복 문제를 해결한다.
- **파일의 단일 저장 경로** : 단일 경로에 계속 파일을 저장하게 되면 나중에 파일을 검색하고 찾는데 속도 문제가 발생할 수 있기 때문에 파일을 날짜별 폴더를 생성해 관리한다.
- **이미지 파일인 경우 브라우저에 출력할 크기** : 업로드한 이미지 파일이 크기가 클 경우 서버에서 브라우저에 많은 양의 데이터를 전송하게 되므로 이미지 파일의 축소본, 즉 썸네일을 생성해서 최소한의 데이터를 전송하도록 한다.

위에서 살펴본 문제점과 해결 방안을 고려하여 파일 업로드를 처리하기 위한 유틸 클래스 2개를 작성할 것인데 그 클래스들의 역할은 다음과 같다.

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

#### # `MediaUtils` 클래스 작성
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

## 5. 게시글 입력 페이지에서 첨부파일 기능 구현

#### # 게시글 첨부파일 영속계층 작성 : `ArticleFileDAO` / `ArticleFileDAOImpl`
`/src/main/기본패키지/upload/Persistence`패키지에 업로드한 파일정보를 첨부파일 테이블에 저장하기 위해 `ArticleFileDAO`인터페이스와 `ArticleFileDAOImpl`클래스를 생성하고 코드를 아래와 같이 작성해준다.
```java
public interface ArticleFileDAO {

    // 파일 추가
    void addFile(String fullName) throws Exception;

}
```
```java
@Repository
public class ArticleFileDAOImpl implements ArticleFileDAO {

    private static final String NAMESPACE = "com.doubles.mvcboard.mappers.upload.ArticleFileMapper";

    private final SqlSession sqlSession;

    @Inject
    public ArticleFileDAOImpl(SqlSession sqlSession) {
        this.sqlSession = sqlSession;
    }

    @Override
    public void addFile(String fileName) throws Exception {
        sqlSession.insert(NAMESPACE + ".addFile", fileName);
    }
}
```

#### # 게시글 첨부파일 SQL Mapper : `articleFileMapper.xml`
`articleFileMapper.xml`을 생성하고 아래와 같이 SQL을 작성해준다. 게시글 테이블에 게시글 내용이 추가되는 동시에 첨부파일 테이블에도 게시글 번호가 같이 저장될 수 있도록 처리하기 위해 게시글 번호는 `LAST_INSERT_ID()`를 이용하여 처리해준다.
```sql
<mapper namespace="com.doubles.mvcboard.mappers.upload.ArticleFileMapper">

    <insert id="addFile">
        INSERT INTO tbl_article_file (
            file_name
            , article_no
        ) VALUES (
            #{fileName}
            , LAST_INSERT_ID()
        )
    </insert>

</mapper>
```

#### # 게시글 서비스 계층 수정 : `ArticleServiceImpl`
게시글이 입력 처리가 되면 함께 게시글 첨부파일명이 게시글 첨부파일 테이블에 함께 입력되도록 `ArticleServiceImpl`클래스의 `create()`를 아래와 같이 수정해준다. 그리고 게시글 입력처리와 게시글 첨부파일 입력처리가 동시에 이루어지기 때문에 트랜잭션 처리를 반드시 해주어야한다.
```java
@Transactional
@Override
public void create(ArticleVO articleVO) throws Exception {

    // 게시글 입력처리
    articleDAO.create(articleVO);
    String[] files = articleVO.getFiles();

    if (files == null)
        return;

    // 게시글 첨부파일 입력처리
    for (String fileName : files)
        articleFileDAO.addFile(fileName);

}
```

#### # 게시글 첨부파일 컨트롤러 작성 : `ArticleFileController`
게시글 첨부파일 컨트롤러는 앞서 말한 것처럼 게시글 입력처리가 되기 전에 클라이언트로부터 AJAX 통신을 통해 미리 파일 데이터를 받아 서버에 저장하는 역할을 수행하게 된다.

파일 업로드 처리 메서드는 다음과 같은 과정을 거치게 된다.

1. 클라이언트로부터 전달 받은 `file`의 정보 파일명, 파일데이터, `request`를 `UploadFileUtils.uploadFile()`의 매개변수로 전달한다.
2. `UploadFileUtils`클래스 내부의 메서드들을 통해 파일 업로드 처리가 완료되고, "/년/월/일/파일명"의 문자열을 리턴 받는다.
3. 리턴 받은 문자열을 호출한 클라이언트 화면으로 리턴한다.

아래의 컨트롤러 코드에서 주의할 점은 `@RequestMapping`애너테이션의 `produces`속성을 `"text/plain;charset=UTF-8"`로 지정한 것인데 클라이언트로 한글을 정상적으로 전송하기 위한 설정이므로 반드시 작성해야 한다.
```java
@RestController
@RequestMapping("/article/file")
public class ArticleFileController {

    // 게시글 파일 업로드 처리
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
}
```

그리고 서버에 파일이 업로드가 완료되고 나면, 바로 화면에서 업로드한 파일을 확인할 수 있도록 컨트롤러에 아래와 같이 파일 출력메서드를 작성해준다. 업로드 파일 출력 처리 메서드는 다음과 같은 과정을 거치게 된다.

1. 클라이언트로부터 전달 받은 업로드 파일명(`fileName`)을 매개변수로 `UploadFileUtils`클래스의 `getHttpHeaders()`메서드를 호출한다.
2. `getHttpHeaders()`메서드는 파일명(`fileName`)을 통해 파일타입을 판별하여 적절한 MINE타입을 지정하고, 클라이언트에게 전송할 `HttpHeaders`객체를 리턴한다.
3. 리턴받은 `HttpHeaders`객체와 파일 데이터를 호출한 클라이언트 화면으로 리턴한다.

```java
// 업로드 파일 출력 처리
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

#### # 게시글 입력 페이지 수정 : `write.jsp`
##### 첨부파일 Drag & Drop 영역 추가
게시글 입력페이지에 아래와 같이 CSS와 HTML코드를 추가해준다.
```css
.fileDrop {
    width: 100%;
    height: 200px;
    border: 2px dotted #0b58a2;
}
```
```xml
<div class="box-body">
    <div class="form-group">
        <label for="title">제목</label>
        <input class="form-control" id="title" name="title" placeholder="제목을 입력해주세요">
    </div>
    <div class="form-group">
        <label for="content">내용</label>
        <textarea class="form-control" id="content" name="content" rows="30"
                  placeholder="내용을 입력해주세요" style="resize: none;"></textarea>
    </div>
    <div class="form-group">
        <label for="writer">작성자</label>
        <input class="form-control" id="writer" name="writer">
    </div>
    <%-- Drag & Drop 영역 --%>
    <div class="form-group">
        <div class="fileDrop">
            <br/>
            <br/>
            <br/>
            <br/>
            <p class="text-center"><i class="fa fa-paperclip"></i> 첨부파일을 드래그해주세요.</p>
        </div>
    </div>
    <%-- Drag & Drop 영역 --%>
</div>
```

##### 첨부파일용 Handlebars템플릿 작성
댓글기능을 구현했을 때와 마찬가지로 Handlebars를 이용하여 HTML코드를 동적으로 생성하도록 아래와 같이 코드를 작성해준다.
```xml
<script id="fileTemplate" type="text/x-handlebars-template">
    <li>
        <span class="mailbox-attachment-icon has-img">
            <img src="{{imgSrc}}" alt="Attachment">
        </span>
        <div class="mailbox-attachment-info">
            <a href="{{getLink}}" class="mailbox-attachment-name">
                <i class="fa fa-paperclip"></i> {{fileName}}
            </a>
            <a href="{{fullName}}" class="btn btn-default btn-xs pull-right delBtn">
                <i class="fa fa-fw fa-remove"></i>
            </a>
        </div>
    </li>
</script>
```
```js

```

## 6. 게시글 조회 페이지의 첨부파일 목록 기능 구현

## 7. 게시글 수정 페이지의 첨부파일 기능 구현
