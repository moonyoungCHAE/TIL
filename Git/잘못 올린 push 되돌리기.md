### 이미 Push 한 후, Push한 Commit 취소하기
1. ```git revert``` : commit & push 후 취소했다는 기록이 남는다.
2. ``` git reset ``` : commit & push했다는 기록이 남지 않는다.
 -- git reset hard로 local branch를 취소하고 싶은 부분까지 되돌린다. (local workspace의 code가 날아간다.)
 -- 바꾸고 싶은 상태로 다시 code를 작성한다.
 -- commit 한다.
 -- push 한다. (이 때, local branch와 origin server branch의 상태가 달라서 충돌이 난다. --> 그냥 git push force로 강제 push하면 잘못된 commit이 사라진다.)