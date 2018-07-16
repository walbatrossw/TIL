이글은 생활코딩 - GIT (Gui)을 참고로 작성하였습니다.

## 이전 상태로 되돌리기

git을 통해 작업 내용을 Commit을 하다보면 이전의 상태로 되돌리고 싶을 경우가 있다. 이전 상태로 되돌리는데는 여러 방법이 있다.

- Discard - Commit을 수행하기 전에 수정사항을 취소
- Reset to Commit - Commit을 수행한 후 수정사항을 취소
    - Hard : 되돌리려는 시점 이후의 모든 Commit 삭제, 현재 작업상태(WorkingCopy)까지 삭제
    - Mixed : 되돌리려는 시점 이후의 모든 Commit 삭제, 현재 작업상태(WorkingCopy)는 유지
- Revert(Reverse Commit) - 현재 Commit 이전의 상태로 되돌리기, 현재 Commit한 작업은 유지되고, 이전 상태로 되돌려 새로운 Commit을 생성
    - 예를 들어 5번 commit을 했다고 가정해보자. 그런데 2번째 Commit의 작업상태로로 되돌리고, 기존의 Commit을 유지시키고 싶다면?
        - 5번째 commit에서 reverse commit 수행 (4번째 commit 상태로 되돌리기)
        - 4번째 commit에서 reverse commit 수행 (3번째 commit 상태로 되돌리기)
        - 3번째 commit에서 reverse commit 수행 (2번째 commit 상태로 되돌리기)

    - 결과 : 저장소에는 총 8번의 Commit이 생기게된다.
        - 1st commit -> 2nd commit -> 3th commit -> 4th commit -> 5th commit -> 6th commit(4th) -> 7th commit(3th) -> 8th commit(2th)
    - 주의 : reverse commit을 순차적으로 진행하지 않으면 conflict가 발생하므로 주의


## branch

보통 안정적인 작업(버그수정, 개선작업)과 불안정한 작업(실험적인 작업 및 수정)을 동시에 진행해야할 경우 사용한다.
일반적인 작업과 불안정한 작업이 혼재되어 있을 경우, 추후에 잠재적인 위험이 존재하기 때문
마치 동일한 프로젝트 폴더를 2개를 각각 따로 작업하는 것과 비슷한 형태라고 생각하면 이해가 쉽다.

- branch 생성
    - master branch가 원본이라고 생각하면 된다.(안정적인 작업 수행)
    - 새로 생성한 branch는 복제된 프로젝트라고 생각하면된다. (실험적인 작업 또는 수정을 수행)

- branch 병합 - mster branch와 새로 생성한 branch를 합치는 과정
    1. master branch로 checkout(master branch를 선택)
    2. merge 새로생성한 branch명 into currnet branch(master branch로 새로생성한 branch를 합치는과정)
    3. 자동으로 병합된 commit이 새로 생성

- 충돌 해결
    - 항상 아름답고 자연스럽게 merge가 되는 경우는 거의 없다.
    - 각각의 branch에서 수정한 내용이 중복되는 위치에 있을 경우에는 merge하는 과정에서 git은 Conflict라는 경고가 발생하고, Commit을 수행하지 않는다.
    - 이 경우에는 작업자가 직접 중복되는 위치의 코드를 수정을 할 수도 있고, git이 처리 할 수도 있다.
        - Resolve Using ‘Mine’ : checkout한 branch의 코드를 사용
        - Resolve Using ‘Their’ : 병합되는 branch의 코드를 사용
        - Mark Resolved : 직접 수정한 뒤에 Commit할 경우
        - 충돌 위치의 코드 설명
            - HEAD : 현재 checkout한 branch의 변경사항
            - === : 서로 중복되는 코드 사이를 구분
            - 병합되는 branch명 : 병합되는 branch의 변경사항

- 충돌의 최소화
    - 충돌을 최소화 하기 위해서는 새로 생성된 branch는 항상 작업을 하기전에 master branch와 merge를 한 뒤에 작업을 수행해주는 것이 좋다.
    - merge가 되지 않은채로 장기간 작업을 수행하다가 한번에 merge를 하게 될 경우 충돌을 해결하기가 상당히 어려워지기 때문
    - 새로 생성된 branch는 되도록이면 master branch의 변경사항을 작업할 때 마다 최신화해준 뒤 작업을 수행해야 충돌을 최소화할 수 있다.

## 마무리
그동안 git의 기능을 제대로 알고 사용하지 못해서 git이 강력한 도구라는 것을 체감하지 못했다. 단순히 이클립스나 STS에서 Commit, push, pull만 했지 이러한 기능이 있다는 것을 몰랐기 때문이다. 그래서 항상 충돌이 나거나 문제가 발생하면 로컬저장소에 있는 코드를 지우고 새로 pull해왔었는데 앞으로는 문제가 발생하면 위에 정리한 기능을 이용하여 해결해 봐야겠다.