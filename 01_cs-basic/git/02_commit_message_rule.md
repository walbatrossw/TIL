# Git - Commit Message Convention

커밋 메시지를 작성할 때는 원칙을 정하고 일관성 있게 작성해야 한다. 아래는 유다시티의 커밋 메시지 스타일
가이드를 참조한 내용이다.

## 1. Commit Message

기본적으로 커밋 메시지는 아래와 같이 제목/본문/꼬리말로 구성한다.

```
type : subject

body

footer
```

## 2. Commit Type
- feat : 새로운 기능 추가
- fix : 버그 수정
- docs : 문서 수정
- style : 코드 포맷팅, 세미콜론 누락, 코드 변경이 없는 경우
- refactor : 코드 리펙토링
- test : 테스트 코드, 리펙토링 테스트 코드 추가
- chore : 빌드 업무 수정, 패키지 매니저 수정

## 3. Subject

- 제목은 50자를 넘기지 않고, 대문자로 작성하고 마침표를 붙이지 않는다.
- 과거시제를 사용하지 않고 명령어로 작성한다.
  - "Fixed" --> "Fix"
  - "Added" --> "Add"

## 4. Body

- 선택사항이기 때문에 모든 커밋에 본문내용을 작성할 필요는 없다.
- 부연설명이 필요하거나 커밋의 이유를 설명할 경우 작성해준다.
- 72자를 넘기지 않고 제목과 구분되기 위해 한칸을 띄워 작성한다.

## 5. footer

- 선택사항이기 때문에 모든 커밋에 꼬리말을 작성할 필요는 없다.
- issue tracker id를 작성할 때 사용한다.

## 6. Example

```
feat: Summarize changes in around 50 characters or less

More detailed explanatory text, if necessary. Wrap it to about 72
characters or so. In some contexts, the first line is treated as the
subject of the commit and the rest of the text as the body. The
blank line separating the summary from the body is critical (unless
you omit the body entirely); various tools like `log`, `shortlog`
and `rebase` can get confused if you run the two together.

Explain the problem that this commit is solving. Focus on why you
are making this change as opposed to how (the code explains that).
Are there side effects or other unintuitive consequenses of this
change? Here's the place to explain them.

Further paragraphs come after blank lines.

 - Bullet points are okay, too

 - Typically a hyphen or asterisk is used for the bullet, preceded
   by a single space, with blank lines in between, but conventions
   vary here

If you use an issue tracker, put references to them at the bottom,
like this:

Resolves: #123
See also: #456, #789
```

## 7. My Thought / Conclusion / Summery

![before-commit-message](https://github.com/walbatrossw/TIL/blob/master/01_cs-basic/git/img/before-commit-message.png?raw=true)

위와 같이 그동안 커밋을 하면서 메시지에는 몇번째 커밋인지 그리고 뭘 했는지에 대해서만 간략하게 작성했다.
나 나름의 규칙이라고 생각해서 그렇게 작성했던 것 같다. 하지만 커밋 메시지에도 나름의 규칙과 일관성을 가져야
한다는 것을 알면서도 제대로 실처하지 못했다. 그래서 개발을 하고나서 문제점이 발생하고 어떻게 해결했는지를
찾으려면 좀처첨 헷갈려서 찾을 수가 없었던 적이 많았다. 물론 문제점을 해결한 상황을 따로 정리하지 않은 것이
가장 큰 문제이기도 하다.

앞으로는 위의 규칙을 지키면서 커밋 메시지를 작성하는 노력을 기울여 보자.

## 8. References

- [깃허브로 취업하기 포스팅](http://sujinlee.me/professional-github/)
- [유다시티 커밋 메시지 스타일 가이드](https://udacity.github.io/git-styleguide/)
