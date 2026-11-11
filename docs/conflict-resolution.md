## 시나리오: 병합과 충돌
### 참여자 : pcochoco
### 상황 1 : non-fast-forward
- 원격 브랜치로 push 시도 
- 원격 브랜치의 변경이 로컬로 동기화되어있지 않아 적용 안됨

### 해결
- ```git fetch origin```
- ```git rebase origin/feat/code-review```
-```git push origin/feat/code-review```

### 상황 2
- main과의 차이로 인한 충돌

### 해결
- ```git rebase origin/main```
-```git status```
-```git add .```
-```git commit```
-```git push origin/main```


#### interactive rebase in progress
Rebase 도중에는 git commit 대신 git rebase --continue를 사용
```git rebase --continue```

### 상황 3
기존 브랜치의 merge 충돌을 해결한 뒤에도 push 불가하다는 오류

```
 ! [rejected]        feat/code-review -> feat/code-review (non-fast-forward)
error: failed to push some refs to ...
hint: Updates were rejected because the tip of your current branch is behind
hint: its remote counterpart. If you want to integrate the remote changes,
hint: use 'git pull' before pushing again.
hint: See the 'Note about fast-forwards' in 'git push --help' for details.
```

### 원인
이미 원격에 Push한 커밋이 있었는데, 그 상태에서 Rebase를 실행하면서 커밋 해시가 새로 생성

### 주의 사항 
이미 Push한 개인 작업 브랜치: Rebase는 가능하지만 원격 이력이 바뀌어 --force-with-lease가 필요할 수 있음 

### 작업 시 주의 
- 이미 원격에 Push한 커밋이 있었는데, 그 상태에서 Rebase를 실행하면서 커밋 해시가 새로 생성
- 아직 Push하지 않은 개인 작업: Rebase를 사용해 최신 main을 반영해도 부담이 적음

### 해결
```git push --force-with-lease origin feat/code-review  ```
- 원격 브랜치의 기존 이력을 새로운 이력으로 교체
- --force-with-lease를 권장하는 이유는 마지막으로 확인한 원격 브랜치 상태가 달라졌다면 Push를 거부

---

## 시나리오: revert 도중 cherry-pick 충돌
### 참여자
- 작성자: machinity
- 상대: islajin

### 상황(What happened)
- 부모 브랜치를 main으로 재설정하기 위해 feature/machinity-* 브랜치들에 대한 revert 진행 도중 cherry-pick 하는 과정에서 다음 명령어 사용시에 공통 작업 파일인 CONTRIBUTING.md에서 충돌이 났던 상황

### 충돌 내용(Conflict markers)
```
<<< HEAD
...
=======
- ### 브랜치 이름을 잘못 만들었을 때

- ```bash
- # 로컬 브랜치 이름 변경
- git branch -m <잘못된-이름> <올바른-이름>

- # 이미 push했다면: 새 이름으로 push 후 원격의 옛 브랜치 삭제
- git push -u origin <올바른-이름>
- git push origin --delete <잘못된-이름>
- ```

- > 열려 있는 PR이 있다면, 원격 브랜치를 삭제하기 전에 GitHub에서 PR의 브랜치를 먼저 확인해야함

---

## 커밋 메시지 컨벤션

형식: `<종류>: <구체적인 변경 내용>`

종류 표시 뒤의 설명은 한국어로 쓰고 `~했음`으로 끝낸다. 메시지만 보고 무엇을 바꿨는지 알 수 있도록 변경 대상을 구체적으로 적는다.

| 종류 | 사용 시점 |
| --- | --- |
| `feat` | 기능 또는 학습 결과물을 추가했을 때 |
| `fix` | 오류를 수정했을 때 |
| `docs` | 문서를 추가하거나 수정했을 때 |
| `refactor` | 동작은 유지하면서 코드 구조를 개선했을 때 |
| `test` | 테스트를 추가하거나 수정했을 때 |
| `chore` | 설정이나 저장소 관리 작업을 했을 때 |

좋은 예:

- `docs: PR 작성 규칙 정리했음`
- `feat: 브랜치 개념 학습 노트 추가했음`
- `fix: 예시 명령어의 오타 수정했음`

금지하는 예: `update`, `fix`, `wip`, `final`, `bug fix`, `edit file`처럼 변경 대상과 내용을 알 수 없는 메시지.

---

## PR 작성 규칙

PR 제목은 변경 내용을 알 수 있도록 한국어로 구체적으로 적는다. PR 본문에는 다음 항목을 모두 포함한다.

1. **연결 Issue:** `Closes #이슈번호` 또는 `Fixes #이슈번호`
2. **변경 사항(What):** 무엇을 변경했는지
3. **변경 이유(Why):** 왜 변경했는지
4. **테스트·검증 방법(How):** 무엇을 확인했고 결과가 어땠는지

수행하지 못한 검증이 있다면 그 이유를 본문에 적는다.

PR을 작성할 때 [PR 본문 템플릿](../.github/PULL_REQUEST_TEMPLATE.md)을 참고한다.
>>> main
```

### 해결 과정(How)
- 선택한 해결 전략: revert, cherry-pick, keep both
- 전략 선택 이유: 
    - revert: 기존 커밋을 지우지 않고, 그 변경을 취소하는 새 커밋을 추가한다. 즉, 히스토리가 그대로 남아 "무엇을 언제, 왜 되돌렸는지" 추적할 수 있다. 이 프로젝트의 목적을 달성하기 위해서는 팀원들이 다같이 '어떤 상황을 마주쳤고, 어떻게 처리했는지' 같은 내용을 추적하며 같이 학습할 필요가 있어 revert를 선택했다.
    - cherry-pick: 각 브랜치가 커밋 1개씩이라 해시 하나만 지정하면 되는 상황이었고, 원하는 커밋만 골라 원하는 위치(main)에 올리는 게 가장 직관적이었다. CONTRIBUTING.md에서 rebase는 사용하지 않기로 결정했고, revert만 사용하는 건 브랜치 구조가 변하지 않아 원하는 바가 아니었으며, 파일을 다시 손으로 작성하는 건 기존 작성 내용에서 누락, 변형될 가능성이 있어 소거법으로 cherry-pick을 선택했다.
    - keep both: 작성한 규칙들이 모두 유지되어야하는 내용이었다.
- 실제로 수행한 명령/절차
```
git switch -C feature/machinity-add-branch-naming-rule main
git cherry-pick 9d248c1

# IDE에서 충돌해결 후 이어서 진행

git log --oneline main..HEAD  
git push --force-with-lease -u origin feature/machinity-add-branch-naming-rule
```

### 결과(Outcome)
- 최종 병합 결과 요약: 삭제될 라인 및 기존 라인 모두 살린 상태로 merge
- 관련 PR/커밋 링크: https://github.com/codyssey2-2-git-collab/codyssey2_2/pull/23

### 배운 점(Learnings)
- 막연히 알고 있던 conflict marker의 의미와 해결방법에 대해 정확히 알게 됨