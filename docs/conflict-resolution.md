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