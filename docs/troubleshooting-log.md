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
