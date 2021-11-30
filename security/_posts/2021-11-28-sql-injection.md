---
layout: post
title: SQL njection
categories: Security
---

# 웹 공격 시리즈 1 - SQL Injection

![](..\..\images\sql-injection.jpeg){: width="70%" height="70%"}

## SQL Injection이란?

SQL Injection은 애플리케이션 클라이언트의 입력 데이터를 통해 악의적인 SQL문을 실행시켜 데이터베이스를 비정상적으로 조작하는 공격 방법이다. 이 방법을 통해 권한을 가지고 있지 않은 사람이 데이터베이스에서 민감한 데이터를 읽고, 수정하고, 삭제하는 등의 작업을 할 수 있다. [OWASP Top 10 - 2021](https://owasp.org/Top10/)에서 3번째에 속해 있으며, 공격 난이도가 쉬운 편이고 큰 피해를 줄 수 있는 공격이다.

> OWASP?

OWASP(Open Web Application Security Project)는 오픈소스 웹 애플리케이션 보안 프로젝트이다. 주로 웹에 관한 정보 노출, 악성 파일 및 스크립트, 보안 취약점 등을 연구하며 10대 웹 애플리케이션의 취약점(OWASP Top 10)을 발표한다.

---





## SQL Injection 종류

### 1.   Error Based SQL Injection

Error Based SQL Injection은 쿼리에 고의적으로 에러를 발생시켜 에러의 내용을 통해 필요한 정보를 찾아내는 방법이다.

![image-20211128160305527](..\..\images\image-20211128160305527.png)

예를 들어 로그인을 하는 상황에서 위와 같은 `users` 테이블이 있고 다음과 같은 쿼리로 유저를 조회한다고 가정해 보자.

```sql
SELECT * FROM users WHERE user_id = ? AND password = ?;
```

하지만 table, column 정보는 보통 외부에 공개하지 않는다. 우선 이 정보를 파악하기 위해 `'`, `''`, `GROUP BY`, `HAVING` 같은 구문을 넣어 고의적으로 에러를 발생시킨다. 여기서 에러 메시지를 통해 table, column 이름에 대한 정보를 획득한다.

예를 들어, `'`를 삽입했을 때 `'`의 짝이 맞지 않아 아래와 같이 에러가 발생하게 된다. 이 에러 메시지가 노출된다면 `password`라는 이름의 column이 있음을 파악할 수 있다.

```sql
You have an error in your SQL syntax; check the manual that corresponds to your MySQL server version for the right syntax to use near '''' AND password = ''' at line 1
```

다음으로, 쿼리에 논리적 에러를 발생시킨다. id에 `test1`을 입력하고 password에 `' OR '1' = '1`이 입력된다면 완성된 쿼리는 다음과 같다.

```sql
SELECT * FROM users WHERE user_id = 'test1' AND password = '' OR '1' = '1';
```

앞의 AND 연산에서는 `user_id`가 `test1`이고, `password`가 비어 있으므로 거짓이 되고, 하뒤의 OR 연산은 항상 참이 된다. AND 연산이 우선으로 적용되기 때문에 `false OR true`가 되고 결과적으로 이 쿼리는 항상 참이 되어 인증 우회가 가능하다.



### 2. Union based SQL Injection

UNION문을 이용해 두 개 이상의 쿼리를 요청해 공격하는 방법이다. 

> UNION?

UNION은 여러 개의 SQL문을 합쳐 하나의 SQL문으로 만들 수 있다. 이 때, 두 테이블의 구조는 같아야 한다(컬럼의 갯수와 데이터 형식이 같다).

![image-20211128171804128](..\..\images\image-20211128171804128.png)

1번에 있었던 `users`테이블에 추가로 위와 같은 `contents`테이블과 이를 검색할 수 있는 input 창이 있고, 아래와 같은 쿼리를 통해 검색이 이루어진다고 가정하자.

```sql
SELECT * FROM contents WHERE title = ?;
```

역시나 우리는 DB에 대한 정보가 없어서 우선 DB 정보부터 획득해야 한다. 검색창에 `' UNION SELECT 1# -- `를 입력하면 완성된 쿼리는 다음과 같다.

```sql
SELECT * FROM contents WHERE title = '' UNION SELECT 1#-- ';
```

이 쿼리는 에러가 발생한다. 컬럼 갯수가 다르기 때문이다. 뒤의 숫자를 2,3,... 이런 식으로 늘려나가다 보면 `contents` 테이블의 컬럼 갯수와 같게 되고, 다음과 같은 결과를 얻을 수 있다. 결과의 숫자는 컬럼 순서를 의미한다.

```sql
SELECT * FROM contents WHERE title = '' UNION SELECT 1,2#-- ';
```

```sql
SELECT * FROM contents WHERE title = '' UNION SELECT 1,2,3#-- ';
```

![image-20211128173725574](..\..\images\image-20211128173725574.png)

다음으로 검색창에 `' UNION SELECT 1,table_name,3 FROM information_schema.tables# --`를 입력한다. 완성된 쿼리와 결과는 다음과 같다.

```sql
SELECT * FROM contents WHERE title = '' UNION SELECT 1,table_name,3 FROM information_schema.tables#-- ';
```

![image-20211128174452536](..\..\images\image-20211128174452536.png)

유저 테이블의 이름이 `users`이라는 것을 유추할 수 있다. 이제 검색창에 `' UNION SELECT 1,column_name,3 FROM information_schema.columns where table_name = 'users'# --`를 입력한다. 완성된 쿼리와 결과는 다음과 같다.

```sql
SELECT * FROM contents WHERE title = '' UNION SELECT 1,column_name,3 FROM information_schema.columns where table_name = 'users'# -- ';
```

![image-20211128174752589](..\..\images\image-20211128174752589.png)

이제 `users` 테이블의 컬럼 이름까지 획득했다. 이제 검색창에 `' UNION SELECT 1,id,password FROM users#-- `을 입력한다. 완성된 쿼리와 결과는 다음과 같다.

```sql
SELECT * FROM contents WHERE title = '' UNION SELECT 1,id,password FROM users#-- ';
```

![image-20211128175126262](..\..\images\image-20211128175126262.png)

유저의 id와 비밀번호를 얻는 데 성공했다.



### 3. Blind SQL Injection

에러가 발생되지 않는 경우에는 1번의 공격 방법을 사용할 수 없다. 이런 경우 쿼리의 참/거짓 동작을 확인해 DB 구조를 파악한다. 조건이 참이면 페이지가 정상적으로 출력되고, 거짓이면 페이지가 출력되지 않음으로 구분하는 원리이다. 

위와 같이 `users`, `contents` 테이블과 검색창이 있다고 가정하자. 검색창에 `qqqq AND 1 = 1# --`를 입력하면 쿼리가 참이 되어 글이 검색될 것이고, `qqqq AND 1 = 2# --`를 입력하면 쿼리가 거짓이 되어 검색되지 않을 것이다. 우리는 이 점을 이용할 것이다.

검색창에 이제 `qqqq' AND substr((SELECT table_name FROM information_schema.tables limit 0, 1), 1, 1) = 'u'# --`를 입력해 본다. 완성된 SQL문은 다음과 같다.

> substr(자를 문자열, 시작 위치, 자를 문자 길이) : 문자열을 자를 때 사용되는 함수

> limit 시작 위치, 갯수 : 출력할 row의 갯수를 지정한다. 시작 위치부터 갯수만큼 출력한다.

```sql
SELECT * FROM contents WHERE title = 'qqqq' AND substr((SELECT table_name FROM information_schema.tables limit 0, 1), 1, 1) = 'u'# -- ';
```

만약 첫번째 테이블 이름의 첫번째 글자가 u라면 글이 검색될 것이고, 아니라면 검색이 되지 않을 것이다. 문자를 조정하면서 어떤 글자인지 파악이 가능하다. substr 함수의 위치를 조정해 나가면서 모든 글자를 파악하고, limit 시작 위치를 조정해 모든 테이블의 이름 정보를 얻을 수 있다. 이 방법으로 유저 테이블의 정보까지 파악할 수 있을 것이다.

검색 효율성을 위해 ascii() 함수와 이진 탐색을 사용할 수 있다. 이진 탐색을 위해서는 대소 비교가 필요한데 문자는 비교가 불가능하므로 아스키 코드로 변환한다. ascii() 함수가 사용된 쿼리는 다음과 같다.

```sql
SELECT * FROM contents WHERE title = 'qqqq' AND ascii(substr((SELECT table_name FROM information_schema.tables limit 0, 1), 1, 1)) > 97# -- ';
```



### 4. Time based SQL Injection

이것도 Blind SQL Injection 기법 중 하나이다. 3번 방법 같은 경우 참/거짓이 나타나지 않는 경우 사용이 불가능하다. Time Based SQL Injection은 sleep() 함수를 이용해 응답 전송 시간을 지연시키는 방식으로 공격한다. 다음과 같은 쿼리의 경우 참이면 5초 뒤에 결과가 반환되고, 거짓이면 결과가 바로 반환될 것이다.

```sql
SELECT * FROM contents WHERE title = 'qqqq' AND ascii(substr((SELECT table_name FROM information_schema.tables limit 0, 1), 1, 1)) > 97 AND sleep(5)# -- ';
```

---





## SQL Injection 방어

### 1. 입력 값 검증

외부 입력값을 신뢰하지 않고 항상 유효성을 검사한다.

- 블랙 리스트 방식 : SQL 예약어, 함수명, 세미콜론이나 주석 등의 특수문자를 블랙리스트로 정의해 공백으로 치환한다. 하지만 공백으로 치환하면 SE**SELECT**LECT가 입력되었을 때 가운데 SELECT가 사라져 다시 SELECT가 만들어지므로 공백보다는 의미 없는 단어로 치환하는 방법이 더 안전하다.
- 화이트 리스트 방식 : 블랙 리스트와는 다르게 허용된 문자를 제외하고 모두 사용할 수 없도록 하는 방법이다.



### 2. Prepared Statement  구문 사용

Prepared Statement 구문을 사용하면 DBMS가 미리 컴파일한 후 사용자의 입력 값을 대입한다. 사용자의 입력 값이 단순 문자열로 인식되기 때문에 공격을 위한 쿼리가 들어가도 제대로 작동하지 않는다.



### 3. 에러 메시지 숨기기

에러 메시지에는 DB에 대한 정보를 얻을 수 있는 경우가 있다. 이런 에러 메시지를 외부에 노출하지 않음으로써 정보 수집을 더 까다롭게 한다.









