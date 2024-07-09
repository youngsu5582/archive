## 강의
### 백발백중 시리즈
```dataview
table 강의_링크
from #데이터베이스 
where 강의명="백발백중 시리즈 - 데이터베이스"
SORT file.name asc
```
## 아티클
```dataview
table 아티클_작성자,추가_정보
from #데이터베이스 
where 아티클_링크 !=null
sort file.ctime asc
```
