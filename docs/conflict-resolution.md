


## 시나리오: 병합과 충돌
### 참여자 : pcochoco
### 상황 1
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

Rebase 도중에는 git commit 대신 git rebase --continue를 사용
```git rebase --continue```


