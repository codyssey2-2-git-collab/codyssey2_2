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
