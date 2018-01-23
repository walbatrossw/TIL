# Clean Coders : Function Part 2

## 8. PrimeNumber 예제
1. Extract Method Object - PrimePrinterHelper
2. remove static
3. extract field
4. extract method - printNumbers
5. change signature printNumbers - add primes, numberOfPrimes as parameters
6. rename parameter primes to numbers
7. change invoke to return prime numbers
8. prepare extract method object
9. prepare extract method object
10. extract method object - NumberPrinter
11. Move inner to upper level - NumberPrinter
12. change numbers, numberOfPrimes as invoke's parameters
13. inline primePrinterHelper.printNumbers
14. rename PrimePrinterHelper to PrimeGenerator
15. prepare parameterize numberOfPrimes
16. parameterize numberOfPrimes
17. extract method - findNextPrime
18. Move inner to upper level - PrimeGenerator
19. rename meaningfully

## 9. One Thing? 한가지 일이란 무엇인가?
- funtion should do one things(함수는 한가지 일만 해야함), do it well(잘해야하고), do it only(오직 한가지만 해야한다)
- 큰 함수를 작은 함수들로 쪼갤 때 흥미로운 일들이 발생한다.
  - 주요 섹션들을 함수로 추출된다.
  - 함수를 서로 다른 추상화 수준으로 분리된다.
    - 만약 함수가 하나 이상의 추상화 수준을 다룬다면 이 함수는 한가지 이상의 일을 하는 것이다.
- 하지만 추상화 수준은 사람마다 기준이 다르고 불분명하다. 그렇다면 어떻게 하면 추상화 수준을 맞추는가????
  - Extract till you drop(포기할 때까지 추출하라)
    1. 함수가 한가지 일만 하는지 어떻게 확신할 수 있는가?
    2. 더이상 extract할 수 없을 때까지 extract하라.
    3. extract할 코드를 가진 함수는 한가지 이상의 일을 하는 것이다.
    4. 4줄 이내의 함수들로 구성된 클래스 : 메서드, 함수들이 증가하게된다. 그렇다면 따로 그룹핑을 하면된다
    5. if, while문 등에서 {}가 보이면 extract할 대상으로 의심해보아라. {}은 extract할 기회이다.

## 10. Conclusion
- 1st rule : funtion should be small
- 2nd rule : smaller than that
- 네이밍을 잘하면 당신뿐이 아니라 모든 사람들의 시간을 절약해준다.
  - 이정표 역할
  - 네비게이션 역할
- 함수를 작게 만들면 모두의 시간을 절약할 수 있다.
- 함수를 여러 클래스에 잘 배분하려면 함수를 작게 만들어야 한다.
