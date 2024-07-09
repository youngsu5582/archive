# Java
## 강의
### 케빈 TV - 모던 자바 못다한 이야기
```dataview
table map(rows.file.path, (m) => link(m)) as "노트"
from #자바
where 강의명="모던 자바 (자바8) 못다한 이야기"
group by file.folder
```
### 백기선의 라이브 스터디
```dataview
table map(rows.file.path, (m) => link(m)) as "노트"
from #자바 
where 추가_정보="백기선 라이브 스터디"
group by (주차)
sort 주차 desc
```
## 도서
### 


### 아티클
```dataview
table 아티클_작성자,추가_정보
from #자바
where 아티클_작성자!=null
sort file.ctime asc
```
