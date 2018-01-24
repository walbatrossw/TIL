본 포스팅은 [코드로 배우는스프링 웹프로젝트](http://www.yes24.com/24/goods/19720776?scode=032&OzSrank=1)를 참조하여 작성한 내용입니다. 개인적으로 학습한 내용을 복습하기 위한 내용이기 때문에 내용상 오류가 있을 수 있습니다. 기존의 Spring MVC 관련 포스팅들이 제대로 정리되지 않은 것 같아 처음부터 차분히 정리하면서 포스팅을 진행하고 있습니다.


이번에는 Controller에서 가장 빈번하게 사용하는 5가지의 경우를 통해 Controller를 작성하는 방법을 연습해보고, 작성한 Controller를 WAS없이 테스트해보자.

## 01) `void`리턴 타입인 경우
- `VoidController`
  ```java
  @Controller
  public class VoidController {

      private static final Logger logger = LoggerFactory.getLogger(VoidController.class);

      @RequestMapping("doA")
      public void doA() {
          logger.info("doA called....");
      }

      @RequestMapping("doB")
      public void doB() {
          logger.info("doB called....");
      }

  }
  ```
- Browser 및 console 화면
  ![doA](http://cfile9.uf.tistory.com/image/99F03C335A1959261D6B69)
  ![doB](http://cfile5.uf.tistory.com/image/994DB6335A195926046213)
## 02) `String`리턴 타입인 경우
- `StringController`
  ```java
  @Controller
  public class StringController {

      private static final Logger logger = LoggerFactory.getLogger(StringController.class);

      @RequestMapping("doC")
      public String doC(@ModelAttribute("msg") String msg) {
          logger.info("doC called....");

          return "result";
      }

  }
  ```

- `result.jsp`
  ```xml
  <%@ page contentType="text/html;charset=UTF-8" language="java" %>
  <html>
  <head>
      <title>doC Result</title>
  </head>
  <body>
      <h2>doC</h2>
      <p>message : ${msg}</p>
  </body>
  </html>
  ```
- Browser 및 console 화면
  ![doC](http://cfile24.uf.tistory.com/image/993A52335A1959262BE936)

## 03) 만들어진 결과 데이터를 view에 전달할 경우
- `VoController`
  ```java
  @Controller
  public class VoController {

      private static final Logger logger = LoggerFactory.getLogger(VoController.class);

      @RequestMapping("/doD")
      public String doD(Model model) {
          ProductVO vo = new ProductVO("sample product", 100000);
          logger.info("doD : " + vo);
          model.addAttribute(vo);
          model.addAttribute("vo", vo);

          return "productDetail";
      }
  }
  ```
- `ProductVO`
  ```java
  public class ProductVO {

      private String name;
      private double price;

      public ProductVO(String name, double price) {
          super();
          this.name = name;
          this.price = price;
      }

      public String getName() {
          return name;
      }

      public double getPrice() {
          return price;
      }

      @Override
      public String toString() {
          return "ProductVO{" +
                  "name='" + name + '\'' +
                  ", price=" + price +
                  '}';
      }
  }
  ```
- `productDetail.jsp`
  ```xml
  <%@ page contentType="text/html;charset=UTF-8" language="java" %>
  <html>
  <head>
      <title>doD</title>
  </head>
  <body>
      <h2>doD</h2>
      <p>model.addAttribute(vo) : ${productVO}</p>
      <br/>
      <p>model.addAttribute("vo", vo) : ${vo}</p>
  </body>
  </html>
  ```
- Browser 및 console 화면
  ![doD](http://cfile21.uf.tistory.com/image/9984E9335A195926035E2B)

## 04) `redirect`를 해야할 경우
- `RedirectController`
  ```java
  @Controller
  public class RedirectController {

      private static final Logger logger = LoggerFactory.getLogger(RedirectController.class);

      @RequestMapping("/doE")
      public String doE(RedirectAttributes rttr) {
          logger.info("doE called but redirect to /doF");
          rttr.addFlashAttribute("msg", "This is message! with redirected");
          return "redirect:/doF";
      }

      @RequestMapping("/doF")
      public void doF(@ModelAttribute String msg) {
          logger.info("doF called...." + msg);
      }
  }
  ```
- Browser 및 console 화면
  ![doE](http://cfile4.uf.tistory.com/image/998942335A19592620DC4B)

## 05) `JSON`데이터를 생성하는 경우
- `JsonController`
  ```java
  @Controller
  public class JsonController {

      private static final Logger logger = LoggerFactory.getLogger(JsonController.class);

      @RequestMapping("doJson")
      @ResponseBody
      public ProductVO doJson() {
          ProductVO vo = new ProductVO("sample product", 300000);
          logger.info(String.valueOf(vo));
          return vo;
      }
  }
  ```

- Browser 및 console 화면
  ![doJson](http://cfile27.uf.tistory.com/image/990799335A19592609ACEB)

## 06) WAS없이 Controller 테스트

이번에는 `spring-test`모듈을 통해 별도의 WAS구동없이 컨트롤러를 테스트를 해보자. `Spring 3.2` 버전부터는 `jUnit`만을 사용하여 Spring MVC에서 작성된 컨트롤러를 테스트할 수 있었는데, WAS에서 테스트를 하는 것이 어려울 경우 아주 유용하게 사용할 수 있다.

- `pom.xml` : 테스트 진행을 위해 `javax.servlet` 라이브러리 버전 변경
  - 기존의 라이브러리
    ```xml
    <dependency>
        <groupId>javax.servlet</groupId>
        <artifactId>servlet-api</artifactId>
        <version>2.5</version>
        <scope>provided</scope>
    </dependency>
    ```
  - 버전을 변경한 라이브러리
    ```xml
    <dependency>
        <groupId>javax.servlet</groupId>
        <artifactId>javax.servlet-api</artifactId>
        <version>3.1.0</version>
    </dependency>
    ```

- 테스트 코드 작성
  ```java
  @RunWith(SpringJUnit4ClassRunner.class)
  @WebAppConfiguration
  @ContextConfiguration(locations = {"file:src/main/webapp/WEB-INF/spring/*.xml"})
  public class SampleControllerTest {

      private static final Logger logger = LoggerFactory.getLogger(SampleControllerTest.class);

      @Inject
      private WebApplicationContext wac;

      // 브라우저에서 요청과 응답을 의미하는 객체
      private MockMvc mockMvc;

      @Before // 테스트를 진행하기전 수행하는 작업
      public void setup() {
          // MockMvc 객체를 생성
          this.mockMvc = MockMvcBuilders.webAppContextSetup(this.wac).build();
          logger.info("setup.....");
      }

      // 각각의 Controller 테스트 코드 작성

      @Test
      public void testDoA() throws Exception {
          mockMvc.perform(MockMvcRequestBuilders.get("/doA"))
                  .andDo(print())
                  .andExpect(status().isOk());
      }

      @Test
      public void testDoB() throws Exception {
          mockMvc.perform(MockMvcRequestBuilders.get("/doB"))
                  .andDo(print())
                  .andExpect(status().isOk());
      }

      @Test
      public void testDoC() throws Exception {
          mockMvc.perform(MockMvcRequestBuilders.get("/doC?msg=Hello"))
                  .andDo(print())
                  .andExpect(status().isOk());
      }

      @Test
      public void testDoD() throws Exception {
          mockMvc.perform(MockMvcRequestBuilders.get("/doD"))
                  .andDo(print())
                  .andExpect(status().isOk())
                  .andExpect(model().attributeExists("vo"));
      }

      @Test
      public void testDoE() throws Exception {
          mockMvc.perform(MockMvcRequestBuilders.get("/doE"))
                  .andDo(print())
                  .andExpect(status().is3xxRedirection())
                  .andExpect(redirectedUrl("/doF"));
      }

      @Test
      public void testDoJson() throws Exception {
          mockMvc.perform(MockMvcRequestBuilders.get("/doJson"))
                  .andDo(print())
                  .andExpect(status().isOk())
                  .andExpect(content().contentType("application/json;charset=utf-8"));
      }
  }
  ```
- 테스트 실행 결과
  - `testDoA()`
    ```
    INFO : com.doubles.ex00.SampleControllerTest - setup.....
    INFO : com.doubles.ex00.sample.VoidController - doA called....

    MockHttpServletRequest:
          HTTP Method = GET
          Request URI = /doA
           Parameters = {}
              Headers = {}

    Handler:
                 Type = com.doubles.ex00.sample.VoidController
               Method = public void com.doubles.ex00.sample.VoidController.doA()

    Async:
        Async started = false
         Async result = null

    Resolved Exception:
                 Type = null

    ModelAndView:
            View name = doA
                 View = null
                Model = null

    FlashMap:
           Attributes = null

    MockHttpServletResponse:
               Status = 200
        Error message = null
              Headers = {}
         Content type = null
                 Body =
        Forwarded URL = /WEB-INF/views/doA.jsp
       Redirected URL = null
              Cookies = []

    ```
  - `testDoB()`
    ```
    INFO : com.doubles.ex00.SampleControllerTest - setup.....
    INFO : com.doubles.ex00.sample.VoidController - doB called....

    MockHttpServletRequest:
          HTTP Method = GET
          Request URI = /doB
           Parameters = {}
              Headers = {}

    Handler:
                 Type = com.doubles.ex00.sample.VoidController
               Method = public void com.doubles.ex00.sample.VoidController.doB()

    Async:
        Async started = false
         Async result = null

    Resolved Exception:
                 Type = null

    ModelAndView:
            View name = doB
                 View = null
                Model = null

    FlashMap:
           Attributes = null

    MockHttpServletResponse:
               Status = 200
        Error message = null
              Headers = {}
         Content type = null
                 Body =
        Forwarded URL = /WEB-INF/views/doB.jsp
       Redirected URL = null
              Cookies = []

    ```
  - `testDoC()`
    ```
    INFO : com.doubles.ex00.SampleControllerTest - setup.....
    INFO : com.doubles.ex00.sample.StringController - doC called....
    INFO : com.doubles.ex00.sample.StringController - msg = Hello

    MockHttpServletRequest:
          HTTP Method = GET
          Request URI = /doC
           Parameters = {msg=[Hello]}
              Headers = {}

    Handler:
                 Type = com.doubles.ex00.sample.StringController
               Method = public java.lang.String com.doubles.ex00.sample.StringController.doC(java.lang.String)

    Async:
        Async started = false
         Async result = null

    Resolved Exception:
                 Type = null

    ModelAndView:
            View name = result
                 View = null
            Attribute = msg
                value = Hello
               errors = []

    FlashMap:
           Attributes = null

    MockHttpServletResponse:
               Status = 200
        Error message = null
              Headers = {}
         Content type = null
                 Body =
        Forwarded URL = /WEB-INF/views/result.jsp
       Redirected URL = null
              Cookies = []
    ```
  - `testDoD()`
    ```
    INFO : com.doubles.ex00.SampleControllerTest - setup.....
    INFO : com.doubles.ex00.sample.VoController - doD : ProductVO{name='sample product', price=100000.0}

    MockHttpServletRequest:
          HTTP Method = GET
          Request URI = /doD
           Parameters = {}
              Headers = {}

    Handler:
                 Type = com.doubles.ex00.sample.VoController
               Method = public java.lang.String com.doubles.ex00.sample.VoController.doD(org.springframework.ui.Model)

    Async:
        Async started = false
         Async result = null

    Resolved Exception:
                 Type = null

    ModelAndView:
            View name = productDetail
                 View = null
            Attribute = productVO
                value = ProductVO{name='sample product', price=100000.0}
               errors = []
            Attribute = vo
                value = ProductVO{name='sample product', price=100000.0}
               errors = []

    FlashMap:
           Attributes = null

    MockHttpServletResponse:
               Status = 200
        Error message = null
              Headers = {}
         Content type = null
                 Body =
        Forwarded URL = /WEB-INF/views/productDetail.jsp
       Redirected URL = null
              Cookies = []
    ```
  - `testDoE()`
    ```
    INFO : com.doubles.ex00.SampleControllerTest - setup.....
    INFO : com.doubles.ex00.sample.RedirectController - doE called but redirect to /doF

    MockHttpServletRequest:
          HTTP Method = GET
          Request URI = /doE
           Parameters = {}
              Headers = {}

    Handler:
                 Type = com.doubles.ex00.sample.RedirectController
               Method = public java.lang.String com.doubles.ex00.sample.RedirectController.doE(org.springframework.web.servlet.mvc.support.RedirectAttributes)

    Async:
        Async started = false
         Async result = null

    Resolved Exception:
                 Type = null

    ModelAndView:
            View name = redirect:/doF
                 View = null
                Model = null

    FlashMap:
            Attribute = msg
                value = This is message! with redirected

    MockHttpServletResponse:
               Status = 302
        Error message = null
              Headers = {Location=[/doF]}
         Content type = null
                 Body =
        Forwarded URL = null
       Redirected URL = /doF
              Cookies = []
    ```
  - `testDoJson()`
    ```
    INFO : com.doubles.ex00.SampleControllerTest - setup.....
    INFO : com.doubles.ex00.sample.JsonController - ProductVO{name='sample product', price=300000.0}

    MockHttpServletRequest:
          HTTP Method = GET
          Request URI = /doJson
           Parameters = {}
              Headers = {}

    Handler:
                 Type = com.doubles.ex00.sample.JsonController
               Method = public com.doubles.ex00.domain.ProductVO com.doubles.ex00.sample.JsonController.doJson()

    Async:
        Async started = false
         Async result = null

    Resolved Exception:
                 Type = null

    ModelAndView:
            View name = null
                 View = null
                Model = null

    FlashMap:
           Attributes = null

    MockHttpServletResponse:
               Status = 200
        Error message = null
              Headers = {Content-Type=[application/json;charset=UTF-8]}
         Content type = application/json;charset=UTF-8
                 Body = {"name":"sample product","price":300000.0}
        Forwarded URL = null
       Redirected URL = null
              Cookies = []
    ```

* 테스트 코드를 작성하고 테스트를 하는 것의 장점
  *  웹페이지를 테스트하려면 매번 입력 항목을 입력해서 제대로 동작하는지 확인하는데, 여러번 웹페이지를 입력하는 것보다 테스트 코드를 통해 처리하는 것이 개발 시간을 단축할 수 있다.
  * JSP 등에서 발생하는 에러를 해결하는 과정에서 매법 WAS에 만들어진 Controller 코드를 수정해서 배포하는 작업은 많은 시간을 소모한다.
  * Controller에서 결과 데이터만을 확인할 수 있기 떄문에 문제 발생시 원인을 파악할 수 있는 시간이 절약된다.
