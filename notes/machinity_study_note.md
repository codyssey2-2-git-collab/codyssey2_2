# Git 브랜치 & 팀 협업 학습 노트

> 목표: 브랜치의 내부 동작, GitHub Flow, PR/코드 리뷰, 충돌 해결, `reset`/`revert`/`stash`, 문제 해결 기록법을 **스스로 설명**할 수 있게 정리한다.

---

## 0. 먼저 알아야 할 Git의 기본 모델

Git은 파일의 "변경분"이 아니라 **스냅샷(커밋)** 을 쌓는다.

- **커밋(commit)**: 특정 시점 프로젝트 전체의 스냅샷 + 메타데이터(작성자, 시간, 메시지) + **부모 커밋의 해시**
- 커밋은 부모를 가리키며 이어지므로 히스토리는 **DAG(방향성 비순환 그래프)** 가 된다.
- 커밋 해시(SHA)는 내용으로 계산되므로, 커밋은 **한 번 만들어지면 바뀌지 않는다(immutable)**.

```
A ← B ← C        (C의 부모는 B, B의 부모는 A)
```

이 사실 하나가 브랜치, reset, revert를 이해하는 열쇠다.

---

## 1. 브랜치: 커밋을 가리키는 포인터

### 1-1. 내부 동작

브랜치는 **특정 커밋 해시 하나를 담은 아주 가벼운 참조(ref)** 일 뿐이다.

```bash
cat .git/refs/heads/main
# 3f2a9c1e... (커밋 해시 40자 한 줄)
```

- 브랜치 생성 = 파일 하나(41바이트) 만들기 → 그래서 **생성/전환이 거의 공짜**
- 새 커밋을 만들면 **현재 브랜치 포인터가 새 커밋으로 자동 이동**
- **HEAD** = "지금 내가 서 있는 위치". 보통 브랜치를 가리킨다.

```bash
cat .git/HEAD
# ref: refs/heads/main
```

### 1-2. 그림으로 보기

```
                 main
                  ↓
A ← B ← C   (HEAD → main)

$ git switch -c feature
                 main
                  ↓
A ← B ← C
                  ↑
               feature  (HEAD → feature)

$ git commit (D 생성)
                 main
                  ↓
A ← B ← C ← D
              ↑
           feature (HEAD)
```

- `main`은 C에 그대로, `feature`만 D로 이동했다.
- 커밋을 복사한 게 아니라 **포인터만 하나 더 생겼다**는 점이 핵심.

> **Detached HEAD**: `git checkout <해시>`처럼 HEAD가 브랜치가 아닌 커밋을 직접 가리키는 상태. 이때 만든 커밋은 어떤 브랜치도 가리키지 않아 나중에 잃어버리기 쉽다 → 작업하려면 `git switch -c 새브랜치`.

### 1-3. 왜 브랜치를 나눠서 작업하는가

| 이유 | 설명 |
|---|---|
| **격리** | 실험/미완성 코드가 `main`(배포 가능한 코드)을 오염시키지 않는다 |
| **병렬 작업** | 여러 사람이 서로의 작업 중간 상태에 영향받지 않고 동시에 개발 |
| **리뷰 단위** | 브랜치 = 하나의 변경 주제 → PR로 묶어 검토하기 쉽다 |
| **되돌리기 쉬움** | 실패한 실험은 브랜치째 버리면 끝 |
| **추적성** | "이 기능은 어떤 커밋들로 들어왔나"가 명확 |

### 1-4. 자주 쓰는 명령

```bash
git branch                 # 로컬 브랜치 목록
git branch -a              # 원격 포함 전체
git switch -c feat/login   # 생성 + 전환 (= checkout -b)
git switch main            # 전환
git branch -d feat/login   # 병합된 브랜치 삭제
git log --oneline --graph --all   # 브랜치 구조 시각화 ★
```

### 1-5. 병합 방식: fast-forward vs 3-way merge

**Fast-forward**: `main`이 그동안 안 움직였으면 포인터만 앞으로 당긴다 (새 커밋 없음).
```
A ← B ← C ← D        →   main이 C에서 D로 이동
```

**3-way merge**: 양쪽 다 진행됐으면 공통 조상(base)·내 쪽·상대 쪽 세 개를 비교해 **merge commit**(부모 2개)을 만든다.
```
        D ← E   (feature)
       /     \
A ← B ← C ← F ← M   (main, M = merge commit)
```

---

## 2. GitHub Flow

### 2-1. 정의

**`main`은 항상 배포 가능한 상태**로 유지하고, 모든 변경은 **짧게 사는 브랜치 → PR → 리뷰 → 병합** 으로만 들어가는 단순한 브랜치 전략.

### 2-2. 흐름

```
1. main에서 브랜치 생성       git switch -c feat/xxx
2. 작은 단위로 커밋            git commit
3. 원격에 push                 git push -u origin feat/xxx
4. Pull Request 생성          (GitHub에서)
5. 리뷰 & 토론 & 수정 커밋      (CI 테스트 통과 확인)
6. main에 병합                 Merge / Squash / Rebase
7. 배포 → 브랜치 삭제
```

### 2-3. 원칙

- `main`에 **직접 push 금지** (Branch protection rule로 강제)
- 브랜치 이름은 의도가 드러나게: `feat/`, `fix/`, `docs/`, `refactor/`
- 브랜치는 **짧게** 유지 → 오래 살수록 `main`과 멀어져 충돌이 커진다
- 작업 중 `main`이 바뀌면 자주 가져와 동기화 (`git pull origin main` 또는 rebase)

### 2-4. 왜 팀 협업에 전략이 필요한가

전략이 없으면:
- 누가 `main`에 깨진 코드를 올렸는지 모르고, 배포 가능한 버전이 무엇인지 불분명
- 같은 파일을 동시에 고쳐 대형 충돌이 주기적으로 발생
- 리뷰 없이 들어간 코드로 품질·지식 공유가 무너짐

전략은 **"코드가 main에 들어가는 유일한 경로"를 합의**하는 것이다. 경로가 하나면 검증(리뷰·CI)을 그 경로에 걸 수 있다.

### 2-5. 비교: Git Flow

| | GitHub Flow | Git Flow |
|---|---|---|
| 주요 브랜치 | `main` + 기능 브랜치 | `main`, `develop`, `feature`, `release`, `hotfix` |
| 복잡도 | 낮음 | 높음 |
| 적합 | 지속 배포(웹 서비스), 소규모 팀 | 버전 단위 릴리스(패키지, 펌웨어, 앱 스토어) |

---

## 3. Pull Request와 코드 리뷰

### 3-1. PR의 목적

PR은 "내 브랜치를 main에 합쳐 주세요"라는 **요청이자, 병합 전 검토를 위한 토론 공간**이다.

- 변경 내용(diff)을 한눈에 보여줌
- 리뷰 코멘트, 승인(Approve), 변경 요청(Request changes)
- CI(테스트·린트) 결과를 병합 조건으로 연결
- **왜 이렇게 바꿨는지 기록이 영구히 남는다** → 나중에 `git blame` → PR 링크로 맥락 추적

### 3-2. 좋은 PR 작성법

```markdown
## 개요
로그인 실패 시 에러 메시지가 표시되지 않던 문제 수정

## 변경 사항
- AuthService에서 401 응답 처리 추가
- 에러 토스트 컴포넌트 연결

## 테스트 방법
1. 잘못된 비밀번호로 로그인
2. "비밀번호가 올바르지 않습니다" 토스트 확인

## 관련 이슈
Closes #42
```

- **작게**: 리뷰어가 집중할 수 있는 크기 (대략 수백 줄 이하)
- **하나의 주제만**: 기능 추가 + 리팩터링 + 포매팅 섞지 않기
- 셀프 리뷰 먼저: 올리기 전에 diff를 직접 한 번 읽기

### 3-3. 코드 리뷰의 가치

| 가치 | 설명 |
|---|---|
| 결함 조기 발견 | 버그·엣지 케이스를 배포 전에 잡는다 (뒤로 갈수록 수정 비용 증가) |
| 지식 공유 | 최소 두 명이 그 코드를 이해 → 특정인 의존(버스 팩터) 감소 |
| 일관성 | 컨벤션·설계 방향 유지 |
| 학습 | 리뷰를 주고받으며 서로의 기법을 배움 |
| 기록 | 설계 결정의 근거가 남음 |

**리뷰 매너**: 사람이 아닌 코드에 대해, 이유와 함께, 제안형으로.
> ❌ "이거 틀렸네요" → ✅ "여기서 `user`가 null이면 NPE가 날 것 같은데, 가드를 넣는 건 어떨까요?"

### 3-4. 병합 방식

| 방식 | 결과 | 특징 |
|---|---|---|
| Create a merge commit | merge commit 생성 | 브랜치 히스토리 그대로 보존 |
| Squash and merge | 커밋 1개로 압축 | main 히스토리가 깔끔 (PR 1개 = 커밋 1개) |
| Rebase and merge | 커밋들을 main 위에 재배치 | 선형 히스토리, merge commit 없음 |

---

## 4. 충돌(Conflict)

### 4-1. 왜 발생하는가

3-way merge에서 Git은 **base(공통 조상)** 대비 양쪽 변경을 비교한다.

- 한쪽만 바꾼 부분 → 자동으로 그쪽을 채택
- **양쪽이 같은 위치(같은 줄 근처)를 서로 다르게 바꿈** → Git이 어느 쪽이 맞는지 판단 불가 → **충돌**

그 밖의 경우: 한쪽은 파일을 수정, 다른 쪽은 삭제 (modify/delete conflict) 등.

> 충돌은 에러가 아니라 **"사람이 결정해야 하는 지점"** 이라는 신호다.

### 4-2. 충돌 마커의 의미

```
<<<<<<< HEAD
const timeout = 3000;
=======
const timeout = 5000;
>>>>>>> feat/slow-network
```

| 마커 | 의미 |
|---|---|
| `<<<<<<< HEAD` | 여기부터 **현재 브랜치(내 쪽, ours)** 의 내용 |
| `=======` | 양쪽 내용의 **구분선** |
| `>>>>>>> feat/slow-network` | 여기까지 **병합해 들어오는 쪽(theirs)** 의 내용, 뒤에 브랜치명/커밋 |

> 참고: `git config merge.conflictstyle diff3` (또는 `zdiff3`)을 켜면 `|||||||` 아래에 **base(원래 내용)** 도 보여줘서 "둘이 원래 무엇을 바꾸려 했는지" 판단이 쉬워진다.

> ⚠️ rebase 중에는 ours/theirs가 직관과 반대다. rebase는 "상대 브랜치 위에 내 커밋을 하나씩 다시 얹는" 과정이라 **HEAD = 상대(upstream) 쪽**, 들어오는 쪽 = 내 커밋.

### 4-3. 해결 절차

```bash
git merge feat/slow-network
# CONFLICT (content): Merge conflict in config.js

git status                  # 충돌 파일 확인 (both modified)
# 1) 파일을 열어 마커 3종을 모두 지우고 최종 내용으로 편집
#    - 한쪽 선택, 양쪽 합치기, 또는 새로 작성
# 2) 빌드/테스트로 동작 확인
git add config.js           # "해결 완료" 표시
git commit                  # merge commit 생성 (rebase면 git rebase --continue)

# 포기하고 병합 전으로 돌아가기
git merge --abort
```

한쪽을 통째로 채택:
```bash
git checkout --ours   config.js
git checkout --theirs config.js
```

### 4-4. 충돌 줄이는 습관

- 브랜치를 짧게, PR을 작게
- `main`을 자주 내 브랜치로 가져오기
- 포매터를 팀 전체가 동일하게 (공백/줄바꿈 충돌 방지)
- 같은 파일을 크게 손댈 땐 팀에 미리 공유

---

## 5. `reset` vs `revert` vs `stash`

### 5-1. 한 줄 요약

| 명령 | 하는 일 | 히스토리 | 공유된 커밋에 써도 되나 |
|---|---|---|---|
| `reset` | **브랜치 포인터를 과거로 옮김** | 커밋이 사라진 것처럼 **다시 씀** | ❌ (push 후엔 위험) |
| `revert` | 기존 커밋을 **취소하는 새 커밋**을 추가 | 기록 **보존**, 앞으로만 진행 | ✅ |
| `stash` | 커밋 안 한 변경을 **임시 보관함**에 치워둠 | 히스토리와 무관 | (해당 없음) |

### 5-2. `git reset` — 포인터를 되감기

```
A ← B ← C ← D  (main)
$ git reset --hard B
A ← B          (main)     C, D는 브랜치에서 떨어져 나감
```

옵션에 따라 **어디까지 되돌리는지**가 다르다:

| 옵션 | 브랜치 포인터 | Staging(Index) | 작업 디렉터리 | 용도 |
|---|---|---|---|---|
| `--soft` | 이동 | 유지 | 유지 | 커밋만 취소, 변경은 staged로 남김 → 커밋 합치기/메시지 다시 쓰기 |
| `--mixed` (기본) | 이동 | 초기화 | 유지 | 커밋+add 취소, 파일 변경은 남김 |
| `--hard` | 이동 | 초기화 | **초기화** | 모든 변경 폐기 ⚠️ |

```bash
git reset --soft HEAD~1    # 직전 커밋 취소, 변경은 staged 상태로
git reset HEAD~1           # 직전 커밋 취소, 변경은 unstaged로
git reset --hard HEAD~1    # 직전 커밋과 변경 모두 버림
git restore --staged a.js  # (참고) 특정 파일 add만 취소
```

**언제**: 아직 **push하지 않은 로컬 커밋**을 정리할 때.
**왜 공유 후엔 금지?** 동료는 이미 C, D를 기반으로 작업 중인데 내가 히스토리를 지우고 `push --force` 하면 동료의 히스토리와 어긋나 혼란·작업 유실이 생긴다.

> 💡 실수로 `reset --hard` 했다면: `git reflog`에 HEAD가 지나간 기록이 남아 있으므로 `git reset --hard <reflog의 해시>`로 복구 가능 (일정 기간 내).
> 단, **커밋하지 않은 변경**은 reflog에도 없으니 복구 불가.

### 5-3. `git revert` — 취소 커밋을 추가

```
A ← B ← C ← D   (main)
$ git revert C
A ← B ← C ← D ← C'   (C' = C의 변경을 정반대로 적용한 새 커밋)
```

```bash
git revert <commit>          # 해당 커밋을 되돌리는 커밋 생성
git revert HEAD              # 직전 커밋 되돌리기
git revert -m 1 <merge커밋>   # merge commit 되돌리기 (1 = 유지할 부모, 보통 main 쪽)
```

**언제**: **이미 push되어 공유된** 커밋을 되돌릴 때. 운영 장애 시 문제 커밋을 빠르게 무효화할 때.
**장점**: 히스토리를 다시 쓰지 않으니 다른 사람에게 안전하고, "언제 무엇을 왜 되돌렸는지"가 기록에 남는다.

### 5-4. `git stash` — 작업 중인 변경을 잠시 치워두기

상황: 기능 작업 중인데 급한 버그 수정 요청이 옴. 아직 커밋하기엔 어중간한 상태.

```bash
git stash push -m "로그인 폼 작업 중"   # 변경 보관, 작업 디렉터리 깨끗해짐
git switch main
git switch -c fix/urgent
# ...버그 수정, 커밋, PR...
git switch feat/login
git stash list                         # stash@{0}: On feat/login: 로그인 폼 작업 중
git stash pop                          # 꺼내서 적용 + 목록에서 삭제
# git stash apply                      # 적용만 하고 목록엔 남김
# git stash -u                         # untracked(새 파일)까지 포함
# git stash drop stash@{0}             # 버리기
```

**언제**: 브랜치 전환, `pull` 전에 로컬 변경을 잠시 비워야 할 때.
**주의**: stash는 오래 두면 잊힌다. 장기 보관이 필요하면 WIP 커밋을 만드는 편이 안전하다.

### 5-5. 상황별 선택 가이드

```
되돌리고 싶은 게 커밋인가?
├─ 아니오 (커밋 안 한 변경) ─────────────── 잠시 치우기 → stash
│                                         아예 버리기 → git restore <file>
└─ 예
   ├─ 이미 push해서 다른 사람이 볼 수 있다 ─→ revert
   └─ 아직 내 로컬에만 있다 ─────────────→ reset (--soft / --mixed / --hard)
```

---

## 6. 문제 해결을 "기록하고 재현 가능하게" 남기기

### 6-1. 왜 기록하는가

- 같은 문제가 다시 생겼을 때 **검색해서 바로 해결**
- 다른 팀원이 **같은 절차를 그대로 따라** 해결 가능 (재현성)
- "왜 이 커밋/revert가 있는가"에 대한 근거 보존
- 사후 분석(회고)으로 **재발 방지책** 도출

### 6-2. 재현 가능한 기록의 요소

1. **상황(Context)**: 어느 브랜치에서, 무엇을 하려다
2. **증상(Symptom)**: 실제 에러 메시지·출력 원문
3. **확인한 상태**: `git status`, `git log --oneline --graph` 출력
4. **원인(Root cause)**: 왜 발생했는가
5. **해결 절차**: 실행한 **명령어를 순서대로** 그대로
6. **검증**: 해결됐음을 어떻게 확인했는가
7. **재발 방지**: 프로세스/설정 개선점

### 6-3. 기록 템플릿

```markdown
## [트러블슈팅] feat/login → main 병합 시 config.js 충돌

- 날짜: 2026-11-09
- 작업자: 
- 관련 PR/이슈: #42

### 상황
feat/login 브랜치를 main에 병합하려고 `git merge main` 실행

### 증상
CONFLICT (content): Merge conflict in config.js
Automatic merge failed; fix conflicts and then commit the result.

### 상태 확인
$ git status
both modified: config.js

$ git log --oneline --graph --all
(출력 첨부)

### 원인
main에서 timeout을 5000으로 변경(#38), 내 브랜치에서 3000으로 변경.
같은 줄을 양쪽이 다르게 수정해 자동 병합 불가.

### 해결 절차
1. config.js 열어 충돌 마커 확인
2. #38 작성자와 논의 → 느린 네트워크 대응이 목적이므로 5000 채택
3. 마커 제거 후 저장
4. `npm test` 통과 확인
5. `git add config.js && git commit`

### 검증
- 로컬 테스트 통과, CI 통과
- 로그인 타임아웃 동작 수동 확인

### 재발 방지
- 설정값은 환경변수로 분리해 코드 충돌 지점 제거
- 공용 설정 변경 시 팀 채널에 공지
```

### 6-4. 시나리오 예시: 실수 커밋

**A. 아직 push 안 함 — 비밀키를 커밋해버림**
```bash
git reset --soft HEAD~1      # 커밋 취소, 변경은 유지
git restore --staged .env    # .env만 staging에서 제외
echo ".env" >> .gitignore
git commit -m "feat: API 클라이언트 추가"
```

**B. 이미 push됨 — 버그 있는 커밋이 main에 들어감**
```bash
git log --oneline            # 문제 커밋 해시 확인: a1b2c3d
git revert a1b2c3d           # 취소 커밋 생성
git push                     # 기록이 남는 안전한 되돌림
```
> ⚠️ 비밀키가 **push된** 경우 revert로는 히스토리에 키가 그대로 남는다. **키를 즉시 폐기·재발급**하는 것이 먼저이고, 히스토리 정리(`git filter-repo` 등)는 그 다음 문제다.

**C. 잘못된 브랜치(main)에서 작업하고 있었음 (커밋 전)**
```bash
git stash
git switch -c feat/correct-branch
git stash pop
```

**D. 잘못된 브랜치(main)에 커밋까지 함 (push 전)**
```bash
git branch feat/correct-branch   # 현재 커밋을 가리키는 새 브랜치 생성
git reset --hard HEAD~1          # main을 한 칸 되돌림 (커밋은 새 브랜치에 남아 있음)
git switch feat/correct-branch
```
→ 브랜치가 "포인터"라는 개념을 이해하면 이 해결법이 자연스럽게 나온다.

---

## 7. 자가 점검 질문

- [ ] 브랜치를 만들면 커밋이 복사되는가? (→ 아니다. 포인터 하나가 추가될 뿐)
- [ ] HEAD와 브랜치의 관계를 설명할 수 있는가?
- [ ] fast-forward와 3-way merge는 언제 각각 일어나는가?
- [ ] GitHub Flow에서 `main`의 역할은? 왜 직접 push를 막는가?
- [ ] PR이 단순한 "병합 버튼"이 아니라는 근거 3가지는?
- [ ] 충돌이 나는 정확한 조건은? `=======` 위와 아래는 각각 누구 것인가?
- [ ] push한 커밋을 되돌릴 때 `reset`이 아니라 `revert`를 쓰는 이유는?
- [ ] `reset --soft / --mixed / --hard`의 차이를 표 없이 말할 수 있는가?
- [ ] `reset --hard`로 날린 커밋을 복구하는 방법은?
- [ ] 트러블슈팅 기록에 반드시 들어가야 할 요소는?

---

## 8. 명령어 치트시트

```bash
# 브랜치
git switch -c <name>            git branch -d <name>
git log --oneline --graph --all

# 병합 / 충돌
git merge <branch>              git merge --abort
git checkout --ours|--theirs <file>
git config --global merge.conflictstyle zdiff3

# 되돌리기
git reset --soft|--mixed|--hard <commit>
git revert <commit>             git revert -m 1 <merge-commit>
git reflog

# 임시 보관
git stash push -m "msg"         git stash -u
git stash list                  git stash pop | apply | drop

# 원격
git push -u origin <branch>     git pull origin main
```