# 컨트롤러 작성연습, WAS 없이 컨트롤러 테스트

본 포스팅은 [코드로 배우는스프링 웹프로젝트](http://www.yes24.com/24/goods/19720776?scode=032&OzSrank=1)를 참조하여 작성한 내용입니다. 개인적으로 학습한 내용을 복습하기 위한 내용이기 때문에 내용상 오류가 있을 수 있습니다. 기존의 Spring MVC 관련 포스팅들이 제대로 정리되지 않은 것 같아 처음부터 차분히 정리하면서 포스팅을 진행하고 있습니다.

그리고 본 포스팅의 예제는 STS 또는 Eclipse를 사용하지 않고 IntelliJ를 통해 구현하고 있습니다. 그래서 기존의 STS에서 생성된 Spring 프로젝트의 스프링관련 설정 파일명과 프로젝트 구조가 약간 다를 수 있습니다. [IntelliJ를 통한 Spring MVC 프로젝트 생성](http://doublesprogramming.tistory.com/171?category=667155) 포스팅을 참고해주시면 감사하겠습니다.

포스팅하고 있는 현재 프로젝트의 예제가 혹시 필요하신 분은 github(https://github.com/walbatrossw/spring-mvc-ex)를 통해 얻으실 수 있습니다.

---

이번에는 Controller에서 가장 빈번하게 사용하는 5가지의 경우를 통해 Controller를 작성하는 방법을 연습해보고, 작성한 Controller를 WAS없이 테스트해보자.

## 1. `void`리턴 타입인 경우
`VoidController`작성
```java
@Controller
public class VoidController {

    private static final Logger logger = LoggerFactory.getLogger(VoidController.class);

    @RequestMapping("/doA")
    public void doA() {
        logger.info("/doA called...");
    }

    @RequestMapping("/doB")
    public void doB() {
        logger.info("/doB called...");
    }

}
```
`/doA`, `/doB`요청시 브라우저와 콘솔화면 화면

![doA](https://github.com/walbatrossw/TIL/blob/master/04_spring-framework_orm/spring-mvc-basic/img/spring_controller_test_without_was/doA.png?raw=true)

![doB](https://github.com/walbatrossw/TIL/blob/master/04_spring-framework_orm/spring-mvc-basic/img/spring_controller_test_without_was/doB.png?raw=true)

## 2. `String`리턴 타입인 경우
`StringController`을 아래와 같이 작성해준다.
```java
@Controller
public class StringController {

    private static final Logger logger = LoggerFactory.getLogger(StringController.class);

    @RequestMapping("/doC")
    public String doC(@ModelAttribute("msg") String msg) {
        logger.info("/doC called");
        return "tutorial/result";
    }

}
```

`result.jsp`을 아래와 같이 작성해준다.
```xml
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
<head>
    <title>String Controller Test</title>
</head>
<body>
    <span>Hello ${msg}</span>
</body>
</html>
```
`/doC` 요청시 브라우저와 콘솔화면 확인

![doC](https://github.com/walbatrossw/TIL/blob/master/04_spring-framework_orm/spring-mvc-basic/img/spring_controller_test_without_was/doC.png?raw=true)

## 3. 만들어진 결과 데이터를 view에 전달할 경우
`DomainController`을 아래와 같이 작성해준다.
```java
@Controller
public class DomainController {

    private static final Logger logger = LoggerFactory.getLogger(DomainController.class);

    @RequestMapping("/doD")
    public String doD(Model model) {

        ProductVO product = new ProductVO("desktop", 10000);
        logger.info("/doD called");
        model.addAttribute(product);

        return "/tutorial/product_detail";
    }

}
```
`ProductVO`을 아래와 같이 작성해준다.
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
`product_detail.jsp`을 아래와 같이 작성해준다.
```xml
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
<head>
    <title>Domain Controller Test</title>
</head>
<body>
    <span>${productVO.name}</span>
    <span>${productVO.price}</span>
</body>
</html>
```
`/doD` 요청시 브라우저와 콘솔화면 확인

![doD](https://github.com/walbatrossw/TIL/blob/master/04_spring-framework_orm/spring-mvc-basic/img/spring_controller_test_without_was/doD.png?raw=true)

## 4. `redirect`를 해야할 경우
`RedirectController`을 아래와 같이 작성해준다.
```java
@Controller
public class RedirectController {

    private static final Logger logger = LoggerFactory.getLogger(RedirectController.class);

    @RequestMapping("/doE")
    public String doE(RedirectAttributes redirectAttributes) {

        logger.info("/doE called and redirect to /doF");
        redirectAttributes.addAttribute("msg", "this is the message with redirected");

        return "redirect:/doF";
    }

    @RequestMapping("/doF")
    public void doF(@ModelAttribute String msg) {

        logger.info("/doF called" + msg);

    }

}
```
`/doE` 요청시 브라우저와 콘솔화면 확인

![doE](https://github.com/walbatrossw/TIL/blob/master/04_spring-framework_orm/spring-mvc-basic/img/spring_controller_test_without_was/doE.png?raw=true)

## 5. `JSON`데이터를 생성하는 경우
`JsonController`을 아래와 같이 작성해준다.
```java
@Controller
public class JasonController {

    @RequestMapping("/doJson")
    @ResponseBody
    public ProductVO doJson() {

        ProductVO productVO = new ProductVO("laptop", 3000000);

        return productVO;
    }

}
```

`/doJson` 요청시 브라우저와 콘솔화면 확인

![doJson](https://github.com/walbatrossw/TIL/blob/master/04_spring-framework_orm/spring-mvc-basic/img/spring_controller_test_without_was/doJson.png?raw=true)

## 6. WAS없이 Controller 테스트

이번에는 `spring-test`모듈을 통해 별도의 WAS구동없이 컨트롤러를 테스트를 해보자. `Spring 3.2` 버전부터는 `jUnit`만을 사용하여 Spring MVC에서 작성된 컨트롤러를 테스트할 수 있었는데, WAS에서 테스트를 하는 것이 어려울 경우 아주 유용하게 사용할 수 있다.

`pom.xml` : 테스트 진행을 위해 `javax.servlet` 라이브러리 버전 변경
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

테스트 코드를 아래와 같이 작성해준다.
```java
import org.junit.Before;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.test.context.ContextConfiguration;
import org.springframework.test.context.junit4.SpringJUnit4ClassRunner;
import org.springframework.test.context.web.WebAppConfiguration;
import org.springframework.test.web.servlet.MockMvc;
import org.springframework.test.web.servlet.request.MockMvcRequestBuilders;
import org.springframework.test.web.servlet.setup.MockMvcBuilders;
import org.springframework.web.context.WebApplicationContext;

import javax.inject.Inject;

import static org.springframework.test.web.servlet.result.MockMvcResultHandlers.print;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.*;

@RunWith(SpringJUnit4ClassRunner.class)
@WebAppConfiguration
@ContextConfiguration(locations = {"file:src/main/webapp/WEB-INF/spring-config/**/*.xml"})
public class ControllerTest {

    private static final Logger logger = LoggerFactory.getLogger(ControllerTest.class);

    @Inject
    private WebApplicationContext webApplicationContext;

    private MockMvc mockMvc;

    @Before
    public void setup() {
        this.mockMvc = MockMvcBuilders.webAppContextSetup(this.webApplicationContext).build();
        logger.info("setup...");
    }

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
        mockMvc.perform(MockMvcRequestBuilders.get("/doC?msg=world"))
                .andDo(print())
                .andExpect(status().isOk());
    }

    @Test
    public void testDoD() throws Exception {
        mockMvc.perform(MockMvcRequestBuilders.get("/doD"))
                .andDo(print())
                .andExpect(status().isOk())
                .andExpect(model().attributeExists("productVO"));
    }

    @Test
    public void testDoE() throws Exception {
        mockMvc.perform(MockMvcRequestBuilders.get("/doE"))
                .andDo(print())
                .andExpect(status().is3xxRedirection())
                .andExpect(redirectedUrl("/doF?msg=this+is+the+message+with+redirected"));
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
테스트 실행 결과 콘솔에 출력된 내용을 아래와 같다.

**`testDoA()`**
```
INFO : com.doubles.mvcboard.tutorial.ControllerTest - setup...
INFO : com.doubles.mvcboard.tutorial.controller.VoidController - /doA called...

MockHttpServletRequest:
      HTTP Method = GET
      Request URI = /doA
       Parameters = {}
          Headers = {}

Handler:
             Type = com.doubles.mvcboard.tutorial.controller.VoidController
           Method = public void com.doubles.mvcboard.tutorial.controller.VoidController.doA()

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

**`testDoB()`**
```
INFO : com.doubles.mvcboard.tutorial.ControllerTest - setup...
INFO : com.doubles.mvcboard.tutorial.controller.VoidController - /doB called...

MockHttpServletRequest:
      HTTP Method = GET
      Request URI = /doB
       Parameters = {}
          Headers = {}

Handler:
             Type = com.doubles.mvcboard.tutorial.controller.VoidController
           Method = public void com.doubles.mvcboard.tutorial.controller.VoidController.doB()

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

**`testDoC()`**
```
INFO : com.doubles.mvcboard.tutorial.ControllerTest - setup...
INFO : com.doubles.mvcboard.tutorial.controller.StringController - /doC called

MockHttpServletRequest:
      HTTP Method = GET
      Request URI = /doC
       Parameters = {msg=[world]}
          Headers = {}

Handler:
             Type = com.doubles.mvcboard.tutorial.controller.StringController
           Method = public java.lang.String com.doubles.mvcboard.tutorial.controller.StringController.doC(java.lang.String)

Async:
    Async started = false
     Async result = null

Resolved Exception:
             Type = null

ModelAndView:
        View name = tutorial/result
             View = null
        Attribute = msg
            value = world
           errors = []

FlashMap:
       Attributes = null

MockHttpServletResponse:
           Status = 200
    Error message = null
          Headers = {}
     Content type = null
             Body =
    Forwarded URL = /WEB-INF/views/tutorial/result.jsp
   Redirected URL = null
          Cookies = []
```

**`testDoD()`**
```
INFO : com.doubles.mvcboard.tutorial.ControllerTest - setup...
INFO : com.doubles.mvcboard.tutorial.controller.DomainController - /doD called

MockHttpServletRequest:
      HTTP Method = GET
      Request URI = /doD
       Parameters = {}
          Headers = {}

Handler:
             Type = com.doubles.mvcboard.tutorial.controller.DomainController
           Method = public java.lang.String com.doubles.mvcboard.tutorial.controller.DomainController.doD(org.springframework.ui.Model)

Async:
    Async started = false
     Async result = null

Resolved Exception:
             Type = null

ModelAndView:
        View name = /tutorial/product_detail
             View = null
        Attribute = productVO
            value = ProductVO{name='desktop', price=10000.0}
           errors = []

FlashMap:
       Attributes = null

MockHttpServletResponse:
           Status = 200
    Error message = null
          Headers = {}
     Content type = null
             Body =
    Forwarded URL = /WEB-INF/views//tutorial/product_detail.jsp
   Redirected URL = null
          Cookies = []
```

**`testDoE()`**
```
INFO : com.doubles.mvcboard.tutorial.ControllerTest - setup...
INFO : com.doubles.mvcboard.tutorial.controller.RedirectController - /doE called and redirect to /doF

MockHttpServletRequest:
      HTTP Method = GET
      Request URI = /doE
       Parameters = {}
          Headers = {}

Handler:
             Type = com.doubles.mvcboard.tutorial.controller.RedirectController
           Method = public java.lang.String com.doubles.mvcboard.tutorial.controller.RedirectController.doE(org.springframework.web.servlet.mvc.support.RedirectAttributes)

Async:
    Async started = false
     Async result = null

Resolved Exception:
             Type = null

ModelAndView:
        View name = redirect:/doF
             View = null
        Attribute = msg
            value = this is the message with redirected

FlashMap:
       Attributes = null

MockHttpServletResponse:
           Status = 302
    Error message = null
          Headers = {Location=[/doF?msg=this+is+the+message+with+redirected]}
     Content type = null
             Body =
    Forwarded URL = null
   Redirected URL = /doF?msg=this+is+the+message+with+redirected
          Cookies = []
```

**`testDoJson()`**
```
INFO : com.doubles.mvcboard.tutorial.ControllerTest - setup...

MockHttpServletRequest:
      HTTP Method = GET
      Request URI = /doJson
       Parameters = {}
          Headers = {}

Handler:
             Type = com.doubles.mvcboard.tutorial.controller.JasonController
           Method = public com.doubles.mvcboard.tutorial.domain.ProductVO com.doubles.mvcboard.tutorial.controller.JasonController.doJson()

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
             Body = {"name":"laptop","price":3000000.0}
    Forwarded URL = null
   Redirected URL = null
          Cookies = []
```


#### 테스트 코드를 작성하고 테스트를 하는 것의 장점
*  웹페이지를 테스트하려면 매번 입력 항목을 입력해서 제대로 동작하는지 확인하는데, 여러번 웹페이지를 입력하는 것보다 테스트 코드를 통해 처리하는 것이 개발 시간을 단축할 수 있다.
* JSP 등에서 발생하는 에러를 해결하는 과정에서 매법 WAS에 만들어진 Controller 코드를 수정해서 배포하는 작업은 많은 시간을 소모한다.
* Controller에서 결과 데이터만을 확인할 수 있기 떄문에 문제 발생시 원인을 파악할 수 있는 시간이 절약된다.
