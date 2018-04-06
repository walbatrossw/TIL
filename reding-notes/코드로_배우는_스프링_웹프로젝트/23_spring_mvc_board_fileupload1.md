
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

## 1. 게시판 첨부파일 업로드 처리 방식 정리

파일 업로드를 처리하는 방식은 아래와 같이 크게 2가지로 나눌 수 있다.

- **파일데이터와 `<form>`태그의 데이터를 함께 전송하는 방식** : `<input type="file">`태그를 사용하여 파일을 선택하고, `<form>`데이터와 함께 전송
- **파일데이터와 `<form>`태그의 데이터를 구분해서 전송하는 방식** : AJAX 통신을 통해 파일을 미리 서버에 저장한 뒤 `<form>`데이터를 전송

본 예제에서는 게시판 첨부파일 업로드 기능을 AJAX방식으로 파일데이터와 `<form>`데이터를 구분해서 전송하는 방식으로 구현한다. AJAX 통신을 통해 먼저 파일을 서버에 저장하고, 게시글 입력 버튼을 클릭하면 서버에 저장된 파일명과 게시글 정보(제목, 내용, 작성자)를 `<form>`태그를 통해 DB의 테이블에 저장하는 과정을 거치게 된다. 그 외에 파일 업로드에 대한 세부적인 처리 과정은 아래와 같다.

#### # 게시글 입력 페이지의 첨부파일 업로드처리 과정
1. 게시글 제목, 내용, 작성자를 입력
2. 첨부파일이 필요한 경우 탐색기에서 drag & drop방식으로 파일을 끌어다 놓으면 AJAX방식으로 서버에 파일을 업로드 처리
3. 게시글 등록 버튼을 클릭하면 게시글 정보(제목, 내용, 작성자, 첨부파일 개수)와 첨부파일(파일명) 정보가 함께 DB의 테이블에 저장

#### # 게시글 입력/조회/수정 페이지의 첨부파일 출력처리 과정
- **입력 페이지** : 첨부파일 업로드 처리한 뒤 리턴 받은 업로드 파일명을 통해 AJAX방식으로 화면에 파일을 출력 처리
- **조회/수정 페이지** : 해당 게시글 번호를 통해 첨부파일 테이블에 저장된 파일명들을 조회하여 파일 목록을 화면에 출력 처리

#### # 게시글 입력/조회/수정 페이지의 첨부파일 삭제처리 과정
- **입력 페이지** : 삭제 버튼 클릭 시 업로드 파일명을 통해 AJAX방식으로 서버에 저장된 첨부 파일을 삭제 처리
- **조회 페이지** : 조회 페이지에서 게시글 삭제 처리를 할 경우, 첨부파일 테이블에서 해당 게시글의 첨부파일 정보들을 모두 삭제 처리한 후에 서버에 저장된 첨부 파일들도 함께 삭제 처리
- **수정 페이지** : 삭제 버튼 클릭 시 업로드 파일명을 통해 AJAX방식으로 첨부파일 테이블의 첨부파일 정보를 삭제 처리하고, 첨부파일 개수 갱신 작업을 수행, 그리고 서버에 저장된 첨부파일도 함께 삭제 처리

#### # 게시글 수정 페이지의 첨부파일 수정처리 과정
- 해당 게시글의 첨부파일 정보는 수정 처리하지 않고, 테이블에서 모두 삭제 처리한 뒤에 새로 입력하는 방식으로 처리

## 2. 파일 업로드 기능을 위한 스프링 설정
스프링에서 파일 업로드를 처리하기 위해서 아래와 같이 몇가지 설정이 필요하다.

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

## 3. 첨부파일 테이블 생성 및 게시글 도메인 클래스 변경
#### # 게시글 첨부파일 테이블 생성
게시글의 첨부정보를 저장하기 위한 `tbl_article_file`테이블을 생성해준다. 첨부파일의 이름(`file_name`)은 고유한 속성을 가지도록 처리할 것이기 때문에 기본키로 설정하였고, 참조키는 게시글 번호(`article_no`)로 설정해주었다. 그리고 게시글 테이블(`tbl_article`)에는 게시글 목록페이지에서 첨부파일의 갯수를 출력해주기 위해 칼럼을 아래와 같이 추가해주었다.
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
데이터베이스의 테이블이 변경되었기 때문에 도메인 클래스에도 추가적인 데이터가 필요하게 되었다. 그래서 `ArticleVO`클래스에 다수의 첨부파일 이름을 저장하기 위해 String배열 타입 변수 `files`를 선언하고, 첨부파일의 개수를 의미하는 `fileCnt`변수를 선언해주었다. 그리고 게시글이 입력/수정될 때 `ArticleVO`의 인스턴스가 스스로 게시글의 첨부파일 개수의 상태를 가질 수 있게 `setFiles()`메서드를 아래와 같이 작성해준다. 이렇게 되면 외부에서 `setFiles()`를 호출하여 게시글의 첨부파일 개수를 넣어주지 않아도 된다.
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

## 4. 파일 업로드 처리를 위한 유틸 클래스 작성

#### # 서버에 파일을 저장할 때 고려사항

- **동일한 파일명을 저장할 경우** : UUID를 이용하여 고유 값을 생성, 원본파일명 앞에 붙여 중복 문제 해결한다.
- **파일의 단일 저장 경로** : 단일 경로에 계속 파일을 저장하게 되면 나중에 파일을 검색하고 찾는데 속도 문제가 발생할 수 있기 때문에 파일을 날짜별 폴더를 생성해 관리한다.
- **이미지 파일인 경우 브라우저에 출력할 파일의 크기** : 이미지 파일의 용량이 클 경우, 서버에서 브라우저에 많은 양의 데이터를 전송하게 되므로 이미지 파일의 축소본, 썸네일을 생성하여 최소한의 데이터를 브라우저에 전송하도록 처리한다.

위에서 살펴본 고려사항을 바탕으로 아래와 같이 파일 업로드 처리를 위한 유틸 클래스 2개를 작성한다.

- `UploadFileUtils` : 파일 업로드/삭제/전송, 폴더생성, 파일명 중복방지 등의 기능을 처리할 메서드들을 가진 유틸 클래스

- `MediaUtils` : 파일의 타입이 이미지인지를 판별하는 메서드를 가진 유틸 클래스

#### # `UploadFileUtils` 클래스 작성
`/src/main/java/기본패키지/commons/util`패키지에 파일 업로드와 관련된 기능을 처리할 `UploadFileUtils`클래스를 아래와 같이 작성해준다.
유틸 클래스로 사용하기 때문에 기본적으로 클래스 내부의 모든 메서드는 `static`메서드로 선언하여 인스턴스 생성 없이 바로 사용이 가능하도록 하였다. 그리고 파일 업로드 처리, 삭제, 출력을 위한 httpHeader설정 메서드는 `public`메서드로 선언하고 파일 업로드 컨트롤러에서 바로 접근하여 사용이 가능하게 했다.

```java
public class UploadFileUtils {

    private static final Logger logger = LoggerFactory.getLogger(UploadFileUtils.class);

    // 파일 업로드 처리
    public static String uploadFile(MultipartFile file, HttpServletRequest request) throws Exception {

        String originalFileName = file.getOriginalFilename(); // 파일명
        byte[] fileData = file.getBytes();  // 파일 데이터

        // 1. 파일명 중복 방지 처리
        String uuidFileName = getUuidFileName(originalFileName);

        // 2. 파일 업로드 경로 설정
        String rootPath = getRootPath(originalFileName, request); // 기본경로 추출(이미지 or 일반파일)
        String datePath = getDatePath(rootPath); // 날짜 경로 추출, 날짜 폴더 생성

        // 3. 서버에 파일 저장
        File target = new File(rootPath + datePath, uuidFileName); // 파일 객체 생성
        FileCopyUtils.copy(fileData, target); // 파일 객체에 파일 데이터 복사

        // 4. 이미지 파일인 경우 썸네일이미지 생성
        if (MediaUtils.getMediaType(originalFileName) != null) {
            uuidFileName = makeThumbnail(rootPath, datePath, uuidFileName);
        }

        // 5. 파일 저장 경로 치환
        return replaceSavedFilePath(datePath, uuidFileName);
    }

    // 파일 삭제 처리
    public static void deleteFile(String fileName, HttpServletRequest request) {

        String rootPath = getRootPath(fileName, request); // 기본경로 추출(이미지 or 일반파일)

        // 1. 원본 이미지 파일 삭제
        MediaType mediaType = MediaUtils.getMediaType(fileName);
        if (mediaType != null) {
            String originalImg = fileName.substring(0, 12) + fileName.substring(14);
            new File(rootPath + originalImg.replace('/', File.separatorChar)).delete();
        }

        // 2. 파일 삭제(썸네일이미지 or 일반파일)
        new File(rootPath + fileName.replace('/', File.separatorChar)).delete();
    }

    // 파일 출력을 위한 HttpHeader 설정
    public static HttpHeaders getHttpHeaders(String fileName) throws Exception {

        MediaType mediaType = MediaUtils.getMediaType(fileName); // 파일타입 확인
        HttpHeaders httpHeaders = new HttpHeaders();

        // 이미지 파일 O
        if (mediaType != null) {
            httpHeaders.setContentType(mediaType);
            return httpHeaders;
        }

        // 이미지 파일 X
        fileName = fileName.substring(fileName.indexOf("_") + 1); // UUID 제거
        httpHeaders.setContentType(MediaType.APPLICATION_OCTET_STREAM); // 다운로드 MIME 타입 설정
        // 파일명 한글 인코딩처리
        httpHeaders.add("Content-Disposition",
                        "attachment; filename=\"" + new String(fileName.getBytes("UTF-8"),
                                "ISO-8859-1")+"\"");

        return httpHeaders;
    }

    // 기본 경로 추출
    public static String getRootPath(String fileName, HttpServletRequest request) {

        String rootPath = "/resources/upload";
        MediaType mediaType = MediaUtils.getMediaType(fileName); // 파일타입 확인
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
    private static String replaceSavedFilePath(String datePath, String fileName) {
        String savedFilePath = datePath + File.separator + fileName;
        return savedFilePath.replace(File.separatorChar, '/');
    }

    // 파일명 중복방지 처리
    private static String getUuidFileName(String originalFileName) {
        return UUID.randomUUID().toString() + "_" + originalFileName;
    }

    // 썸네일 이미지 생성
    private static String makeThumbnail(String uploadRootPath, String datePath, String fileName) throws Exception {

        // 원본이미지를 메모리상에 로딩
        BufferedImage originalImg = ImageIO.read(new File(uploadRootPath + datePath, fileName));
        // 원본이미지를 축소
        BufferedImage thumbnailImg = Scalr.resize(originalImg, Scalr.Method.AUTOMATIC, Scalr.Mode.FIT_TO_HEIGHT, 100);
        // 썸네일 파일명
        String thumbnailImgName = "s_" + fileName;
        // 썸네일 업로드 경로
        String fullPath = uploadRootPath + datePath + File.separator + thumbnailImgName;
        // 썸네일 파일 객체생성
        File newFile = new File(fullPath);
        // 썸네일 파일 확장자 추출
        String formatName = MediaUtils.getFormatName(fileName);
        // 썸네일 파일 저장
        ImageIO.write(thumbnailImg, formatName, newFile);

        return thumbnailImgName;
    }

}
```

#### # `MediaUtils` 클래스 작성
`UploadFileUtils`클래스와 마찬가지로 `/src/main/java/기본패키지/commons/util`패키지에 이미지 타입 여부를 판별해주는 메서드를 가진 `MediaUtils`클래스를 아래와 같이 작성했다.
```java
class MediaUtils {

    private static Map<String, MediaType> mediaTypeMap;

    // 클래스 초기화 블럭
    static {
        mediaTypeMap = new HashMap<>();
        mediaTypeMap.put("JPG", MediaType.IMAGE_JPEG);
        mediaTypeMap.put("GIF", MediaType.IMAGE_GIF);
        mediaTypeMap.put("PNG", MediaType.IMAGE_PNG);
    }

    // 파일 타입
    static MediaType getMediaType(String fileName) {
        String formatName = getFormatName(fileName);
        return mediaTypeMap.get(formatName);
    }

    // 파일 확장자 추출
    static String getFormatName(String fileName) {
        return fileName.substring(fileName.lastIndexOf(".") + 1).toUpperCase();
    }

}
```

## 5. 게시글 작성 페이지의 파일 첨부 기능 구현

#### # 게시글 첨부파일 영속계층 작성 : `ArticleFileDAO` / `ArticleFileDAOImpl`
`/src/main/기본패키지/upload/Persistence`패키지에 첨부파일의 정보를 테이블에 저장하기 위해 `ArticleFileDAO`인터페이스와 `ArticleFileDAOImpl`클래스를 생성하고 코드를 아래와 같이 작성해준다.
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

#### # 게시글 서비스 계층 수정
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

##### 첨부파일 업로드 처리 매핑 메서드
게시글 첨부파일 컨트롤러는 앞서 말한 것처럼 게시글 입력처리가 되기 전에 클라이언트로부터 AJAX통신을 통해 첨부파일을 미리 서버에 저장하는 역할을 수행하게 된다.

```java
@RestController
@RequestMapping("/article/file")
public class ArticleFileController {

    private final ArticleFileService articleFileService;

    @Inject
    public ArticleFileController(ArticleFileService articleFileService) {
        this.articleFileService = articleFileService;
    }

    // 게시글 파일 업로드
    @RequestMapping(value = "/upload", method = RequestMethod.POST, produces = "text/plain;charset=UTF-8")
    public ResponseEntity<String> uploadFile(MultipartFile file, HttpServletRequest request) {
        ResponseEntity<String> entity = null;
        try {
            String savedFilePath = UploadFileUtils.uploadFile(file, request);
            entity = new ResponseEntity<>(savedFilePath, HttpStatus.CREATED);
        } catch (Exception e) {
            e.printStackTrace();
            entity = new ResponseEntity<>(HttpStatus.BAD_REQUEST);
        }
        return entity;
    }

}
```

파일 업로드 처리는 다음과 같은 과정을 거치게 된다.
1. 클라이언트로부터 전달받은 `file`과 `request`를 `UploadFileUtils`클래스의 `uploadFile()`메서드의 매개변수로 전달한다.
2. `uploadFile()`메서드는 파일업로드 처리를 수행하고, "/년/월/일/UUID_파일명"의 문자열을 리턴한다.
3. `uploadFile()`메서드로부터 리턴 받은 문자열을 호출한 클라이언트 화면으로 리턴한다.

위의 컨트롤러 코드에서 주의할 점은 `@RequestMapping`애너테이션의 `produces`속성을 `"text/plain;charset=UTF-8"`로 지정한 것인데 클라이언트로 한글을 정상적으로 전송하기 위한 설정이므로 반드시 작성해야 한다.

##### 첨부파일 출력 매핑 메서드
첨부파일이 서버에 업로드가 완료되고 나면, 화면에 업로드한 첨부파일을 확인할 수 있도록 아래와 같이 컨트롤러에 첨부파일 출력 메서드를 작성해준다.
```java
// 게시글 첨부파일 출력
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
첨부파일 출력 처리는 다음과 같은 과정을 거치게 된다.

1. 클라이언트로부터 전달받은 첨부파일명(`fileName`)과 `request`를 `UploadFileUtils`클래스의 `getHttpHeaders()`메서드의 매개변수로 전달한다.
2. `getHttpHeaders()`메서드는 첨부파일명(`fileName`)을 통해 파일타입을 판별하여 적절한 MINE타입을 지정하고, 클라이언트에게 전송할 `HttpHeaders`객체를 리턴한다.
3. `getHttpHeaders()`메서드로부터 리턴받은 `HttpHeaders`객체와 첨부파일 데이터를 호출한 클라이언트 화면으로 리턴한다.

##### 첨부파일 삭제 처리 매핑 메서드
첨부파일을 삭제할 수 있도록 아래와 같이 컨트롤러에 첨부파일 삭제 메서드를 작성해준다.
```java
// 게시글 파일 삭제 : 게시글 작성 페이지
@RequestMapping(value = "/delete", method = RequestMethod.POST)
public ResponseEntity<String> deleteFile(String fileName, HttpServletRequest request) {
    ResponseEntity<String> entity = null;

    try {
        UploadFileUtils.deleteFile(fileName, request);
        entity = new ResponseEntity<>("DELETED", HttpStatus.OK);
    } catch (Exception e) {
        e.printStackTrace();
        entity = new ResponseEntity<>(e.getMessage(), HttpStatus.BAD_REQUEST);
    }

    return entity;
}
```
첨부파일 삭제 처리는 다음과 같은 과정을 거치게 된다.
1. 클라이언트로부터 전달 받은 첨부파일명(`fileName`)과 `request`를 `UploadFileUtils`클래스의 `deleteFile()`메서드의 매개변수로 전달한다.
2. `deleteFile()`메서드는 첨부파일명을 `MediaUtils`클래스의 `getMediaType()`메서드의 매개변수로 전달하고, 이미지 타입 여부를 판별한다.
3. 이미지 파일인 경우 원본 이미지 파일을 삭제 처리하고, 썸네일 파일도 삭제 처리한다.
4. 일반 파일인 경우 파일을 삭제 처리를 수행한다.

#### # 게시글 작성 페이지 수정 : `write.jsp`
게시글 작성 페이지는 첨부파일 영역을 추가하고, 첨부파일 업로드/출력/삭제를 위한 AJAX통신이 이루어지도록 JS코드를 작성해준다.

##### 파일 업로드 영역 CSS
```css
.fileDrop {
    width: 100%;
    height: 200px;
    border: 2px dotted #0b58a2;
}
```

##### 파일 업로드 영역 HTML
```xml
<div class="col-lg-12">
    <form role="form" id="writeForm" method="post" action="${path}/article/paging/search/write">
        <div class="box box-primary">
            <div class="box-header with-border">
                <h3 class="box-title">게시글 작성</h3>
            </div>
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
                <%--첨부파일 영역 추가--%>
                <div class="form-group">
                    <div class="fileDrop">
                        <br/>
                        <br/>
                        <br/>
                        <br/>
                        <p class="text-center"><i class="fa fa-paperclip"></i> 첨부파일을 드래그해주세요.</p>
                    </div>
                </div>
                <%--첨부파일 영역 추가--%>
            </div>
            <div class="box-footer">
                <ul class="mailbox-attachments clearfix uploadedFileList"></ul>
            </div>
            <div class="box-footer">
                <button type="button" class="btn btn-primary listBtn"><i class="fa fa-list"></i> 목록</button>
                <div class="pull-right">
                    <button type="reset" class="btn btn-warning"><i class="fa fa-reply"></i> 초기화</button>
                    <button type="submit" class="btn btn-success"><i class="fa fa-save"></i> 저장</button>
                </div>
            </div>
        </div>
    </form>
</div>
```

##### 첨부파일 출력을 위한 Handlebars템플릿
댓글기능을 구현했을 때와 마찬가지로 Handlebars템플릿을 이용하여 HTML코드를 동적으로 생성하도록 아래와 같이 코드를 작성해준다.
```xml
<script id="fileTemplate" type="text/x-handlebars-template">
    <li>
        <span class="mailbox-attachment-icon has-img">
            <img src="{{imgSrc}}" alt="Attachment">
        </span>
        <div class="mailbox-attachment-info">
            <a href="{{originalFileUrl}}" class="mailbox-attachment-name">
                <i class="fa fa-paperclip"></i> {{originalFileName}}
            </a>
            <a href="{{fullName}}" class="btn btn-default btn-xs pull-right delBtn">
                <i class="fa fa-fw fa-remove"></i>
            </a>
        </div>
    </li>
</script>
```

##### 첨부파일 업로드/출력/삭제를 위한 JS파일 작성
첨부파일 업로드/출력/삭제처리를 위한 JS코드는 `/resources/dist/js`에 `article_file_upload.js`파일을 생성해서 따로 작성해주고, 아래와 같이 게시글 작성페이지에 포함시켜준다. 첨부파일 기능을 처리하는 JS코드의 내용이 많고, 게시글 입력/조회/수정 페이지에 공통으로 들어가는 코드의 중복을 방지하기 위해서이다. 그리고 일반 파일의 경우 썸네일 이미지가 없기 때문에 파일 아이콘 이미지를 출력해야 한다. 그래서 `/resources/dist/upload/files/`경로에 `file-icon.png`파일을 넣어준다.
```xml
<script type="text/javascript" src="/resources/dist/js/article_file_upload.js"></script>
```
```js
// Handlebars 파일템플릿 컴파일
var fileTemplate = Handlebars.compile($("#fileTemplate").html());

// 첨부파일 drag & drop 영역 선택자
var fileDropDiv = $(".fileDrop");

// 전체 페이지 첨부파일 drag & drop 기본 이벤트 방지
$(".content-wrapper").on("dragenter dragover drop", function (event) {
    event.preventDefault();
});

// 첨부파일 영역 drag & drop 기본 이벤트 방지
fileDropDiv.on("dragenter dragover", function (event) {
    event.preventDefault();
});

// 첨부파일 drag & drop 이벤트 처리 : 파일업로드 처리 -> 파일 출력
fileDropDiv.on("drop", function (event) {
    event.preventDefault();
    // drop 이벤트 발생시 전달된 파일 데이터
    var files = event.originalEvent.dataTransfer.files;
    // 단일 파일 데이터만을 처리하기 때문 첫번째 파일만 저장
    var file = files[0];
    // formData 객체 생성, 파일데이터 저장
    var formData = new FormData();
    formData.append("file", file);
    // 파일 업로드 AJAX 통신 메서드 호출
    uploadFile(formData);
});

// 파일 업로드 AJAX 통신
function uploadFile(formData) {
    $.ajax({
        url: "/article/file/upload",
        data: formData,
        dataType: "text",
        // processData : 데이터를 일반적인 query string으로 변환처리할 것인지 결정
        // 기본값은 true, application/x-www-form-urlencoded 타입
        // 자동변환 처리하지 않기 위해 false
        processData: false,
        // contentType : 기본값은 true, application/x-www-form-urlencoded 타입
        // multipart/form-data 방식으로 전송하기 위해 false
        contentType: false,
        type: "POST",
        success: function (data) {
            printFiles(data); // 첨부파일 출력 메서드 호출
            $(".noAttach").remove();
        }
    });
}

// 첨부파일 출력
function printFiles(data) {
    // 파일 정보 처리
    var fileInfo = getFileInfo(data);
    // Handlebars 파일 템플릿에 파일 정보들을 바인딩하고 HTML 생성
    var html = fileTemplate(fileInfo);
    // Handlebars 파일 템플릿 컴파일을 통해 생성된 HTML을 DOM에 주입
    $(".uploadedFileList").append(html);
    // 이미지 파일인 경우 파일 템플릿에 lightbox 속성 추가
    if (fileInfo.fullName.substr(12, 2) === "s_") {
        // 마지막에 추가된 첨부파일 템플릿 선택자
        var that = $(".uploadedFileList li").last();
        // lightbox 속성 추가
        that.find(".mailbox-attachment-name").attr("data-lightbox", "uploadImages");
        // 파일 아이콘에서 이미지 아이콘으로 변경
        that.find(".fa-paperclip").attr("class", "fa fa-camera");
    }
}

// 게시글 입력/수정 submit 처리시에 첨부파일 정보도 함께 처리
function filesSubmit(that) {
    var str = "";
    $(".uploadedFileList .delBtn").each(function (index) {
        str += "<input type='hidden' name='files[" + index + "]' value='" + $(this).attr("href") + "'>"
    });
    that.append(str);
    that.get(0).submit();
}

// 파일 삭제(입력페이지) : 첨부파일만 삭제처리
function deleteFileWrtPage(that) {
    var url = "/article/file/delete";
    deleteFile(url, that);
}

// 파일 삭제 AJAX 통신
function deleteFile(url, that) {
    $.ajax({
        url: url,
        type: "post",
        data: {fileName: that.attr("href")},
        dataType: "text",
        success: function (result) {
            if (result === "DELETED") {
                alert("삭제되었습니다.");
                that.parents("li").remove();
            }
        }
    });
}

// 파일 정보 처리
function getFileInfo(fullName) {

    var originalFileName;   // 화면에 출력할 파일명
    var imgSrc;             // 썸네일 or 파일아이콘 이미지 파일 출력 요청 URL
    var originalFileUrl;    // 원본파일 요청 URL
    var uuidFileName;       // 날짜경로를 제외한 나머지 파일명 (UUID_파일명.확장자)

    // 이미지 파일이면
    if (checkImageType(fullName)) {
        imgSrc = "/article/file/display?fileName=" + fullName; // 썸네일 이미지 링크
        uuidFileName = fullName.substr(14);
        var originalImg = fullName.substr(0, 12) + fullName.substr(14);
        // 원본 이미지 요청 링크
        originalFileUrl = "/article/file/display?fileName=" + originalImg;
    } else {
        imgSrc = "/resources/upload/files/file-icon.png"; // 파일 아이콘 이미지 링크
        uuidFileName = fullName.substr(12);
        // 파일 다운로드 요청 링크
        originalFileUrl = "/article/file/display?fileName=" + fullName;
    }
    originalFileName = uuidFileName.substr(uuidFileName.indexOf("_") + 1);

    return {originalFileName: originalFileName, imgSrc: imgSrc, originalFileUrl: originalFileUrl, fullName: fullName};
}

// 이미지 파일 유무 확인
function checkImageType(fullName) {
    var pattern = /jpg$|gif$|png$|jpge$/i;
    return fullName.match(pattern);
}

```

##### 원본 이미지 파일 출력을 위한 ligthbox 라이브러리 적용
첨부파일이 이미지 타입인 경우 화면에 출력할 때 썸네일을 출력하고, 해당 썸네일을 클릭하면 원본 이미지 파일을 출력해주기 위해서 `head.jsp`와 `plugin_js.jsp`에 `lightbox`플러그인 css와 js파일을 아래와 같이 추가해준다.

> lightbox플러그인의 사용법은 이 곳(http://lokeshdhakar.com/projects/lightbox2/)을 참조하면 된다.

**`head.jsp`**
```xml
<%--lightbox css--%>
<link rel="stylesheet" href="/bower_components/lightbox/css/lightbox.css">
```
**`plugin_js.jsp`**
```xml
<%--lightbox js--%>
<script src="/bower_components/lightbox/js/lightbox.js"></script>
```
그리고 `article_file_upload.js`의 첨부파일 출력 메서드 코드를 보면 Handlebars 파일 템플릿을 통해 출력처리가 완료되고 나서 파일명이 썸네일인 경우에는 lightbox속성이 추가되도록 처리한 것을 확인 할 수 있다.
```js
// 첨부파일 출력
function printFiles(data) {
    // 파일 정보 처리
    var fileInfo = getFileInfo(data);
    // Handlebars 파일 템플릿에 파일 정보들을 바인딩하고 HTML 생성
    var html = fileTemplate(fileInfo);
    // Handlebars 파일 템플릿 컴파일을 통해 생성된 HTML을 DOM에 주입
    $(".uploadedFileList").append(html);
    // 이미지 파일인 경우 파일 템플릿에 lightbox 속성 추가
    if (fileInfo.fullName.substr(12, 2) === "s_") {
        // 마지막에 추가된 첨부파일 템플릿 선택자
        var that = $(".uploadedFileList li").last();
        // lightbox 속성 추가
        that.find(".mailbox-attachment-name").attr("data-lightbox", "uploadImages");
        // 파일 아이콘에서 이미지 아이콘으로 변경
        that.find(".fa-paperclip").attr("class", "fa fa-camera");
    }
}
```

##### 게시글 입력 페이지(`write.jsp`)의 첨부파일 이벤트 처리 JS 코드
게시글 저장 버튼을 클릭하면 `article_file_upload.js`의 `filesSubmit()`메서드를 호출하여 `<input type="hidden"/>`태그를 생성하고 `value`에 업로드 처리한 첨부 파일명을 추가 시켜 최종적으로 `<form>`태그를 `submit`처리하게 된다.
```js
// 게시글 저장 버튼 클릭 이벤트 처리
$("#writeForm").submit(function (event) {
    event.preventDefault();
    var that = $(this);
    filesSubmit(that);
});
```
첨부 파일의 삭제 버튼을 클릭하면 `article_file_upload.js`의 `deleteFileWrtPage()`메서드를 호출하여 파일 삭제처리를 수행하게 된다.
```js
// 파일 삭제 버튼 클릭 이벤트
$(document).on("click", ".delBtn", function (event) {
    event.preventDefault();
    var that = $(this);
    deleteFileWrtPage(that);
});
```
#### # 구현 확인
##### 게시글 입력페이지의 첨부파일 업로드 영역
![file](https://github.com/walbatrossw/develop-notes/blob/master/reding-notes/%EC%BD%94%EB%93%9C%EB%A1%9C_%EB%B0%B0%EC%9A%B0%EB%8A%94_%EC%8A%A4%ED%94%84%EB%A7%81_%EC%9B%B9%ED%94%84%EB%A1%9C%EC%A0%9D%ED%8A%B8/photo/2018-04-06%2013-48-29.png?raw=true)

##### 게시글 입력페이지의 첨부파일 업로드 처리 및 출력처리 확인
![file2](https://github.com/walbatrossw/develop-notes/blob/master/reding-notes/%EC%BD%94%EB%93%9C%EB%A1%9C_%EB%B0%B0%EC%9A%B0%EB%8A%94_%EC%8A%A4%ED%94%84%EB%A7%81_%EC%9B%B9%ED%94%84%EB%A1%9C%EC%A0%9D%ED%8A%B8/photo/2018-04-06%2014-02-33.png?raw=true)
- 파일 탐색기에서 첨부파일을 선택해 업로드 영역에 드래그
- 첨부파일 업로드 처리 후 파일이 정상적으로 서버에 저장 확인, 이미지 파일인 경우 썸네일이 생성되었는지 확
- 한글 파일명 정상 출력 확인
- 이미지 파일인 경우 썸네일 이미지 생성/출력 처리 확인
- 일반 파일인 경우 파일 아이콘 이미지 출력 확인

##### 게시글 입력페이지의 첨부파일 삭제 처리 확인
![file3](https://github.com/walbatrossw/develop-notes/blob/master/reding-notes/%EC%BD%94%EB%93%9C%EB%A1%9C_%EB%B0%B0%EC%9A%B0%EB%8A%94_%EC%8A%A4%ED%94%84%EB%A7%81_%EC%9B%B9%ED%94%84%EB%A1%9C%EC%A0%9D%ED%8A%B8/photo/2018-04-06%2014-10-48.png?raw=true)
- 첨부파일 삭제버튼을 클릭하면 파일이 정상적으로 서버에서 삭제되었는지 확인
- 첨부파일이 삭제처리가 되고 화면에서 해당파일이 사라졌는지 확인

##### 게시글 입력페이지의 lightbox 라이브러리 적용 확인
![file4](https://github.com/walbatrossw/develop-notes/blob/master/reding-notes/%EC%BD%94%EB%93%9C%EB%A1%9C_%EB%B0%B0%EC%9A%B0%EB%8A%94_%EC%8A%A4%ED%94%84%EB%A7%81_%EC%9B%B9%ED%94%84%EB%A1%9C%EC%A0%9D%ED%8A%B8/photo/2018-04-06%2014-13-12.png?raw=true)
- 첨부파일의 이미지인 경우 파일명 클릭시 원본 이미지가 팝업창으로 뜨는지 확인

##### 게시글 입력페이지에서 일반파일인 경우 파일명 클릭시 다운로드 창 팝업 확인
![file5](https://github.com/walbatrossw/develop-notes/blob/master/reding-notes/%EC%BD%94%EB%93%9C%EB%A1%9C_%EB%B0%B0%EC%9A%B0%EB%8A%94_%EC%8A%A4%ED%94%84%EB%A7%81_%EC%9B%B9%ED%94%84%EB%A1%9C%EC%A0%9D%ED%8A%B8/photo/2018-04-06%2014-14-35.png?raw=true)

##### 게시글 입력처리 완료 후에 게시글 목록 페이지 이동, 파일 개수 확인, 테이블에 파일정보가 등록되었는지 확인
![file6](https://github.com/walbatrossw/develop-notes/blob/master/reding-notes/%EC%BD%94%EB%93%9C%EB%A1%9C_%EB%B0%B0%EC%9A%B0%EB%8A%94_%EC%8A%A4%ED%94%84%EB%A7%81_%EC%9B%B9%ED%94%84%EB%A1%9C%EC%A0%9D%ED%8A%B8/photo/2018-04-06%2014-19-13.png?raw=true)

## 6. 게시글 조회 페이지의 첨부파일 목록 구현/ 게시글 삭제 처리시 첨부파일삭제 처리 구현
게시글 조회페이지에서는 첨부파일 기능이 추가됨에 따라 추가적으로 구현해야할 것들은 다음과 같다.
- 첨부파일 목록 출력
- 게시글 삭제 처리시 서버에 저장된 첨부파일 삭제, 테이블에 저장된 첨부파일 정보 삭제

**첨부파일 목록 출력**은 아래와 같은 과정을 거치게 된다.
1. AJAX 방식으로 현재 조회하고 있는 게시글의 첨부파일 목록을 가져온다.
2. 테이블로부터 가져온 첨부파일 목록에서 첨부파일명을 통해 AJAX 방식으로 파일을 출력처리 한다.

**게시글 삭제시 첨부파일의 삭제처리**는 아래와 같은 과정을 거치게 된다.
1. AJAX 방식으로 해당 게시글의 첨부파일을 서버에서 삭제처리를 먼저 수행한다.
2. 해당 게시글의 첨부파일 정보를 테이블에서 먼저 삭제 처리를 수행하고 난 뒤, 게시글을 삭제 처리한다.

#### # 게시글 첨부파일 영속 계층 : `ArticleFileDAO` / `ArticleFileDAOImpl`
아래와 같이 `ArticleFileDAO`인터페이스에 첨부파일 목록을 가져오는 추상 메서드를 추가하고, `ArticleFileDAOImpl`클래스에 메서드를 구현해준다.
```java
// 첨부 파일 목록
List<String> getArticleFiles(Integer articleNo) throws Exception;

// 게시글의 첨부 파일 전체 삭제
void deleteFiles(Integer articleNo) throws Exception;
```
```java
// 첨부 파일 목록
@Override
public List<String> getArticleFiles(Integer articleNo) throws Exception {
    return sqlSession.selectList(NAMESPACE + ".getArticleFiles", articleNo);
}

// 게시글의 첨부 파일 전체 삭제
@Override
public void deleteFiles(Integer articleNo) throws Exception {
    sqlSession.delete(NAMESPACE + ".deleteFiles", articleNo);
}
```

#### # 게시글 첨부파일 SQL Mapper : `articleFileMapper.xml`
특정 게시글의 첨부파일 목록을 가져오는 SQL과 해당 게시글의 첨부파일들을 삭제처리하는 SQL을 아래와 같이 작성한다.
```sql
<select id="getArticleFiles" resultType="string">
    SELECT file_name
    FROM tbl_article_file
    WHERE article_no = #{articleNo}
    ORDER BY reg_date
</select>

<delete id="deleteFiles">
    DELETE FROM tbl_article_file
    WHERE article_no = #{articleNo}
</delete>
```

#### # 게시글 첨부파일 비지니스 계층 : `ArticleFileService` / `ArticleServiceImpl`
아래와 같이 `ArticleFileService`인터페이스를 생성하고, 첨부파일 목록을 가져오는 추상메서드를 작성해준다. 그리고 `ArticleFileServiceImpl`클래스에 첨부파일 목록 메서드를 구현해준다.
```java
public interface ArticleFileService {

    // 첨부파일 목록
    List<String> getArticleFiles(Integer articleNo) throws Exception;

}
```

```java
@Service
public class ArticleFileServiceImpl implements ArticleFileService {

    private final ArticleFileDAO articleFileDAO;

    @Inject
    public ArticleFileServiceImpl(ArticleFileDAO articleFileDAO) {
        this.articleFileDAO = articleFileDAO;
    }

    // 첨부파일 목록
    @Override
    public List<String> getArticleFiles(Integer articleNo) throws Exception {
        return articleFileDAO.getArticleFiles(articleNo);
    }

}
```

#### # 게시글 비지니스 계층 수정 : `ArticleServiceImpl`
게시글이 삭제 처리되기 전에 해당 게시글의 첨부파일들을 테이블에서 먼저 삭제처리 하도록 아래와 같이 코드를 작성하고, 트랜잭션처리를 위해 `@Transactional`애너테이션을 붙여준다.
```java
private final ArticleDAO articleDAO;

private final ArticleFileDAO articleFileDAO;

@Inject
public ArticleServiceImpl(ArticleDAO articleDAO, ArticleFileDAO articleFileDAO) {
    this.articleDAO = articleDAO;
    this.articleFileDAO = articleFileDAO;
}

// 다른 메서드 생략 ...

// 게시글 삭제 처리
@Transactional
@Override
public void delete(Integer articleNo) throws Exception {
    articleFileDAO.deleteFiles(articleNo);
    articleDAO.delete(articleNo);
}
```

#### # 게시글 첨부파일 컨트롤러 : `ArticleFileController`
아래와 같이 컨트롤러에 첨부파일 목록 매핑 메서드를 아래와 같이 작성해준다.
```java
// 게시글 첨부 파일 목록
@RequestMapping(value = "/list/{articleNo}", method = RequestMethod.GET)
public ResponseEntity<List<String>> getFiles(@PathVariable("articleNo") Integer articleNo) {
    ResponseEntity<List<String>> entity = null;
    try {
        List<String> fileList = articleFileService.getArticleFiles(articleNo);
        entity = new ResponseEntity<>(fileList, HttpStatus.OK);
    } catch (Exception e) {
        e.printStackTrace();
        entity = new ResponseEntity<>(HttpStatus.BAD_REQUEST);
    }
    return entity;
}
```
그리고 해당 게시글의 첨부파일 전체를 삭제하는 매핑 메서드를 아래와 같이 작성해준다.
```java
// 게시글 파일 전체 삭제
@RequestMapping(value = "/deleteAll", method = RequestMethod.POST)
public ResponseEntity<String> deleteAllFiles(@RequestParam("files[]") String[] files, HttpServletRequest request) {

    if (files == null || files.length == 0)
        return new ResponseEntity<>("DELETED", HttpStatus.OK);

    ResponseEntity<String> entity = null;

    try {
        for (String fileName : files)
            UploadFileUtils.deleteFile(fileName, request);
        entity = new ResponseEntity<>("DELETED", HttpStatus.OK);
    } catch (Exception e) {
        e.printStackTrace();
        entity = new ResponseEntity<>(e.getMessage(), HttpStatus.BAD_REQUEST);
    }

    return entity;
}
```

#### # 게시글 목록 페이지 수정 : `read.jsp`

##### 게시글 첨부파일 출력 영역 HTML코드
게시글의 첨부파일 목록이 출력될 영역을 아래와 같이 추가해준다.
```xml
<%--업로드 파일 정보 영역--%>
<div class="box-footer uploadFiles">
    <ul class="mailbox-attachments clearfix uploadedFileList"></ul>
</div>
<%--업로드 파일 정보 영역--%>
```
##### 게시글 첨부파일 Handlebars 파일 템플릿
게시글 첨부파일을 출력하기 위해 Handlebars 파일 템플릿 코드를 아래와 추가적으로 작성해준다. 기존의 첨부파일 Handlebars와 다른점은 `<li>`태그에 `data-src`속성을 추가하였는데 이 속성을 통해 게시글 삭제할 때 함께 수행할 첨부파일 삭제처리를 위한 정보로 쓰이기 위해서이다.
```xml
<script id="fileTemplate" type="text/x-handlebars-template">
    <li data-src="{{fullName}}">
        <span class="mailbox-attachment-icon has-img">
            <img src="{{imgSrc}}" alt="Attachment">
        </span>
        <div class="mailbox-attachment-info">
            <a href="{{originalFileUrl}}" class="mailbox-attachment-name">
                <i class="fa fa-paperclip"></i> {{originalFileName}}
            </a>
        </div>
    </li>
</script>
```

##### 게시글 첨부파일 JS파일 수정 : `article_file_upload.js`
`article_file_upload.js`파일에 아래와 같이 파일 목록을 가져오는 코드를 작성해준다. 만약 첨부파일이 없을 경우 첨부파일이 없다는 메시지를 출력하게 해주었다.
```js
// 파일 목록 : 게시글 조회, 수정페이지
function getFiles(articleNo) {
    $.getJSON("/article/file/list/" + articleNo, function (list) {
        if (list.length === 0) {
            $(".uploadedFileList").html("<span class='noAttach'>첨부파일이 없습니다.</span>");
        }
        $(list).each(function () {
            printFiles(this);
        })
    });
}
```

##### 게시글 목록 페이지 JS 영역
게시글 조회페이지가 로드되면 첨부파일 목록을 출력하기 위해 `getFiles()`메서드를 호출해준다.
```js
// 현재 게시글 번호
var articleNo = "${article.articleNo}";  
// 첨부파일 목록
getFiles(articleNo);
```
게시글 삭제 버튼 클릭시 서버에 저장된 첨부파일 삭제처리하는 메서드를 아래와 같이 작성해준다. 그리고 기존의 코드에서 추가적으로 변경된 내용은 다음과 같다.
- 댓글이 달린 게시글은 삭제처리 되지 않도록 조건문을 추가
- 해당 게시글의 첨부파일들을 삭제처리하기 위해서 배열에 각각의 첨부파일명을 담아 AJAX 삭제 요청

```js
// 게시글 삭제 클릭 이벤트
$(".delBtn").on("click", function () {

    // 댓글이 달린 게시글 삭제처리 방지
    var replyCnt = $(".replyDiv").length;
    if (replyCnt > 0) {
        alert("댓글이 달린 게시글은 삭제할수 없습니다.");
        return;
    }

    // 첨부파일명들을 배열에 저장
    var arr = [];
    $(".uploadedFileList li").each(function () {
        arr.push($(this).attr("data-src"));
    });

    // 첨부파일 삭제요청
    if (arr.length > 0) {
        $.post("/article/file/deleteAll", {files: arr}, function () {

        });
    }

    // 삭제처리
    formObj.attr("action", "/article/paging/search/remove");
    formObj.submit();
});
```


#### # 구현확인
##### 게시글 조회 화면에서 정상적으로 게시글 첨부파일 목록이 나오는 것을 확인
![file6](https://github.com/walbatrossw/develop-notes/blob/master/reding-notes/%EC%BD%94%EB%93%9C%EB%A1%9C_%EB%B0%B0%EC%9A%B0%EB%8A%94_%EC%8A%A4%ED%94%84%EB%A7%81_%EC%9B%B9%ED%94%84%EB%A1%9C%EC%A0%9D%ED%8A%B8/photo/2018-04-06%2014-23-07.png?raw=true)

##### 게시글 처리완료 후에 서버에 저장된 첨부파일 삭제 및 첨부파일 테이블에 파일정보 삭제 확인
삭제처리 전
![file7](https://github.com/walbatrossw/develop-notes/blob/master/reding-notes/%EC%BD%94%EB%93%9C%EB%A1%9C_%EB%B0%B0%EC%9A%B0%EB%8A%94_%EC%8A%A4%ED%94%84%EB%A7%81_%EC%9B%B9%ED%94%84%EB%A1%9C%EC%A0%9D%ED%8A%B8/photo/2018-04-06%2014-25-16.png?raw=true)

삭제처리 후
![file8](https://github.com/walbatrossw/develop-notes/blob/master/reding-notes/%EC%BD%94%EB%93%9C%EB%A1%9C_%EB%B0%B0%EC%9A%B0%EB%8A%94_%EC%8A%A4%ED%94%84%EB%A7%81_%EC%9B%B9%ED%94%84%EB%A1%9C%EC%A0%9D%ED%8A%B8/photo/2018-04-06%2014-27-23.png?raw=true)

## 7. 게시글 수정 페이지의 첨부파일 수정 구현
첨부파일을 정상적으로 수정하기 위해서는 다음과 같은 조건이 필요하다.

- 첨부파일 삭제버튼을 클릭하면 서버에 저장된 첨부파일이 삭제되고, 테이블의 파일정보도 함께 삭제되야한다.
- 수정 버튼을 클릭하면 첨부파일의 기존의 파일정보는 테이블에서 삭제되고, 새로 테이블에 입력된다.
- 첨부파일의 개수가 이전과 다를때는 갱신을 해야한다.

위의 조건을 고려하여 게시글 수정페이지에서 첨부파일 수정을 구현해보자.

#### # 게시글 첨부파일 영속계층 : `ArticleFileDAO` / `ArticleFileDAOImpl`
`ArticleFileDAO`인터페이스에 첨부파일 삭제/수정/개수 갱신 추상 메서드를 선언하고, `ArticleFileDAOImpl`클래스에서 구현해준다.
```java
// 첨부파일 삭제
void deleteFile(String fileName) throws Exception;

// 첨부파일 수정
void replaceFile(String fileName, Integer articleNo) throws Exception;

// 첨부파일 개수 갱신
void updateFileCnt(Integer articleNo) throws Exception;
```

```java
@Override
public void deleteFile(String fileName) throws Exception {
    sqlSession.delete(NAMESPACE + ".deleteFile", fileName);
}

@Override
public void replaceFile(String fileName, Integer articleNo) throws Exception {
    Map<String, Object> paramMap = new HashMap<>();
    paramMap.put("fileName", fileName);
    paramMap.put("articleNo", articleNo);
    sqlSession.insert(NAMESPACE + ".replaceFile", paramMap);
}

@Override
public void updateFileCnt(Integer articleNo) throws Exception {
    sqlSession.update(NAMESPACE + ".updateFileCnt", articleNo);
}
```

#### # 게시글 첨부파일 SQL Mapper : `articleFileMapper.xml`
```sql
<delete id="deleteFile">
    DELETE FROM tbl_article_file
    WHERE file_name = #{fileName}
</delete>

<delete id="deleteFiles">
    DELETE FROM tbl_article_file
    WHERE article_no = #{articleNo}
</delete>

<insert id="replaceFile">
    INSERT INTO tbl_article_file (
        file_name
        , article_no
    ) VALUES (
        #{fileName}
        , #{articleNo}
    )
</insert>

<update id="updateFileCnt">
    UPDATE tbl_article
    SET file_cnt = (
        SELECT COUNT(article_no)
        FROM tbl_article_file
        WHERE article_no = #{articleNo}
    )
    WHERE article_no = #{articleNo}
</update>
```

#### # 게시글 서비스 계층 수정 : `ArticleServiceImpl`
첨부파일 기능이 추가되면서 게시글 서비스 계층의 수정처리는 아래와 같이 3가지 과정을 거치게 된다.

1. 게시글 수정처리
2. 기존의 첨부파일 정보는 전체 삭제처리
3. 첨부파일 정보를 새로 테이블에 입력 처리

위와 같이 3가지 작업이 순차적으로 동시에 이루어지기 때문에 트랜잭션처리를 위해 `@Transactional`애너테이션을 붙여준다.

```java
@Transactional
@Override
public void update(ArticleVO articleVO) throws Exception {
    Integer articleNo = articleVO.getArticleNo();
    String[] files = articleVO.getFiles();

    articleDAO.update(articleVO);
    articleFileDAO.deleteFiles(articleNo);

    if (files == null)
        return;
    for (String fileName : files)
        articleFileDAO.replaceFile(fileName, articleNo);
}
```

#### # 게시글 첨부파일 컨트롤러 : `ArticleFileController`
게시글 수정화면에서 첨부파일을 삭제할 경우, 게시글 입력페이지의 첨부파일 삭제와 다르게 서버에 저장된 파일뿐만 아니라 테이블의 파일정보도 함께 삭제처리해야하기 때문에 첨부파일 삭제처리 매핑 메서드를 추가적으로 작성해야한다. 특정 게시글의 첨부파일들을 조회할 수 있도록 `@PathVariable`애너테이션을 통해 게시글 번호를 가져오게 된다.
```java
// 게시글 첨부파일 삭제 : 게시글 수정
@RequestMapping(value = "/delete/{articleNo}", method = RequestMethod.POST)
public ResponseEntity<String> deleteFile(@PathVariable("articleNo") Integer articleNo,
                                         String fileName,
                                         HttpServletRequest request) {
    ResponseEntity<String> entity = null;

    try {
        UploadFileUtils.deleteFile(fileName, request);
        articleFileService.deleteFile(fileName, articleNo);
        entity = new ResponseEntity<>("DELETED", HttpStatus.OK);
    } catch (Exception e) {
        e.printStackTrace();
        entity = new ResponseEntity<>(e.getMessage(), HttpStatus.BAD_REQUEST);
    }

    return entity;
}
```

게시글을 수정처리 시에 서버에 저장된 게시글의 기존 첨부파일들을 삭제처리하기 위한 매핑 메서드를 아래와 같이 작성해준다.
```java
// 게시글 첨부파일 전체 삭제
@RequestMapping(value = "/deleteAll", method = RequestMethod.POST)
public ResponseEntity<String> deleteAllFiles(@RequestParam("files[]") String[] files, HttpServletRequest request) {

    if (files == null || files.length == 0)
        return new ResponseEntity<>("DELETED", HttpStatus.OK);

    ResponseEntity<String> entity = null;

    try {
        for (String fileName : files)
            UploadFileUtils.deleteFile(fileName, request);
        entity = new ResponseEntity<>("DELETED", HttpStatus.OK);
    } catch (Exception e) {
        e.printStackTrace();
        entity = new ResponseEntity<>(e.getMessage(), HttpStatus.BAD_REQUEST);
    }

    return entity;
}
```

#### # 게시글 수정 페이지 수정 : `modify.jsp`
##### 파일 업로드 영역 추가, Handlebars 파일 템플릿 작성
게시글 작성페이지와 마찬가지로 CSS, HTML코드, Handlebars 파일 템플릿 코드를 `modify.jsp`에 아래와 같이 작성해준다.
```css
.fileDrop {
    width: 100%;
    height: 200px;
    border: 2px dotted #0b58a2;
}
```
```html
<div class="form-group">
    <div class="fileDrop">
        <br/>
        <br/>
        <br/>
        <br/>
        <p class="text-center"><i class="fa fa-paperclip"></i> 첨부파일을 드래그해주세요.</p>
    </div>
</div>
```
```xml
<script id="fileTemplate" type="text/x-handlebars-template">
    <li>
        <span class="mailbox-attachment-icon has-img">
            <img src="{{imgSrc}}" alt="Attachment">
        </span>
        <div class="mailbox-attachment-info">
            <a href="{{originalFileUrl}}" class="mailbox-attachment-name">
                <i class="fa fa-paperclip"></i> {{originalFileName}}
            </a>
            <a href="{{fullName}}" class="btn btn-default btn-xs pull-right delBtn">
                <i class="fa fa-fw fa-remove"></i>
            </a>
        </div>
    </li>
</script>
```

##### 파일 업로드 JS파일 첨부파일 삭제 함수 추가 : `article_file_upload.js`
위에서 언급했던 것 처럼 게시글 입력페이지의 첨부파일 삭제와 다르게 수정페이지의 첨부파일 삭제는 게시글 번호가 필요하기 때문에 아래와 같이 요청 URL에 `articleNo`를 붙여서 파일 삭제 요청을 처리하도록 하였다.

```java
// 파일 삭제(입력페이지) : 첨부파일만 삭제처리
function deleteFileWrtPage(that) {
    var url = "/article/file/delete";
    deleteFile(url, that);
}

// 추가된 함수
// 파일 삭제(수정페이지) : 서버에 저장된 첨부파일과 DB에 저장된 첨부파일 정보 삭제처리
function deleteFileModPage(that, articleNo) {
    var url = "/article/file/delete/" + articleNo;
    deleteFile(url, that);
}

// 파일 삭제 AJAX 통신
function deleteFile(url, that) {
    $.ajax({
        url: url,
        type: "post",
        data: {fileName: that.attr("href")},
        dataType: "text",
        success: function (result) {
            if (result === "DELETED") {
                alert("삭제되었습니다.");
                that.parents("li").remove();
            }
        }
    });
}
```

##### 수정페이지의 JS코드
수정페이지에서의 첨부파일 삭제처리와 수정처리 코드는 아래처럼 수정해준다.
```js
// 전역 변수 선언
var articleNo = "${article.articleNo}"; // 현재 게시글 번호

// 첨부파일 삭제 버튼 클릭 이벤트
$(document).on("click", ".delBtn", function (event) {
    event.preventDefault();
    if (confirm("삭제하시겠습니까? 삭제된 파일은 복구할 수 없습니다.")) {
        var that = $(this);
        deleteFileModPage(that, articleNo);
    }
});

// 첨부파일 목록 호출
getFiles(articleNo);

// 수정 처리시 첨부파일 정보도 함께 처리
$("#modifyForm").submit(function (event) {
    event.preventDefault();
    var that = $(this);
    filesSubmit(that);
});
```

#### # 구현 확인

##### 게시글 수정페이지에서 파일 업로드 영역, 파일목록 출력확인
![file8](https://github.com/walbatrossw/develop-notes/blob/master/reding-notes/%EC%BD%94%EB%93%9C%EB%A1%9C_%EB%B0%B0%EC%9A%B0%EB%8A%94_%EC%8A%A4%ED%94%84%EB%A7%81_%EC%9B%B9%ED%94%84%EB%A1%9C%EC%A0%9D%ED%8A%B8/photo/2018-04-06%2015-42-56.png?raw=true)

##### 게시글 수정페이지의 파일 삭제처리 확인
파일 삭제버튼 클릭, 파일 삭제 확인 메시지 alert창 팝업 확인
![file9](https://github.com/walbatrossw/develop-notes/blob/master/reding-notes/%EC%BD%94%EB%93%9C%EB%A1%9C_%EB%B0%B0%EC%9A%B0%EB%8A%94_%EC%8A%A4%ED%94%84%EB%A7%81_%EC%9B%B9%ED%94%84%EB%A1%9C%EC%A0%9D%ED%8A%B8/photo/2018-04-06%2015-56-45.png?raw=true)

파일 삭제 처리 확인(서버 첨부파일 삭제, 테이블 첨부파일 정보 삭제)
![file10](https://github.com/walbatrossw/develop-notes/blob/master/reding-notes/%EC%BD%94%EB%93%9C%EB%A1%9C_%EB%B0%B0%EC%9A%B0%EB%8A%94_%EC%8A%A4%ED%94%84%EB%A7%81_%EC%9B%B9%ED%94%84%EB%A1%9C%EC%A0%9D%ED%8A%B8/photo/2018-04-06%2016-00-26.png?raw=true)

##### 게시글 수정페이지의 첨부파일 수정처리 확인
새로운 첨부파일 추가 업로드
![file11](https://github.com/walbatrossw/develop-notes/blob/master/reding-notes/%EC%BD%94%EB%93%9C%EB%A1%9C_%EB%B0%B0%EC%9A%B0%EB%8A%94_%EC%8A%A4%ED%94%84%EB%A7%81_%EC%9B%B9%ED%94%84%EB%A1%9C%EC%A0%9D%ED%8A%B8/photo/2018-04-06%2016-05-19.png?raw=true)

게시글 수정처리완료 후 게시글 조회페이지, 테이블에 수정된 파일정보 입력확인
![file12](https://github.com/walbatrossw/develop-notes/blob/master/reding-notes/%EC%BD%94%EB%93%9C%EB%A1%9C_%EB%B0%B0%EC%9A%B0%EB%8A%94_%EC%8A%A4%ED%94%84%EB%A7%81_%EC%9B%B9%ED%94%84%EB%A1%9C%EC%A0%9D%ED%8A%B8/photo/2018-04-06%2016-07-58.png?raw=true)

## 8. 게시글 첨부파일 기능 구현 정리

게시글의 첨부파일 기능을 구현하는데 책의 내용을 그대로 작성하지 않고, 나름 열심히 고민해서 구현했다. 변수명과 메서드명을 보다 이해하기 쉽게 변경하였고, 파일업로드 유틸 클래스의 메서드와 자바스크립트 코드는 최대한 재사용이 가능하게 잘게 쪼개서 작성하였다. 그리고 필요하다고 생각되는 기능이나 로직을 추가도 해보았다. 전에는 무작정 책의 코드를 따라치고, 기능을 구현하는 것에만 집중해서 정작 시간이 지나고 다시 코드를 보면 왜 이렇게 코드를 작성했는지 이해가 안되는 경우가 많았는데 이번에는 일일이 코드를 분석하고, 나에 맞게 변경해보면서 코드의 내용이 머리 속에 남게 되었다. 덕분에 기능을 구현하면서 오류를 찾고, 정리하는데 시간이 오래 걸렸지만 나름 스스로 만족스러웠다.
하지만 그럼에도 불구하고 여전히 불만족스러운 부분이 존재하는데 그 부분은 입력페이지와 수정페이지에서 파일업로드를 하고, 게시글을 저장하지 않고 다른페이지로 이동하게 되면 업로드한 파일은 여전히 서버에 저장되는 것인데 이 부분을 어떻게 처리할지 고민을 더해봐야겠다.
