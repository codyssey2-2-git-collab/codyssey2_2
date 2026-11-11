# Troubleshooting Log

## 시나리오: amend
### 참여자
- machinity

### 상황
- feature/machinity-add-branch-naming-rule 로컬 브랜치 생성 및 첫 commit, push 이후 PR 생성 전 추가 수정이 필요했지만 사소한 내용이라 새로운 commit을 생성하긴 애매했던 상황

### 시도한 명령/절차
```
# 로컬 브랜치 첫 commit, push
git add .
git commit -m "docs: CONTRIBUTING.md 브랜치 네이밍 규칙 추가"
git push -u origin feature/machinity-add-branch-naming-rule

# amend로 추가 수정사항 반영
git add CONTRIBUTING.md
git status
git commit --amend
git push origin feature/machinity-add-branch-naming-rule --force
```

### 결과
```
# 첫 commit 이후 log
% git log
commit 8e21f4119cdc70643defe073ef518c31b87a8ff9 (HEAD -> feature/machinity-add-branch-naming-rule, origin/feature/machinity-add-branch-naming-rule)
Author: Machinity <Machinity@gmail.com>
Date:   Mon Nov 9 01:27:56 2026 +0900

    docs: CONTRIBUTING.md 브랜치 네이밍 규칙 추가

commit df3d104588717aeddfe1b7c1fca3117465aa7385
Author: Isla <Islajiin@gmail.com>
Date:   Sat Nov 7 14:47:07 2026 +0900

    chore: 협업 저장소 기본 구조 구성했음
docs: CONTRIBUTING.md 브랜치 네이밍 규칙 추가

# ------------------------------------------------

# amend 수정사항 반영 이후 log
% git log
commit 9d248c189b8a009593d5688b39ccb7b409283939 (HEAD -> feature/machinity-add-branch-naming-rule, origin/feature/machinity-add-branch-naming-rule)
Author: Machinity <Machinity@gmail.com>
Date:   Mon Nov 9 01:27:56 2026 +0900

    docs: CONTRIBUTING.md 브랜치 네이밍 규칙 추가

commit df3d104588717aeddfe1b7c1fca3117465aa7385
Author: Isla <Islajiin@gmail.com>
Date:   Sat Nov 7 14:47:07 2026 +0900

    chore: 협업 저장소 기본 구조 구성했음
docs: CONTRIBUTING.md 브랜치 네이밍 규칙 추가
```
- 추가 commit 없이 PR 생성시 'file changed'에서 추가 수정사항이 잘 반영됨
- 주의할 점:
  - 이미 다른 사람과 공용으로 사용하는 브랜치일 때는 force push로 인해 원격 commit이 사라질 수 있기 때문에 꼬임과 충돌을 유발하므로 사용 금지
  - 이미 코드 리뷰가 끝나서 PR 승인까지 완료된 상태라면 force push하는 순간 리뷰어들의 승인 상태가 초기화되거나 코드 변경사항을 추적하기 어려워져 동료들을 번거롭게 할 수 있으므로 사용 금지

### 왜 이 방법을 선택했는가(Why)
- 사소한 수정사항 때문에 무의미한 commit이 추가 생성되는 것을 방지할 수 있음
- 혼자 사용하는 브랜치였고, PR이 생성되기 이전이었기 때문에 사용하기 적합한 방법이었음

---

## 시나리오: git reset --soft HEAD~1

### 참여자
- 길연진(Islajin): PR #12 리뷰 반영 및 해결 과정 기록

### 상황
- [PR #12](https://github.com/codyssey2-2-git-collab/codyssey2_2/pull/12)에서 병합 후 최신 `main`을 기준으로 다음 브랜치를 시작하는 단계를 추가해 달라는 리뷰를 받았다.
- 노트에 단계를 추가하고 로컬 커밋 `41d040f`를 만들었지만, 원격에 올리기 전에 문구를 더 정확하게 다듬기로 했다.
- 파일 수정은 유지하면서 방금 만든 **로컬 커밋만** 취소할 필요가 있었다.

### 시도한 명령/절차
```bash
git switch feature/islajin-learning-note
git add notes/islajin-github-flow.md
git commit -m "docs: GitHub Flow 다음 작업 흐름 추가"
git log -1 --format='%h %s'

git reset --soft HEAD~1
git log -1 --format='%h %s'
git status --short
git diff --cached -- notes/islajin-github-flow.md
```

### 결과
```text
# reset 전 HEAD
41d040f docs: GitHub Flow 다음 작업 흐름 추가

# reset 후 HEAD
8309099 docs: 길연진 Github Flow 학습 노트 추가

# reset 후 git status --short
M  notes/islajin-github-flow.md
```
- `git diff --cached`에서 추가한 6번 단계가 남아 있는 것을 확인했다. `M `은 변경 내용이 스테이징되어 있음을 뜻한다.
- 이후 문구를 보완하여 `55e3242`(`docs: GitHub Flow 다음 작업 흐름 보완`)로 다시 커밋하고 PR #12에 push했다. 취소한 `41d040f`는 원격에 올리지 않았다.

### 왜 이 방법을 선택했는가(Why)
- 아직 공유하지 않은 최신 로컬 커밋을 정리하면서 수정 내용을 그대로 유지할 수 있기 때문이다.
- `--soft`는 HEAD만 이전 커밋으로 옮기고 인덱스의 변경 내용은 남긴다.

### 주의할 점
- 이미 공유된 `main`이나 팀원이 함께 쓰는 브랜치의 커밋을 이 방법으로 재작성하지 않는다.
- 원격에 공개된 커밋을 취소할 때는 협업 영향을 확인하고 `git revert` 같은 방법을 검토한다.

---

## 시나리오: git stash / git stash pop

### 참여자
- 길연진(Islajin): PR #12와 PR #2 작업 전환 및 해결 과정 기록

### 상황
- PR #12의 노트 수정 내용이 스테이징된 상태에서 [PR #2](https://github.com/codyssey2-2-git-collab/codyssey2_2/pull/2)의 리뷰 요청과 문서 충돌을 처리해야 했다.
- 미완료 수정 내용을 커밋하지 않고 보관한 뒤 다른 브랜치로 이동했다가 복구했다.

### 시도한 명령/절차
```bash
# PR #12 브랜치에서 작업 보관
git status --short
git stash push -m 'PR #12 리뷰 반영 작업 임시 보관' -- notes/islajin-github-flow.md
git stash list -1
git status --short

# PR #2 브랜치에서 작업한 뒤 돌아오기
git switch feature/islajin-contribution-rules
git switch feature/islajin-learning-note

# 보관한 변경 내용 복구
git stash pop
git status --short
git diff -- notes/islajin-github-flow.md
```

### 결과
```text
# 보관 전 git status --short
M  notes/islajin-github-flow.md

# 보관 직후 git stash list -1
stash@{0}: On feature/islajin-learning-note: PR #12 리뷰 반영 작업 임시 보관

# 보관 직후 작업 트리: 변경 없음

# git stash pop 후 git status --short
 M notes/islajin-github-flow.md
```
- `git diff`에서 추가했던 6번 단계가 복구된 것을 확인했다. `stash pop` 후 해당 stash 항목은 제거되었다.
- 복구된 변경은 스테이징되지 않은 상태여서 다시 `git add`한 뒤 커밋했다.

### 왜 이 방법을 선택했는가(Why)
- PR #12의 미완료 수정 내용을 안전하게 보관하고 PR #2 브랜치로 전환하기 위해서다.
- 불필요한 임시 커밋 없이 원래 브랜치에서 작업을 이어갈 수 있었다.

### 주의할 점
- `stash pop` 후에는 `git status`와 `git diff`로 변경 내용이 제대로 돌아왔는지 확인한다. 복구 대상 파일이 그 사이 변경됐다면 충돌이 생길 수 있다.
- `stash pop`은 기본적으로 원래의 스테이징 상태까지 복원하지 않을 수 있으므로, 커밋 전에 필요한 파일을 다시 `git add`한다.

---

## 시나리오: revert
### 참여자
- machinity

### 상황
- feature/machinity-* 브랜치들 생성시 부모 브랜치의 연관/하위 작업이 아니었음에도 브랜치들을 꼬리 물기로 생성하여, 형상관리에 문제가 있다고 판단. 따라서, 해당 브랜치들의 부모 브랜치를 main으로 변경해야했던 상황

### 시도한 명령/절차

#### 1단계: 최신화 + 백업
```
git switch main
git fetch --all --prune
git pull

# 혹시 모를 사고 대비용 백업 태그 (로컬에만)
git tag backup/main-before-revert 92095aa
git tag backup/study-note d5da556
```
#### 2단계: merge revert 커밋 만들기

최신 merge부터 역순으로 되돌립니다.
```
git switch -c feature/machinity-revert-merges

git revert -m 1 92095aa   # PR #13 되돌리기
git revert -m 1 06d1312   # PR #6 되돌리기
```
-m 1은 “merge 커밋의 1번 부모(main 쪽)를 기준으로 되돌려라”라는 뜻입니다. merge 커밋을 revert할 때는 반드시 필요합니다.
에디터가 열리면 커밋 메시지를 팀 컨벤션에 맞게 수정하거나 그대로 저장하면 됩니다.

검증: 두 PR을 되돌렸으니 파일 상태가 PR #4 직후(7218ec7)와 같아야 합니다.
```
git diff 7218ec7 HEAD     # 아무것도 안 나오면 정상
```
#### 3단계: revert PR 올리고 merge
bash
git push -u origin feature/machinity-revert-merges

GitHub에서 base를 main으로 지정해 PR을 만들고 merge합니다. 그다음 로컬을 갱신합니다.

```
git switch main
git pull
```

참고: GitHub의 merge된 PR 페이지에 있는 Revert 버튼으로도 같은 일을 할 수 있습니다. 다만 PR이 두 개로 나뉘고 직접 검증하기 어려워서 CLI 방식을 추천합니다.

#### 4단계: 각 브랜치를 main 기준으로 재작성

같은 패턴을 4번 반복합니다. switch -C는 "기존 브랜치가 있어도 지정한 지점에서 새로 만든다"는 뜻입니다.

```
# ① branch-naming-rule
git switch -C feature/machinity-add-branch-naming-rule main
git cherry-pick 9d248c1
git log --oneline main..HEAD          # 커밋 1개만 보이면 정상
git push --force-with-lease -u origin feature/machinity-add-branch-naming-rule

# ② troubleshooting-log-01-amend
git switch -C feature/machinity-add-troubleshooting-log-01-amend main
git cherry-pick b8fd11b
git log --oneline main..HEAD
git push --force-with-lease -u origin feature/machinity-add-troubleshooting-log-01-amend

# ③ readme-tableofcontents
git switch -C feature/machinity-update-readme-tableofcontents main
git cherry-pick d394b16
git log --oneline main..HEAD
git push --force-with-lease -u origin feature/machinity-update-readme-tableofcontents

# ④ study-note
git switch -C feature/machinity-add-study-note main
git cherry-pick d5da556
git log --oneline main..HEAD
git push --force-with-lease -u origin feature/machinity-add-study-note
```
force push가 필요한 이유는 원격 브랜치에 옛 커밋이 남아 있기 때문입니다. --force-with-lease는 그사이 누가 원격에 푸시했다면 거부해 주는 안전한 force입니다.
cherry-pick 충돌이 나면 해당 파일을 열어 <<<<<<< 구간을 정리한 뒤 아래를 실행합니다.
bash
  git add <파일>
  git cherry-pick --continue

꼬였으면 git cherry-pick --abort 후 다시 시도하면 됩니다. 파일이 서로 달라서 충돌 가능성은 낮지만, README 목차가 다른 문서를 참조하면 날 수 있습니다.

#### 5단계: PR 4개 다시 올리기
각 브랜치마다 base를 main으로 지정해 PR을 생성합니다. 기존 PR들은 이미 merge·종료된 상태라 새 PR이 필요합니다.
PR #4에서 추가한 PR 템플릿이 있으니 본문에 맞춰 작성합니다.
하나를 merge한 뒤 다음 PR에 “branch is out of date”가 뜨면 Update branch를 누르거나, 로컬에서 git rebase main 후 --force-with-lease 푸시를 하면 됩니다.

#### 6단계: 확인 및 정리
```
git switch main && git pull
git log --oneline --graph --all
```

각 machinity 브랜치가 main에서 한 칸씩 뻗어 나와 merge되는 모양이면 성공입니다. 마지막으로 백업 태그를 정리합니다.

```
git tag -d backup/main-before-revert backup/study-note
```

### 결과

revert 전 그래프
```
% git log --oneline --graph --all
...
| | * d394b16 (origin/feature/machinity-update-readme-tableofcontents, feature/machinity-update-readme-tableofcontents) docs: README.md에 소개 및 목차 추가
| | * b8fd11b (origin/feature/machinity-add-troubleshooting-log-01-amend, feature/machinity-add-troubleshooting-log-01-amend) docs: troubleshooting-log.md 첫번째 amend 용례 추가
| |/
| * 9d248c1 (origin/feature/machinity-add-branch-naming-rule, feature/machinity-add-branch-naming-rule) docs: CONTRIBUTING.md 브랜치 네이밍 규칙 추가
|/
| * fbc3770 (origin/feature/islajin-contribution-rules) docs: 커밋 메시지와 PR 작성 규칙 정리했음
|/
* df3d104 chore: 협업 저장소 기본 구조 구성했음
```

revert 후 그래프
```
*   1789e51 (HEAD -> feature/update-troubleshooting-log-revert, origin/main, origin/HEAD, main) Merge pull request #20 from codyssey2-2-git-collab/feat/code-review
|\
| * 2fe7672 (origin/feat/code-review) docs: rebase 시 커밋 생성 문제 기록
| * dfffaf2 docs: rebase 시 충돌 해결
| * 255f936 docs: code review, merge와 충돌 관리 전략
* |   2da5b2a Merge pull request #14 from codyssey2-2-git-collab/feature/machinity-add-study-note
|\ \
| * | c219e1c (origin/feature/machinity-add-study-note, feature/machinity-add-study-note) docs: machinity_study_note.md 생성
* | |   176f952 Merge pull request #18 from codyssey2-2-git-collab/feature/troubleshooting-log
|\ \ \
| * | | 30951ea (origin/feature/troubleshooting-log) docs: reset과 stash 실습 기록 추가
* | | |   9da96fa Merge pull request #25 from codyssey2-2-git-collab/feature/machinity-add-troubleshooting-log-01-amend
|\ \ \ \
| * | | | 3dde151 (origin/feature/machinity-add-troubleshooting-log-01-amend, feature/machinity-add-troubleshooting-log-01-amend) docs: troubleshooting-log.md 첫번째 amend 용례 추가
| | |/ /
| |/| |
* | | |   b9fb479 Merge pull request #24 from codyssey2-2-git-collab/feature/machinity-update-readme-tableofcontents
|\ \ \ \
| * | | | 57c5108 (origin/feature/machinity-update-readme-tableofcontents, feature/machinity-update-readme-tableofcontents) docs: README.md에 소개 및 목차 추가
| |/ / /
* | | |   c71d417 Merge pull request #23 from codyssey2-2-git-collab/feature/machinity-add-branch-naming-rule
|\ \ \ \
| |/ / /
|/| | |
| * | | badb9a4 (origin/feature/machinity-add-branch-naming-rule, feature/machinity-add-branch-naming-rule) docs: CONTRIBUTING.md 브랜치 네이밍 규칙 추가
|/ / /
* | |   fe737e6 Merge pull request #22 from codyssey2-2-git-collab/feature/machinity-revert-merges
|\ \ \
| |_|/
|/| |
| * |   8a7f606 (origin/feature/machinity-revert-merges, feature/machinity-revert-merges) Merge remote-tracking branch 'origin/main' into feature/machinity-revert-merges
| |\ \
| |/ /
|/| |
* | |   72a5d61 Merge pull request #12 from codyssey2-2-git-collab/feature/islajin-learning-note
```
- 결과 참조 링크: 
[commit 808d7fa](https://github.com/codyssey2-2-git-collab/codyssey2_2/commit/808d7fa93cdb95daa59cfad7d39f4c900cdb6114),
[commit 4ded0e2](https://github.com/codyssey2-2-git-collab/codyssey2_2/commit/4ded0e2a28e37fc285e331b8426a1c32d100acd9),
[PR #22](https://github.com/codyssey2-2-git-collab/codyssey2_2/pull/22)
- revert 전 9d248c1 에서 이후 브랜치들이 갈라졌으나, revert 후 8a7f606과 fe737e6 사이에서 main으로부터 분기해나간 feature/machinity-* 브랜치들이 최종적으로 main으로 머지된 형상을 확인 가능
- 주의할 점:
  - revert 작업 도중 다른 팀원이 원격 push한 경우 안전을 위해 git push --force-with-lease 명령어 사용
  - cherry-pick 충돌이 나면 해당 파일을 열어 <<<<<<< 구간을 정리한 뒤 add, continue로 지속

### 왜 이 방법을 선택했는가(Why)
- main은 여러 팀원이 함께 쓰는 공유 브랜치이고, 내 merge 사이에 다른 팀원의 PR(#4)이 섞여 있었다.
  reset + force push는 원격 히스토리를 다시 쓰기 때문에 팀원들의 로컬과 충돌하고 다른 사람의 작업까지 영향을 준다.
- revert는 기존 커밋을 지우지 않고, 그 변경을 취소하는 새 커밋을 추가한다.
  따라서 히스토리가 그대로 남아 "무엇을 언제, 왜 되돌렸는지" 추적할 수 있다.

