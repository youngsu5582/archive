# 스프링
## 도서


```dataview
table
from #스프링 
where 도서명="스프링 입문을 위한 자바 객체 지향의 원리와 이해"
```

```dataview
table
from #스프링 
where 도서명=null
```
# JPA
## 강의
### JPA 프로그래밍 기본기 다지기
```dataview
table 강의_링크
from #JPA 
where 강의명="JPA 프로그래밍 기본기 다지기"
SORT file.name asc
```
### 어라운드 허브 스튜디오 JPA 강의
```dataview
table 강의_링크
from #JPA 
where 강의명="어라운드 허브 스튜디오 - JPA 강의"
SORT file.name asc
```

## 정리

```dataview
table 추가_정보
from #JPA 
where 강의명=null
```

