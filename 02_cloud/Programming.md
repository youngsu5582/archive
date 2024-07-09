# Test
## 도서
### 단위 테스트
```dataview
table file.mtime
from #테스트
where 도서명="단위 테스트 : 생산성과 품질을 위한 단위 테스트 원칙과 패턴"
sort file.mtime asc
```
### 테스트 주도 개발 시작하기
```dataview
table file.mtime
from #테스트
where 도서명="테스트 주도 개발 시작하기"
sort file.mtime desc
```
## 아티클
```dataview
table 아티클_작성자,추가_정보
from #테스트
where 도서명=null
sort file.ctime asc
```

# 리팩토링
## 도서
### 객체지향과 사실의 오해
```dataview
table
from #리팩토링
where 도서명="객체지향의 사실과 오해"
sort file.mtime asc
```
## 아티클
```dataview
table 아티클_작성자,추가_정보
from #리팩토링
where 아티클_작성자 !=null
```
## 테크톡
```dataview
table 테크톡_발표자,테크톡_링크
from #리팩토링
where 테크톡_발표자 !=null
```
