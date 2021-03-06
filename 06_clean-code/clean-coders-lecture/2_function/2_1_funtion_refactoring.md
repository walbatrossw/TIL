
# Clean Coders : Function Part 1

## 1. 원칙
- 함수는 한가지 일만 수행해야한다.
- 그렇다면 함수의 크기는???
    - 과거에는 작은 화면에 볼 수 있도록 20줄 이내이어야한다고 했지만, 요즘은 대형모니터를 사용하거나 두대 이상의 모니터를 사용하기 때문에 작을 이유는 없다.
- indentation(들여쓰기), while, nested if(중첩if문) 등은 없어야 한다.
- 잘 지어진 서술적인 긴 이름을 갖는 많은 갯수의 작은 함수들로 유지해야한다.
  - small many functions
  - nice descriptive long name functions

## 2. The First Rule of Functions
- 더 이상 작아질 수 없을 만큼 작아야한다.
- 큰 함수를 보면 클래스로 추출할 생각을 해야한다. -> extract method object(리펙토링기법)
- 클래스는 일련의 변수들에 동작하는 기능들의 집합이다.

## 3. Fitness Example - Big Function Refactoring(Extract Method Object)
1. extract method object(메서드 객체 추출) - TestalbeHtmlMaker
    ![1](https://github.com/walbatrossw/TIL/blob/master/06_clean-code/clean-coders-lecture/2_function/fitness-example-code-diff-pic2/1.png?raw=true)
    - 여러 변수에 동작하는 큰 함수는 여러 필드에 대한 기능을 갖는 객체이다.
    - 앞서 언급한 것 처럼 큰 함수를 보면 클래스로 추출할 생각을 해야한다고 했던 것처럼 큰 함수를 추출
2. extract field(필드 변수 추출) - wikiPage, buffer
    ![2](https://github.com/walbatrossw/TIL/blob/master/06_clean-code/clean-coders-lecture/2_function/fitness-example-code-diff-pic2/2.png?raw=true)
    - 함수의 여러 곳에서 공통적으로 사용되는 변수는 필드 변수로 변환함으로써 메서드 추출시 파라미터 개수를 줄일 수 있고, 2개 이상의 변수가 수정되어 메소드 추출이 불가능한 경우를 방지할 수 있다.
3. extract method - includePage : 중복코드를 메서드로 추출
    - extract variable - mode : 중복코드 중에서 서로 다른 부분들을 파라미터로 처리하기 위해 변수로 추출
    ![3.1](https://github.com/walbatrossw/TIL/blob/master/06_clean-code/clean-coders-lecture/2_function/fitness-example-code-diff-pic2/3.1.png?raw=true)
    - move line up for extract method : 추출된 변수의 라인이동
    ![3.2](https://github.com/walbatrossw/TIL/blob/master/06_clean-code/clean-coders-lecture/2_function/fitness-example-code-diff-pic2/3.2.png?raw=true)
    - extract method - includePage : 메서드로 추출
    ![3.3](https://github.com/walbatrossw/TIL/blob/master/06_clean-code/clean-coders-lecture/2_function/fitness-example-code-diff-pic2/3.3.png?raw=true)

4. extract method - includeInherited  
    - extract variable - pageName for extract method : 중복코드 중에서 서로 다른 부분들을 파라미터로 처리하기 위해 변수로 추출
    ![4.1](https://github.com/walbatrossw/TIL/blob/master/06_clean-code/clean-coders-lecture/2_function/fitness-example-code-diff-pic2/4.1.png?raw=true)
    - move line up for extract method : 추출된 변수의 라인이동
    ![4.2](https://github.com/walbatrossw/TIL/blob/master/06_clean-code/clean-coders-lecture/2_function/fitness-example-code-diff-pic2/4.2.1.png?raw=true)
    - extract method - includeInherited : 추출후 불필요한 괄호 제거(괄호는 하나 이상의 책임을 갖는다는 증후)
    ![4.3](https://github.com/walbatrossw/TIL/blob/master/06_clean-code/clean-coders-lecture/2_function/fitness-example-code-diff-pic2/4.3.png?raw=true)
    - inline variable after extract method : 중복 코드에서 다른 부분을 파라미터 처리를 위해 추출한 변수들을 인라인하여 코드를 깨끗하게 정리
    ![4.4](https://github.com/walbatrossw/TIL/blob/master/06_clean-code/clean-coders-lecture/2_function/fitness-example-code-diff-pic2/4.4.png?raw=true)

5. extract method - includeSetups, includeTearDowns
  ![5](https://github.com/walbatrossw/TIL/blob/master/06_clean-code/clean-coders-lecture/2_function/fitness-example-code-diff-pic2/5.png?raw=true)
6. merge if statement : 동일한 조건의 if 문장을 합쳐 가독성을 증대시키자.
  ![6](https://github.com/walbatrossw/TIL/blob/master/06_clean-code/clean-coders-lecture/2_function/fitness-example-code-diff-pic2/6.png?raw=true)
7. if condition extract method : if문의 조건을 메서드로 추출하여 가독성 증대
  ![7](https://github.com/walbatrossw/TIL/blob/master/06_clean-code/clean-coders-lecture/2_function/fitness-example-code-diff-pic2/7.png?raw=true)
8. change StringBuffer to String : 가독성 증대를 위해 먼저 buffer를 content로 rename하고 StringBuffer를 String으로 변경하고 컴파일 오류제거, 테스트 확인
  ![8](https://github.com/walbatrossw/TIL/blob/master/06_clean-code/clean-coders-lecture/2_function/fitness-example-code-diff-pic2/8.png?raw=true)
9. extract method - surroundPageWithSetupsAndTearDowns
  ![9](https://github.com/walbatrossw/TIL/blob/master/06_clean-code/clean-coders-lecture/2_function/fitness-example-code-diff-pic2/9.png?raw=true)
10. rename - SetUpTearDownSurrounder, surruond : 의미있는 이름으로 변경
  ![10](https://github.com/walbatrossw/TIL/blob/master/06_clean-code/clean-coders-lecture/2_function/fitness-example-code-diff-pic2/10.png?raw=true)

## 4. 위 예제의 개선의 결과는?
1. 읽기 쉬워졌다.
2. 이해하기도 쉬워졌다.
3. 함수가 자신의 의도를 잘 전달하게 되었다.

## 5. 개선을 할 수 밖에 없었던 이유는?
- 작을수록 좋다!
  - 함수의 첫번째 규칙인 함수는 작아질수 있을 만큼 작아져야 한다.
- 블록이 적을수록 좋다!
  - if, else, while 문장 등의 내부블럭은 한줄이어야하며 괄호가 없어야하고, 함수 호출이어야한다.
- Indenting(들여쓰기)이 적을수록 좋다!
  - 함수는 중첩구조를 갖을 만큼 커서는 안되며 들여쓰기는 한두단계정도가 좋다.

## 6.function should do one thing
- 리펙토링 전의 함수(하나 이상의 일을 수행)
  - creating buffers
  - fetching pages
  - searching for inherited pages
  - rendering paths
  - appending arcane strings
  - generating HTML
- 리팩토링 후의 함수(한가지 단순한 일을 수행)
  - including setups and teradowns into test pages
    * surruond메서드라는 결과물이 3가지 일을 하는 것 아닌가?
      * 아래와 같이 3가지 스텝을 거치지만 더 잘개 쪼갤수 없는 비지니스 로직이기 때문에 한가지 일을 수행한다고 볼 수 있다.
        1. 페이지가 테스트 페이지인지 결정
        2. 테스트 페이지인 경우 setups, teardowns를 include
        3. HTML페이지로 렌더링
- 한가지 일만 하도록 하는 것 ? : 칼같이 딱 나눌순 없다.
  - 원래 코드는 추상화 수준이 다른 많은 단계들을 포함하므로 한가지 이상의 일을 한다는 것이 명확하지만 이를 의미있게 최종적으로 줄이는 것이 상당히 어려운 작업이다.
  - 함수가 하나 이상의 일을 한다고 말할 수 있는 경우는 단순한 구현의 재인용이 아닌 이름으로 함수를 추출할 수 있을 때이다.
- Reading code from top to bottom
  - 함수를 구성하는 원칙
  - 코드를 top-down 이야기체로 읽을 수 있도록 해준다.
  - 현재 함수 바로 밑에 현재 함수 다음의 추상화 수준을 갖는 함수들을 배치시킨다.
  - 이 원칙을 바탕으로 구현하기 위한 방법으로는 to-v 형태로 기능을 나누어본다.

## 7. Where do classes go to hide?
큰 함수는 실제로 클래스가 숨어있는 곳이다. 큰 함수는 변수와 인자들, 들여쓰기가 존재하고, 변수들을 사용해서 통신하는 기능들의 집합들이기 떄문에 항상 하나 이상의 클래스로 분리할 수 있다.
