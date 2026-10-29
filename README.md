# DataBaseWithOracleLab
The repository of database using oracle.


# DataBase

관계형 데이터베이스의 원리를 이해하고 SQL로 데이터를 설계·조회·관리하기 위한 학습 자료입니다. 데이터 모델을 읽고 설계하는 과정부터 실제 데이터베이스에서 질의를 작성하고 결과를 검증하는 과정까지 다룹니다.

## 학습 목표

- 데이터베이스와 DBMS의 역할을 설명하고, 파일 기반 데이터 관리와의 차이를 이해합니다.
- 관계형 데이터 모델의 릴레이션, 튜플, 속성, 키, 무결성 제약조건을 구분합니다.
- SQL을 사용해 테이블을 정의하고 데이터를 조회·추가·수정·삭제합니다.
- 조인, 집계, 서브쿼리를 이용해 여러 테이블의 데이터를 분석합니다.
- 개체–관계 모델(ER 모델)과 정규화를 바탕으로 데이터 구조를 설계합니다.
- 트랜잭션, 인덱스, 뷰의 목적과 사용 시 주의점을 이해합니다.

## 학습 내용

| 영역 | 주요 내용 |
| --- | --- |
| 데이터베이스 기초 | 데이터와 정보, DBMS, 데이터 독립성, 관계형 데이터베이스 |
| 관계형 데이터 모델 | 릴레이션, 튜플, 속성, 도메인, 기본키·외래키, 무결성 |
| SQL 기초 | `SELECT`, `WHERE`, `ORDER BY`, `DISTINCT`, 연산자와 함수 |
| SQL 활용 | `JOIN`, `GROUP BY`, `HAVING`, 서브쿼리, 집합 연산 |
| 데이터 정의와 변경 | `CREATE`, `ALTER`, `DROP`, `INSERT`, `UPDATE`, `DELETE` |
| 데이터베이스 설계 | 요구사항 분석, ER 모델, 관계 스키마 변환, 함수 종속, 정규화 |
| 데이터베이스 운영 | 트랜잭션과 ACID, `COMMIT`·`ROLLBACK`, 뷰, 인덱스, 접근 권한 |

개념을 학습한 뒤 SQL 문장을 직접 실행하고, 결과가 예상과 다른 경우 데이터와 조건식을 확인하는 방식으로 학습하는 것을 권장합니다.

## 실습 환경

SQL 예제는 **Oracle AI Database 26ai Free**를 기준으로 작성했습니다. SQL Developer 또는 SQL*Plus에서 실행할 수 있습니다. 다른 DBMS에서는 자료형, 함수, 자동 커밋 및 일부 SQL 문법이 다를 수 있습니다.

1. [Oracle AI Database Free](https://www.oracle.com/database/free/get-started/)를 설치합니다.
2. [Oracle SQL Developer](https://www.oracle.com/database/sqldeveloper/) 또는 SQL*Plus를 준비합니다.
3. 실습용 일반 사용자 계정으로 데이터베이스에 접속합니다. 관리자 계정에서 실습 테이블을 생성하지 않는 것이 좋습니다.
4. SQL 워크시트 또는 SQL*Plus에서 아래 예제를 순서대로 실행합니다.

Oracle AI Database Free의 기본 PDB 연결 정보는 일반적으로 다음과 같습니다. 설치 과정에서 변경했다면 실제 설정값을 사용해야 합니다.

| 항목 | 값 |
| --- | --- |
| 호스트 | `localhost` |
| 포트 | `1521` |
| 서비스 이름 | `FREEPDB1` |
| 사용자 이름·암호 | 별도로 생성한 실습용 계정의 인증 정보 |

SQL*Plus에서는 다음 형식으로 접속할 수 있습니다. 암호는 명령에 직접 기록하지 않고 프롬프트에서 입력합니다.

```bash
sqlplus 사용자이름@localhost:1521/FREEPDB1
```

## SQL 실습: 수강 관리 데이터베이스

학생, 과목, 수강 정보를 세 개의 테이블로 나누어 관리합니다.

- `STUDENTS`: 학생 한 명의 기본 정보
- `COURSES`: 과목 한 개의 기본 정보
- `ENROLLMENTS`: 학생과 과목 사이의 수강 관계 및 점수

학생 한 명은 여러 과목을 수강할 수 있고, 과목 하나에는 여러 학생이 참여할 수 있습니다. 이 다대다 관계는 `ENROLLMENTS`라는 별도의 테이블로 구현합니다.

### 1. 테이블 생성

```sql
CREATE TABLE students (
    student_id   NUMBER(6) PRIMARY KEY,
    student_name VARCHAR2(50) NOT NULL,
    major        VARCHAR2(50) NOT NULL
);

CREATE TABLE courses (
    course_id    NUMBER(6) PRIMARY KEY,
    course_name  VARCHAR2(100) NOT NULL
);

CREATE TABLE enrollments (
    student_id NUMBER(6) NOT NULL,
    course_id  NUMBER(6) NOT NULL,
    score      NUMBER(3) CHECK (score BETWEEN 0 AND 100),
    CONSTRAINT pk_enrollments PRIMARY KEY (student_id, course_id),
    CONSTRAINT fk_enrollments_student
        FOREIGN KEY (student_id) REFERENCES students(student_id),
    CONSTRAINT fk_enrollments_course
        FOREIGN KEY (course_id) REFERENCES courses(course_id)
);
```

`PRIMARY KEY`는 각 행을 식별하며, `FOREIGN KEY`는 다른 테이블과의 참조 관계를 정의합니다. 복합 기본키 `(student_id, course_id)`는 같은 학생의 동일 과목 중복 수강 등록을 방지합니다.

### 2. 데이터 입력

```sql
INSERT INTO students VALUES (1001, '김민지', '컴퓨터교육');
INSERT INTO students VALUES (1002, '이도윤', '수학교육');

INSERT INTO courses VALUES (10, '데이터베이스');
INSERT INTO courses VALUES (20, '프로그래밍');

INSERT INTO enrollments VALUES (1001, 10, 95);
INSERT INTO enrollments VALUES (1001, 20, 88);
INSERT INTO enrollments VALUES (1002, 10, 91);

COMMIT;
```

### 3. 조인으로 수강 내역 조회

```sql
SELECT s.student_name,
       c.course_name,
       e.score
FROM students s
JOIN enrollments e ON s.student_id = e.student_id
JOIN courses c ON e.course_id = c.course_id
ORDER BY s.student_id, c.course_id;
```

예상 결과:

| STUDENT_NAME | COURSE_NAME | SCORE |
| --- | --- | ---: |
| 김민지 | 데이터베이스 | 95 |
| 김민지 | 프로그래밍 | 88 |
| 이도윤 | 데이터베이스 | 91 |

### 4. 집계와 그룹 조건

```sql
SELECT c.course_name,
       ROUND(AVG(e.score), 1) AS avg_score
FROM courses c
JOIN enrollments e ON c.course_id = e.course_id
GROUP BY c.course_name
HAVING AVG(e.score) >= 90
ORDER BY avg_score DESC;
```

`WHERE`는 그룹화 이전의 행을 걸러내고, `HAVING`은 그룹화 이후의 집계 결과에 조건을 적용합니다. 위 질의에서는 평균 점수가 90점 이상인 **데이터베이스(93점)** 과목만 조회됩니다.

### 5. 트랜잭션과 변경 취소

```sql
SAVEPOINT before_score_change;

UPDATE enrollments
SET score = 80
WHERE student_id = 1001 AND course_id = 10;

ROLLBACK TO before_score_change;
```

`ROLLBACK TO`는 지정한 저장점 이후의 변경을 되돌립니다. `COMMIT`으로 확정한 이전 작업까지 되돌리는 명령은 아닙니다. Oracle에서는 일부 DDL 문장이 암묵적 커밋을 발생시키므로 트랜잭션 실습과 스키마 변경 실습을 구분해야 합니다.

> **실습 시 주의:** 예제는 개인 학습용 스키마에서 실행하세요. 테이블을 이미 생성한 상태에서 생성문을 다시 실행하면 객체 이름 중복 오류가 발생할 수 있습니다. `UPDATE`와 `DELETE`를 실행하기 전에는 `WHERE` 조건을 반드시 확인하세요.

## 학습 점검

1. 기본키와 외래키는 각각 어떤 무결성을 보장하나요?
2. 학생과 과목의 다대다 관계를 별도의 수강 테이블로 분리하는 이유는 무엇인가요?
3. 내부 조인과 왼쪽 외부 조인의 결과는 어떤 경우에 달라지나요?
4. `WHERE`와 `HAVING`은 어느 단계에서 조건을 적용하나요?
5. 데이터 중복으로 발생하는 삽입·수정·삭제 이상을 정규화로 어떻게 줄일 수 있나요?
6. 트랜잭션의 ACID 특성이 필요한 상황을 하나씩 설명할 수 있나요?

## 참고 자료

- [Oracle AI Database 26ai 문서](https://docs.oracle.com/en/database/oracle/oracle-database/)
- [Oracle SQL Language Reference](https://docs.oracle.com/en/database/oracle/oracle-database/26/sqlrf/)
- [Oracle SQL Language Quick Reference](https://docs.oracle.com/en/database/oracle/oracle-database/26/sqlqr/)
- [Oracle AI Database Free 설치 안내](https://www.oracle.com/database/free/get-started/)

위 링크는 SQL 구문 및 실습 환경을 확인하기 위한 공식 참고 자료입니다. 각 사이트의 문서와 소프트웨어에는 해당 권리자의 별도 이용 조건이 적용됩니다.

## 라이선스

이 저장소에서 직접 작성한 자료는 [MIT License](LICENSE)에 따라 공개합니다. 외부 사이트의 문서, 이미지, 교재 발췌문 및 소프트웨어는 이 라이선스에 자동으로 포함되지 않습니다. 제3자 자료를 추가하는 경우 해당 자료의 출처와 이용 조건을 별도로 확인해야 합니다.

