# Commit Rule

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
  - ex) "Fixed", "Fix" "Added", "Add"

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

## 7. References

- [깃허브로 취업하기 포스팅](http://sujinlee.me/professional-github/)
- [유다시티 커밋 메시지 스타일 가이드](https://udacity.github.io/git-styleguide/)
