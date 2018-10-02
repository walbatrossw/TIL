# Big-O 표기법 정리

## 1. O(1) 알고리즘

입력데이터와 상관없이 언제나 일정한 처리 시간이 걸리는 알고리즘을 말한다.

```
F(int[] a) {
  return (n[0] == 0) ? true : false;
}
```
위의 함수를 보면 첫번째 배열 방의 값이 0인지를 확인하고, 인자로 받는 배열 방이 크기에 상관없이 언제나 일정한 속도로 결과를 반환한다. 이러한 경우
이 알고리즘을 O(1)의 시간 복잡도를 가진다라고 표현한다.

![O(1)](https://github.com/walbatrossw/TIL/blob/master/01_cs-basic/data-structure/img/big-o/o(n).png?raw=true)

0(1) 알고리즘을 그래프로 그려보면 가로축이 데이터, 세로축을 처리 시간이라고 할 때 데이터가 증가함에 따 성능에 변함이 없다.

## 2. O(n) 알고리즘

입력데이터와 비례해서 처리시간이 걸리는 알고리즘을 말한다.

```
F(int[] n) {
  for i = 0 to n.length
    print i;
}
```
위의 함수는 n개의 데이터를 입력받으면 n번 루프를 돌게 되고, n이 1개 늘어날 때마다 처리가 비례하게 한번씩 늘어나게 된다. 즉, 다시말하면 n의 크기만큼 처리시간이 증가하게 된다.

![O(n)](https://github.com/walbatrossw/TIL/blob/master/01_cs-basic/data-structure/img/big-o/o(1).png?raw=true)

0(n) 알고리즘을 그래프로 그려보면 데이터가 증가함에 따라 비례해서 처리시간도 같이 증가하는 것을 알 수 있다. 언제나 데이터와 시간이 같은 비율로 증가하기 때문에 그래프는 직선으로 표현된다.

## 3. O(n<sup>2</sup>) 알고리즘

```
F(int[] n) {
  for i = 0 to n.length
    for j = 0 to n.length
      print i + j;
}
```

위의 함수를 보면 n개의 데이터를 입력받고 n번 루프를 돌게 되고, 또 그 안에서 n번의 루프를 다시 돌게 된다.

![O(n^2)](https://github.com/walbatrossw/TIL/blob/master/01_cs-basic/data-structure/img/big-o/o(n%5E2).png?raw=true)

위의 함수를 알기 쉽게 그림으로 나타내면 n개의 데이터를 입력받으면 첫번째 루프에서 n번 돌면서 각각의 엘리먼트에서 n번씩 또 반복을 하게된다. 즉, 처리 횟수가 n을 가로, 세로 길이로 가지는 면적만큼이 된다. n이 1씩 증가할 때 마다 가로, 세로 줄이 증가 하게 되고 데이터가 커지면 커질 수록 처리시간의 부담이 커지게 된다.

![O(n^2)2](https://github.com/walbatrossw/TIL/blob/master/01_cs-basic/data-structure/img/big-o/o(n%5E2)2.png?raw=true)

O(n<sup>2</sup>) 알고리즘을 그래프로 그려보면 처음에는 데이터 증가에 따라 조금씩 상승하다가 나중에는 데이터를 하나 추가할 때마다 점점더 그래프의 기울기가 급격히 증가하는 것을 알 수 있다.

## 4. O(nm) 알고리즘

```
F(int[] n, int[] m) {
  for i = 0 to n.length
    for j = 0 to m.length
      print i + j;
}
```

위의 함수를 보고 O(n<sup>2</sup>) 알고리즘이라고 착각하기 쉬운데 함수를 자세히 들여다보면 입력받은 인자 n, m의 길이만큼 루프를 수행하게 된다.

![O(nm)]()

n은 엄청 크고, m은 엄청 작을 수도 있기 때문에 변수가 다르다면 Big-O 표기법에도 반드시 다르게 표시를 해주어야 한다.

![O(nm)2]()

O(nm) 알고리즘도 O(n<sup>2</sup>) 알고리즘과 마찬가지로 데이터가 증가하면 할수록 그래프가 점점 수직에 가까운 형태를 나타내게 된다.


## 5. O(n<sup>3</sup>) 알고리즘

```
F(int[] n) {
  for i = 0 to n.length
    for j = 0 to n.length
      for k = 0 to n.length
        print i + j + k;
}
```

위의 함수를 보면 입력받은 인자 n만큼 3번의 루프를 돌리게 되면 세제곱이 된다.

![0(n^3)]()

위의 함수를 다시 그림으로 그려보면 위와 같이 표현할 수 있는데 O(n<sup>3</sup>) 알고리즘은 O(n<sup>2</sup>) 알고리즘보다 데이터가 증가함에 따라 더 급격하게 처리시간이 늘어나는 것을 확인 할 수 있다.

## 6. O(2<sup>n</sup>) 알고리즘

O(2<sup>n</sup>) 알고리즘은 O(n<sup>2</sup>) 알고리즘과 혼동하기 쉬운데 둘은 너무나 다른 알고리즘이다.
