# #14 스토어드 프로그램

- MySQL에서는 절차적인 처리를 위해 스토어드 프로그램을 이용할 수 있다.

## 스토어드 프로그램의 장단점

- 스토어드 프로그램은 절차적인 처리를 제공하긴 하지만 애플리케이션을 대체할 수 있을지 충분히 고려해야 한다.

#### 스토어드 프로그램의 장점

`데이터베이스의 보안 향상`

- MySQL의 스토어드 프로그램은 자체적인 보안 설정 기능을 갖고 있고 스토어드 프로그램 단위로 실행 권한을 부여할 수 있다.
- 이러한 보안 기능을 조합해서 특정 테이블의 읽기와 쓰기, 또는 특정 칼럼에 대해서만 권한을 설정하는 등의 세밀한 설정이 가능하다.
- MySQL 서버의 스토어드 프로그램은 입력 값의 유효성을 체크한 후 동적인 SQL문장을 생성하므로 SQL의 문법적인 취약점(SQL Injection)을 이용한 해킹은 어렵다.

`기능의 추상화`

- 특정 기능의 추상화를 MySQL 서버의 스토어드 프로그램으로 작성해서 등록하면 개발 언어나 도구와 관계없이 특정 기능(ex: 일련번호 생성 규칙)을 쉽게 활용할 수 있다.

`네트워크 소요 시간 절감`

- 일반적으로 애플리케이션과 데이터베이스 서버는 같은 네트워크 구간에 존재하므로 SQL의 실행 성능에서 네트워크를 경유하는 데 걸리는 시간은 그다지 중요하게 생각하지 않는다.
- 하지만 매우 가벼운 쿼리를 1~200번씩 호출하는 부분에서 네트워크 경유시간은 무시할 수 없는 부분이다.
- 특정 프로그램에서 1~200번씩 실행해야하는 쿼리를 스토어드 프로그램으로 구현한다면 스토어드 프로그램을 호출할 때 한 번만 네트워크를 경유하면 되기 때문에 네트워크 소요 시간을 줄이고 성능을 개선할 수 있다.

`절차적 기능 구현`

- DBMS 서버에서 사용하는 SQL 쿼리는 절차적인 기능을 제공하지 않지만 스토어드 프로그램은 절차적인 기능을 실행할 수 있는 제어 기능을 제공한다.
- 이런 절차적인 기능을 스토어드 프로그램 없이 사용하면 보통 애플리케이션에서 가공하고 다시 데이터베이스에 저장하는 형태로 개발을 진행하는데 이는 위에서 설명한 네트워크 경유 시간이 추가되므로 스토어드 프로그램을 사용하면 이런 부분을 개선할 수 있다.

`개발 업무의 구분`

- 순수하게 애플리케이션 개발 조직과 SQL 개발 조직이 구분돼 있는 회사도 있다.
- 위와 같은 경우 업무의 구분을 명확하게 할 수 있다.

#### 스토어드 프로그램의 단점

`낮은 처리 성능`

- 스토어드 프로그램은 MySQL 엔진에서 해석되고 실행된다. 하지만 MySQL 서버는 스토어드 프로그램과 같은 절차적 코드 처리를 주목적으로 하는 것이 아니라서 스토어드 프로그램의 처리 성능이 다른 프로그램 언어에 비해 상대적으로 떨어진다.
- 또한 다른 DBMS의 스토어드 프로그램과 달리 MySQL 서버의 스토어드 프로그램은 실행시마다 스토어드 프로그램의 코드가 파싱돼야 한다.

`애플리케이션 코드의 조각화`

- 애플리케이션 코드가 스토어드 프로그램과 분산된다면 유지보수가 어려워질 수 있다.

<br>

## 스토어드 프로그램의 문법

- 스토어드 프로그램도 헤더 부분과 본문 부분으로 나눌 수 있다.
- 헤더 부분은 정의부라고 하며, 주로 스토어드 프로그램의 이름과 입출력 값을 명시하는 부분이다. 추가로 보안이나 작동 방식과 관련된 옵션도 명시할 수 있다.
- 본문 부분은 스토어드 프로그램이 호출됐을 때 실행하는 내용을 작성하는 부분이다.

### 스토어드 프로시저

- 스토어드 프로시저는 서로 데이터를 주고받아야 하는 여러 쿼리를 하나의 그룹으로 묶어서 독립적으로 실행하기 위해 사용하는 것이다.
- 각 쿼리가 서로 연관되어 데이터를 주고받으면서 반복적으로 실행되어야 할 때 스토어드 프로시저를 사용하면 MySQL 서버와 클라이언트 간에 네트워크 전송 작업을 최소화하고 수행 시간을 줄일 수 있다.
- 스토어드 프로시저는 반드시 독립적으로 호출돼야 하며, `SELECT`나 `UPDATE` 같은 SQL 문장에서 스토어드 프로시저를 참조할 수 없다.

#### 스토어드 프로시저 생성 및 삭제

`스토어드 프로시저 생성`

```sql
DELIMITER ;;

CREATE PROCEDURE sp_sum (IN param1 INTEGER, IN param2 INTEGER, OUT param3 INTEGER) -- 프로시저 이름과 파라미터 정의
BEGIN -- 스토어드 프로시저 본문 시작점
SET param3 = param1 + param2;
END;; -- 스토어드 프로시저 본문 종료지점

DELIMITER ;
```

- `param1` + `param2`의 결과값을 `param3`에 저장하는 프로시저

`프로시저 생성시 주의사항`

- 스토어드 프로시저는 기본 반환값이 없다. 즉 스토어드 프로시저 내부에서는 값을 반환하는 `RETURN` 명령을 사용할 수 없다.
- 스토어드 프로시저의 각 파라미터는 다음의 3가지 특성 중 하나를 지닌다.
  - `IN 타입`
    - `IN` 타입으로 정의된 파라미터는 입력 전용 파라미터를 의미한다.
    - 스토어드 프로시저 외부에서  스토어드 프로시저를 호출할 때 프로시저에 값을 전달하는 용도로 사용하고 값을 반환하는 용도로 사용하지 않는다.
    - `IN` 타입으로 정의된 파라미터는 스토어드 프로시저 내부에서 읽기 전용으로 이해하면 된다.
  - `OUT` 타입
    - `OUT` 타입으로 정의된 파라미터는 출력 전용 파라미터를 의미한다.
    - 스토어드 프로시저 외부에서 스토어드 프로시저를 호출할 때 어떤 값을 전달하는 용도로는 사용할 수 없다.
    - 스토어드 프로시저의 실행이 완료되면 외부 호출자로 결과 값을 전달하는 용도로만 사용한다.
  - `INOUT` 타입
    - `INOUT` 으로 정의된 파라미터는 입력 및 출력 용도로 모두 사용할 수 있다.
- 일반적으로 MySQL 클라이언트 프로그램에서는 `;` 문자가 쿼리의 끝을 의미하는데 스토어드 프로그램 내부에서 무수히 많은 `;` 문자를 포함하므로 MySQL 클라이언트가 `CREATE PROCEDURE` 명령의 끝을 정확히 찾을 수 없다. 따라서 `DELIMITER` 명령을 통해 `CREATE` 명령의 끝을 정확하게 판별할 수 있도록 문자를 변경해줘야한다.
- 일반적으로 `DELIMITER`는 `;;` 또는 `//`을 사용한다. 그 외에도 스토어드 프로그램에서 사용되지 않는 문자열은 전부 사용 가능하다.
- 위에서 변경한 `DELIMITER`는 `SELECT` , `INSERT` 와 같은 일반적인 SQL 문장에도 영향을 주므로 다시 기본 종료 문자인 `;` 로 변경해야 한다.

<br/>

`스토어드 프로시저 수정`

```sql
ALTER PROCEDURE sp_sum SQL SECURITY DEFINER; -- 보안 관련 설정
```

- 스토어드 프로시저의 파라미터나 프로시저의 본문을 변경할 때는 `ALTER` 가 아니라 `DROP` 후에 다시 생성하는 것이 유일한 방법이다.

`스토어드 프로시저 삭제`

```sql
DROP PROCEDURE sp_sum;
```

<br>

#### 스토어드 프로시저 실행

- 스토어드 프로시저는 반드시 `CALL` 명령어로 실행해야 한다.
- MySQL 클라이언트에서 스토어드 프로시저를 실행할 때 `IN` 타입의 파라미터는 상숫값을 그대로 전달해도 무방하지만 `OUT` 또는 `INOUT` 타입의 파라미터는 세션 변수를 이용해 값을 주고받아야 한다.

```sql
SET @result:=0;
SELECT @result;

CALL sp_sum(1, 2, @result);
SELECT @result;
```

![74](https://github.com/sup2is/dev-note/blob/master/db/mysql/images/real-mysql-8/74.png)

- `IN` 타입은 일반 리터럴로 사용해도 되고 세션변수로 사용해도 된다.
- 자바나 C/C++ 같은 프로그래밍 언어에서는 위와 같이 세션 변수를 사용하지 않고 바로 `OUT` 이나 `INOUT` 타입의 변숫값을 읽어올 수 있다.



#### 스토어드 프로시저의 커서 반환

- 스토어드 프로그램은 명시적으로 커서를 파라미터로 전달받거나 반환할 수 없다.
- 하지만 프로시저 내에서 커서를 오픈하지 않거나 `SELECT` 쿼리의 결과 셋을 fetch하지 않으면 해당 쿼리의 결과 셋은 클라이언트로 바로 전송된다.

```sql
DELIMITER ;;
CREATE PROCEDURE sp_selectEmployees (IN in_empno INTEGER)
BEGIN
	SELECT * FROM employees WHERE emp_no=in_empno; -- SELECT 를 실행했지만 그 결과를 전혀 사용하지 않음
END;;
DELIMITER ;

CALL sp_selectEmployees(10001); 

```

![75](https://github.com/sup2is/dev-note/blob/master/db/mysql/images/real-mysql-8/75.png)

- 실행 결과를 확인하면 `SELECT` 쿼리의 결과 셋을 별도로 반환하는 `OUT` 변수에 담거나 출력처리를 하지 않았음에도 불구하고 쿼리의 결과가 클라이언트로 전송된 것을 확인할 수 있다.
- 이 기능은 JDBC를 이용하는 자바 프로그램에서도 그대로 이용할 수 있고 하나의 스토어드 프로시저에서 2개 이상의 결과 셋을 반환할 수도 있다.
- MySQL 스토어드 프로시저는 아직 메시지를 화면에 출력하는 기능을 제공하지 않으며 별도의 로그 파일에 기록하는 기능도 없다. 스토어드 프로시저를 디버깅해야 하는 경우 위와 같은 기능을 활용해서 변숫값을 트래킹하거나 상태 변화 여부를 쉽게 확인할 수 있다.



#### 스토어드 프로시저 딕셔너리

- MySQL 8.0 이전 버전까지는 스토어드 프로시저는 `mysql` 데이터베이스의 `proc` 테이블에 저장됐지만 MySQL 8.0 부터는 사용자에게 보이지 않는 시스템 테이블로 저장된다.
- 사용자는 단지 `information_schema` 데이터 베이스의 `ROUTINES` 뷰를 통해 스토어드 프로시저의 정보를 조회할 수만 있다.

```sql
SELECT ROUTINE_SCHEMA, ROUTINE_NAME, ROUTINE_TYPE
FROM information_schema.ROUTINES
WHERE routine_schema='employees';
```



![76](https://github.com/sup2is/dev-note/blob/master/db/mysql/images/real-mysql-8/76.png)

<br>

### 스토어드 함수

- 스토어드 함수는 하나의 SQL 문장으로 작성이 불가능한 기능을 하나의 SQL 문장으로 구현해야 할 때 사용한다.
- 스토어드 함수는 상대적으로 스토어드 프로시저보다 제약 사항이 많기 때문에  SQL문장과 관계 없이 별도로 실행되는 기능이라면 스토어드 프로시저를 사용하는게 좋다.
- 스토어드 함수의 유일한 장점은 SQL문장의 일부로 사용할 수 있는 것이다.

`예제`

- 부서별로 가장 최근에 배속된 사원을 2명씩 가져오는 기능
- `dept_emp` 테이블의 데이터를 부서별로 그루핑하는 것 까지는 가능하지만 해당 부서 그룹별로 최근 2명씩만 잘라서 가져오는 방법이 없다.
- 이럴 때 부서 코드를 인자로 입력받아 최근 2명의 사원 번호만 `SELECT` 하고 문자열로 결합해서 반환하는 함수를 만들어서 사용하면 된다.

```sql
SELECT dept_no, sf_getRecentEmp(dept_no)
FROM dept_emp
GROUP BY dept_no;
```

- MySQL 5.7 이전까지는 특정 그룹별로 몇 건씩만 레코드를 조회하는 기능은 위와 같이 스토어드 함수로만 구현 가능했지만 8.0 부터 레터럴 조인과 윈도우 함수를 이용해서 구현할 수 있다.

#### 스토어드 함수 생성 및 삭제

`스토어드 함수 생성`

```sql
DELIMITER ;;

CREATE FUNCTION sf_sum(param1 INTEGER, param2 INTEGER)
	RETURNS INTEGER
BEGIN
	DECLARE param3 INTEGER DEFAULT 0;
	SET param3 = param1 + param2;
	RETURN param3;
END;;

DELIMITER ;


-- 버전이 올라가면서 아래 설정 기본값이 OFF로 되어있는데 ON으로 바꿔주면 스토어드 함수를 생성할 수 있다.
SET GLOBAL log_bin_trust_function_creators = 1;
```

- 스토어드 함수는 `CREATE FUNCTION` 명령으로 생성한다.
- 모든 입력 파라미터는 읽기 전용이라서 `IN` 또는 `OUT` 같은 타입을 지정할 수 없다.
- 스토어드 함수는 반드시 정의부에 `RETURNS` 키워드를 이용해 반환되는 값의 타입을 명시하고 `RUTURN` 명령어로 반환해야 한다.

`스토어드 프로시저와 비교했을 때 스토어드 함수의 BEGIN...END 사이 제약사항`

- `PREPARE`와 `EXECUTE` 명령을 이용한 프리페어 스테이트먼트를 사용할 수 없다.
- 명시적 또는 묵시적인 `ROLLBACK/COMMIT`을 유발하는 SQL 문장을 사용할 수 없다.
- 재귀 호출을 사용할 수 없다.
- 스토어드 함수 내에서 프로시저를 호출할 수 없다.
- 결과 셋을 반환하는 SQL 문장을 사용할 수 없다.
- 스토어드 함수 내에서 커서를 정의하면 반드시 오픈해야하고 스토어드 프로시저와 달리 `SELECT ... INTO` 가 아닌 단순히  `SELECT` 쿼리만 실행하면 오류가 발생한다. 단순 `SELECT` 쿼리가 실행되는 것은 결과적으로 클라이언트로 쿼리의 결과 셋을 반환하는 것과 같기 때문이다. 

```sql
DELIMITER ;;

CREATE FUNCTION sf_resultset_test()
	RETURNS INTEGER
BEGIN
	DECLARE param3 INTEGER DEFAULT 0;
	SELECT 'Start stored function' AS debug_message;
  RETURN param3;
END;;
  
DELIMITER ;
```

 ![77](https://github.com/sup2is/dev-note/blob/master/db/mysql/images/real-mysql-8/77.png)

`스토어드 함수 수정`

```sql
ALTER FUNCTION sf_sum SQL SECURITY DEFINER; -- 보안 관련 설정
```

- 스토어드 프로시저와 마찬가지로 스토어드 함수의 입력 파라미터나 본문은 변경이 불가능하고 특성만 변경할 수 있다.

`스토어드 함수 삭제`

```sql
DROP FUNCTION sf_sum;
```

- 스토어드 프로시저와 마찬가지로 본문이나 입력 파라미터를 변경하려면 삭제 후 다시 생성해야 한다.



#### 스토어드 함수 실행

- 스토어드 함수는 스토어드 프로시저와 달리 `CALL` 명령으로 실행할 수 없고 반드시 `SELECT` 문장으로 실행해야 한다.

```sql
SELECT sf_sum(1, 1) AS sum;
```

![78](https://github.com/sup2is/dev-note/blob/master/db/mysql/images/real-mysql-8/78.png)



### 트리거

- 트리거는 테이블의 레코드가 저장되거나 변경될 때 미리 정의해둔 작업을 자동으로 실행해주는 스토어드 프로그램이다.
- MySQL 트리거는 테이블의 레코드가 `INSERT`, `UPDATE`, `DELETE` 될 때 시작되도록 설정할 수 있다.
- 대표적으로 칼럼의 유효성 체크, 다른 테이블로의 복사나 백업, 계산된 결과를 다른 테이블에 함께 업데이트 하는 등의 작업을 위해 트리거를 자주 사용한다.
- 트리거는 스토어드 함수나 프로시저보다는 필요성이 떨어지는 편이다. 트리거가 없어도 애플리케이션의 개발이 크게 어렵지 않고 트리거가 적용된 테이블에 칼럼을 추가하거나 삭제하는 작업은 임시 테이블에 데이터를 복사하는 작업이 필요한데 이때 레코드마다 트리거를 한 번씩 실행해야 하기 때문에 성능상 좋지 않다.
- MySQL 5.7 버전 이후부터는 하나의 테이블에 대해서 동일 이벤트에 2개 이상의 트리거를 생성할 수 있게 됐다.
- MySQL 서버가 ROW 포맷의 바이너리 로그를 이용해서 복제를 하는 경우 트리거는 복제 소스 서버에서만 실행되고 레플리카 서버에서는 별도로 트리거를 기동하지 않는다. 하지만 이미 복제 소스 서버에서 트리거에 의해 발생된 데이터는 모두 바이너리 로그에 기록되기 때문에 실제 레플리카 서버에서도 트리거를 실행한 것과 동일한 효과를 낸다.



#### 트리거 생성

```sql
CREATE TRIGGER on_delete BEFORE DELETE ON employees
	FOR EACH ROW
BEGIN
	DELETE FROM salaries WHERE emp_no=OLD.emp;
END ;;
```

- 트리거의 생성은 `CREATE TRIGGER` 명령으로 한다.
- 트리거 이름 뒤에는 [`BEFORE` | `AFTER` ] [`INSERT` | `UPDATE` | `DELETE`] 의 6가지 조합이 가능하다.
  - `BEFORE` 트리거는 대상 레코드가 변경되기 전에 실행되고 `AFTER` 트리거는 대상 레코드의 내용이 변경된 후 실행된다.
- 테이블명 뒤에는 트리거가 실행될 단위를 명시하는데 `FOR EACH ROW`만 가능하므로 모든 트리거는 항상 레코드 단위로 실행된다.
- 예제 트리거에서 사용된 `OLD` 키워드는 `employees` 테이블의 변경되기 전 레코드를 지칭한다. `employees` 테이블의 변경될 레코드를 지칭하고자 할 때는 `NEW` 키워드를 사용하면 된다.
- 위 트리거는 `employees` 테이블의 레코드를 삭제하는 쿼리가 실행되면 해당 레코드가 삭제되기 전에 `on_delete` 라는 트리거가 실행되고 트리거가 완료된 이후 테이블의 레코드가 삭제된다.
- 참고로 테이블에 `DROP`이나 `TRUNCATE`가 실행되는 경우에는 트리거 이벤트는 발생하지 않는다.

`대표적인 쿼리에 대해 어떤 트리거 이벤트가 발생하는지 정리한 표`

| SQL 종류                               | 발생 트리거 이벤트 ==> 는 발생하는 이벤트의 순서를 의미      |
| -------------------------------------- | ------------------------------------------------------------ |
| INSERT                                 | `BEFORE INSERT` ==> `AFTER INSERT`                           |
| LOAD DATA                              | `BEFORE INSERT` ==> `AFTER INSERT`                           |
| REPLACE                                | 중복 레코드가 없을 때:  <br />  - `BEFORE INSERT` ==> `AFTER INSERT` <br />중복 레코드가 있을 때: <br />  - `BEFORE DELETE` ==> `AFTER DELETE` ==> `BEFORE INSERT` ==> `AFTER INSERT` |
| INSERT INTO ... <br />ON DUPLICATE SET | 중복 레코드가 없을 때:  <br />  - `BEFORE INSERT` ==> `AFTER INSERT` <br />중복 레코드가 있을 때: <br />  - `BEFORE UPDATE` ==> `AFTER UPDATE` |
| UPDATE                                 | `BEFORE UPDATE` ==> `AFTER UPDATE`                           |
| DELETE                                 | `BEFORE DELETE` ==> `AFTER DELETE`                           |
| TRUNCATE                               | 이벤트 발생하지 않음                                         |
| DROP TABLE                             | 이벤트 발생하지 않음                                         |

`트리거의 BEGIN ... END 의 코드블록 제약 사항`

- 트리거는 외래키 관계에 의해 자동으로 변경되는 경우 호출되지 않는다.
- 복제에 의해 레플리카 서버에 업데이트 되는 데이터는 레코드 기반의 복제 (Row based replication) 에서는 레플리카 서버의 트리거를 기동시키지 않지만 문장 기반의 복제(Statement based replication) 에서는 레플리카 서버에서도 트리거를 기동시킨다.
- 명시적 또는 묵시적인 `ROLLBACK/COMMIT`을 유발하는 SQL 문장을 사용할 수 없다.
- `RETURN` 문장을 사용할 수 없고 트리거를 종료할 때는 `LEAVE` 명령을 사용한다.
- `mysql`과 `information_schema.performance_schema` 데이터베이스에 존재하는 테이블에 대해서는 트리거를 생성할 수 없다.



#### 트리거 실행

- 트리거는 스토어드 프로시저와 함수와 같이 작동을 확인하기 위해 명시적으로 실행해볼 수 있는 방법이 없다.
- 트리거가 등록된 테이블에 직접 레코드를 `INSERT`, `DELETE`, `UPDATE`를 수행해서 작동을 확인해야 한다.

#### 트리거 딕셔너리

- MySQL 8.0 이전 버전까지 트리거는 해당 데이터베이스 디렉터리의 `*.TRG` 라는 파일로 기록됐지만 8.0버전부터는 딕셔너리 정보가 InnoDB 스토리지 엔진을 사용하는 시스템 테이블로 통합 저장되면서 더이상 `*.TRG` 파일로 저장되지 않는다. 
- MySQL 8.0부터는 `mysql` 데이터베이스의 보이지 않는 시스템 테이블로 저장되고 사용자는 `information_schema` 데이터베이스의 `TIRGGERS` 뷰를 통해 조회만 할 수 있다.

```sql
SELECT trigger_schema, trigger_name, event_manipulation, action_timing
FROM information_schema.TRIGGERS
WHERE trigger_schema='employees';
```

![79](https://github.com/sup2is/dev-note/blob/master/db/mysql/images/real-mysql-8/79.png)



### 이벤트

- 주어진 특정한 시간에 스토어드 프로그램을 실행할 수 있는 스케줄러 기능을 이벤트라고 한다.
- MySQL 서버의 이벤트는 스케쥴링을 전담하는 스레드가 있는데 이 스레드가 활성화된 경우에만 이벤트가 실행된다.

```sql
SHOW GLOBAL VARIABLES LIKE 'event_scheduler'; -- 이 설정값이 ON 으로 되어 있어야 한다.
```

![80](https://github.com/sup2is/dev-note/blob/master/db/mysql/images/real-mysql-8/80.png)

```sql
SHOW PROCESSLIST;
```

![81](https://github.com/sup2is/dev-note/blob/master/db/mysql/images/real-mysql-8/81.png)

- MySQL의 이벤트는 전체 실행 이력을 보관하지 않고 가장 최근에 실행된 정보만 `information_schema` 데이터베이스의 EVENTS 뷰에서 확인할 수 있다.
- 실행 이력이 필요한 경우 직접 사용자 테이블을 생성하고 기록해둬야하는데 필요가 없다고 생각해도 기록해두는게 좋다고 한다.



#### 이벤트 생성

- 이벤트는 반복 실행 여부에 따라 크게 일회성 이벤트와 반복성 이벤트로 나눌 수 있다.

`일회성 이벤트`

```sql
CREATE EVENT onetime_job
	ON SCHEDULE AT CURRENT_TIMESTAMP + INTERVAL 1 HOUR
DO
	INSERT INTO daily_rank_log VALUES (NOW(), 'Done');
```

- 현재 시점부터 1시간 뒤에 실행될 이벤트를 등록

 `반복성 이벤트`

```sql
CREATE EVENT daily_rank
	ON SCHEDULE EVERY 1 DAY STARTS '2022-01-01 00:00:00' ENDS '2023-01-01 00:00:00'
DO
	INSERT INTO daily_rank_log VALUES (NOW(), 'Done');
```

- `2022-01-01 00:00:00` 마다 매일 실행되는 이벤트를 등록
- 이벤트 생성 명령의 `EVERY` 절에는 `YEAR`, `MONTH`, `DAY`, `QUARTER`, `HOUR`, `MINUTE`, `WEEK`, `SECOND` 등등 반복 주기를 사용할 수있다.

<br/>

- 반복성, 일회성과 관계없이 이벤트의 처리 내용을 작성하는 `DO` 절은 여러 가지 방식으로 사용할 수 있다.
- `DO`절에는 단순히 하나의 쿼리나 스토어드 프로시저를 호출하는 명령을 사용하거나 `BEGIN ... END` 로 구성되는 복합 절을 사용하 수있다. 단일 SQL의 경우 `BEGIN ... END` 를 사용하지 않아도 무방하다.

```sql
CREATE EVENT daily_rank
	ON SCHEDULE EVERY 1 DAY STARTS '2022-01-01 00:00:00' ENDS '2023-01-01 00:00:00'
DO BEGIN
	INSERT INTO daily_rank_log VALUES (NOW(), 'Start');
	-- // 랭킹 정보 수집 & 처리
	INSERT INTO daily_rank_log VALUES (NOW(), 'Done');
END ;;
```

- 이벤트의 반복성 여부와 관계 없이 `ON COMPLETION` 절을 이용해 완전히 종료된 이벤트를 삭제할지, 그대로 유지할지 선택할 수 있다. 기본적으로 완전히 종료된 이벤트는 자동으로 삭제된다.
- `ON COMPLETION PRESERVE` 옵션을 주면 자동으로 삭제되지 않는다.
- 이벤트는 생성할 때 `ENABLE`, `DISABLE`, `DISABLE ON SLAVE` 3가지 상태로 생성할 수 있다. 기본적으로 생성되면서 복제 소스 서버에서는 `ENABLE` 되며, 복제된 레플리카 서버에서는 `SLAVESIDE_DISABLED` 상태(`DISBLE ON SLAVE` 옵션)로 생성된다.
- 복제 소스 서버에서는 실행된 이벤트가 만들어낸 데이터 변경 사항은 자동으로 레플리카 서버로 복제되기 때문에 레플리카 서버에서는 이벤트를 중복해서 실행할 필요는 없다. 다만 레플리카 서버가 소스 서버로 승격되면 수동으로 이벤트의 상태를 `ENABLE` 상태로 변경해야 한다.

`레플리카 서버에서만 DISABLE된 이벤트 목록 조회`

```sql
SELECT event_schema, event_name
FROM information_schema.EVENTS
WHERE STATUS = 'SLAVESIDE_DISABLED';
```

<br/>

#### 이벤트 실행 및 결과 확인

- 이벤트 또한 트리거와 같이 특정한 사건이 발생해야 실행되는 스토어드 프로그램이라서 테스트를 위해 강제로 실행시켜볼 수는 없다.

```sql
DELIMITER ;;
CREATE TABLE daily_rank_log (exec_dttm DATETIME, exec_msg VARCHAR(50));

CREATE EVENT daily_rank
	ON SCHEDULE AT CURRENT_TIMESTAMP + INTERVAL 1 MINUTE
	ON COMPLETION PRESERVE
DO BEGIN
	INSERT INTO daily_rank_log VALUES (NOW(), 'Done');
END ;;
	
DELIMITER ;
```

`이벤트 확인하기`

```sql
SELECT * FROM information_schema.EVENTS
```

![82](https://github.com/sup2is/dev-note/blob/master/db/mysql/images/real-mysql-8/82.png)

```sql
SELECT * FROM daily_rank_log;
```

![83](https://github.com/sup2is/dev-note/blob/master/db/mysql/images/real-mysql-8/83.png)

- `information_schema` 데이터베이스의 `EVENTS` 뷰는 항상 마지막 실행 로그만 가지고 있기 때문에 전체 실행 로그가 필요한 경우에는 위와 같이 별도의 로그 테이블이 필요하다.

#### 이벤트 딕셔너리

- MySQL 8.0 이전 버전까지 생성된 이벤트의 딕셔너리 정보는 `mysql` 데이터베이스의 `events` 테이블에서 관리됐다.
- MySQL 8.0 버전부터는 다른 스토어드 프로그램과 마찬가지로 `information_schema` 데이터베이스의 `EVENTS` 뷰를 통해 이벤트의 목록과 상세 내용을 확인할 수 있다.



### 스토어드 프로그램 본문(Body) 작성

- 지금까지 살펴본 스토어드 프로그램은 생성하고 실해아는 방법에 조금씩 차이가 있지만 본문부(`BEGIN ... END 블록`)는 모두 똑같은 문법을 사용한다.

#### BEGIN ... END 블록과 트랜잭션

- 스토어드 프로그램의 본문은 `BEGIN` 으로 시작해서 `END` 로 끝나며 하나의 `BEGIN ... END ` 블록은 또 다른 여러 개의 `BEGIN ... END ` 블록을 중첩해서 포함할 수 있다.
- `BEGIN ... END `  내에서 주의해야 할 것은 트랜잭션 처리다. MySQL에서 트랜잭션을 시작하는 명령은 `BEGIN`과 `START TRANSACTION` 인데 `BEGIN ... END ` 안에서 `BEGIN` 은 `BEGIN ... END ` 으로 인식한다. 따라서 스토어드 프로그램 본문에서 트랜잭션 시작은 반드시 `START TRANSACTION` 을 사용해야 한다.
- 트랜잭션은 스토어드 프로시저, 이벤트의 본문에서만 사용 가능하고 스토어드 함수나 트리거에서는 트랜잭션을 사용할 수 없다.
- 스토어드 프로시저에서 트랜잭션을 완료하면 이 스토어드 프로시저를 호출한 애플리케이션이나 SQL 클라이언트 도구에서는 트랜잭션을 조절할 수 없다. 따라서 어떤 부분에서 트랜잭션을 완료해야할지 명확하게 정의해야 한다.



#### 변수

- 스토어드 프로그램의 `BEGIN ... END ` 블록 사이에서 사용하는 변수는 사용자 변수와는 다르므로 혼동하지 않도록 주의해야 한다.
- 여기서 언급하는 변수(로컬 변수)는 `BEGIN ... END ` 블록 내에서만 사용할 수 있다.
- 스토어드 프로그램에서 사용자 변수와 로컬 변수는 거의 혼용해서 제한 없이 사용할 수 있지만 프로시저 내부에서 프리페어 스테이트먼트를 사용하려면 반드시 사용자 변수를 사용해야 한다.
- 스토어드 프로그램에서 사용자 변수를 적절히 용도에 맞게 사용하여 스토어드 프로그램 내부, 외부 간의 데이터 전달용도로 사용할 수 있지만 너무 남용하면 악영향을 미칠수 있다. 로컬 변수가 빠르게 처리되기 때문에 프로그램 내부에서는 가능하면 로컬 변수를 사용하는게 좋다.

`로컬 변수`

```sql
-- // 로컬 변수 정의
DECLARE v_name VARCHAR(50) DEFAULT 'Matt';
DECLARE v_email VARCHAR(50) DEFAULT 'matt@email.com';

-- // 로컬 변수에 값을 할당
SET v_name = 'Kim', v_email = 'kim@email.com';

-- // SELECT .. INTO 구문을 이용한 값의 할당
SELECT emp_no, first_name, last_name INTO v_empno, v_firstname, v_lastname
FROM employees
WHERE emp_no = 10001;
LIMIT 1;
```

- 로컬 변수는 `DECLARE` 명령으로 정의되고 반드시 타입이 함께 명시되어야 한다.
- 로컬 변수에 값을 할당하는 방법은 `SET` 또는 `SELECT ... INTO ...` 문장으로 가능하다.`SELECT ... INTO ...` 는 반드시 1개의 레코드를 반환하는 SQL 이어야 한다. 따라서 `LIMIT 1;` 과 같은 조건을 추가해서 사용하는 것이 좋다.
- 로컬변수는 `BEGIN ... END ` 블록 내에서만 유효하다.
- 로컬 변수는 초기 디폴트값을 설정할 수 있고 명시하지 않으면 `NULL` 로 초기화 된다.
- 스토어드 프로그램의 `BEGIN ... END ` 블록에서는 스토어드 프로그램의 입력 파라미터와 `DECLARE` 에 의해 생성된 로컬 변수, 테이블의 칼럼명 모두 같은 이름을 가질 수 있다. 이 때 다음과 같은 우선순위를 가진다.
  1. `DECLARE`로 정의한 로컬 변수
  2. 스토어드 프로그램의 입력 파라미터
  3. 테이블 칼럼

- 위와 같은 경우 스토어드 프로그램이 복잡해지기 때문에 입력 파라미터는 `p_` 의 접두사, 로컬변수는 `v_` 접두사를 사용하는것도 좋은 방법이다.
- 중첩된 `BEGIN ... END ` 블록은 각각 똑같은 이름의 로컬 변수를 정의할 수 있는데 외부 블록에서는 내부 블록에 정의된 로컬 변수를 참조할 수 없다. 반대로 내부 블록에서 외부 블록의 로컬 변수를 참조할 때는 가장 가까운 외부 블록에 정의된 로컬 변수를 참조한다. 이는 일반적인 프로그래밍 언어의 변수 적용 범위와 동일한 방식이다.

#### 제어문

- 스토어드 프로그램은 SQL문과 달리 조건 비교 및 반복과 같은 절차적인 처리를 위해 여러 가지 제어 문장을 이용할 수 있다.
- 여기서 소개하는 제어문은 역시 `BEGIN ... END ` 안에서만 사용할 수 있다.



##### IF .. ELSEIF ... ELSE ... END IF

```sql
DELIMITER ;;

CREATE FUNCTION sf_greatest(p_value1 INT, p_value2 INT)
	RETURNS INT
BEGIN
	IF p_value1 IS NULL THEN
		RETURN p_value2;
	ELSEIF p_value2 IS NULL THEN
		RETURN p_value1;
	ELSEIF p_value >= p_value2 THEN
		RETURN p_value1;
	ELSE
		RETURN p_value2;
	END IF;
END;;

DELIMITER ;

-- 버전이 올라가면서 아래 설정 기본값이 OFF로 되어있는데 ON으로 바꿔주면 스토어드 함수를 생성할 수 있다.
SET GLOBAL log_bin_trust_function_creators = 1;
```

- 일반적인 프로그래밍 언어의 포맷과 동일하게 사용할 수 있다.

##### CASE WHEN ... THEN ... ELSE ... END CASE

```sql
DELIMITER ;;

CREATE FUNCTION sf_greatest1 (p_value1 INT, p_value2 INT)
	RETURNS INT
BEGIN
	CASE 
		WHEN p_value1 IS NULL THEN
      RETURN p_value2;
    WHEN p_value2 IS NULL THEN
      RETURN p_value1;
    WHEN p_value >= p_value2 THEN
      RETURN p_value1;
    ELSE
      RETURN p_value2;
  END CASE;
END;;

DELIMITER ;
```

- `CASE WHEN` 또한 일반적인 프로그래밍 언어의 포맷과 동일하게 사용 가능하다.





##### 반복 루프

- 반복 루프 처리를 위해서 `LOOP`, `REPEAT`, `WHILE` 구문을 사용할 수 있다.

`LOOP`

```sql
DELIMITER ;;

CREATE FUNCTION sf_factorial1 (p_max INT)
	RETURNS INT
BEGIN
	DECLARE v_factorial INT DEFAULT 1;
	
	factorial_loop : LOOP
		SET v_factorial = v_factorial * p_max;
		SET p_max = p_max - 1;
		IF p_max <= 1 THEN
			LEAVE factorial_loop;
		END IF;
	END LOOP;
	
	RETURN v_factorial;
END ;;

DELIMITER ;

```

- `LOOP` 문장 자체는 비교 조건이 없고 무한 루프를 실행한다는 점에 주의해야 한다.
- `LOOP` 를 벗어나고자 할 때는 `LEAVE` 명령을 사용해서 벗어나야 한다.

`REPEAT`

```sql
DELIMITER ;;

CREATE FUNCTION sf_factorial2 (p_max INT)
	RETURNS INT
BEGIN
	DECLARE v_factorial INT DEFAULT 1;
	
	REPEAT
		SET v_factorial = v_factorial * p_max;
		SET p_max = p_max - 1;
	UNTIL p_max<=1 END REPEAT;
	
	RETURN v_factorial;
END ;;

DELIMITER ;

```

- 반복문을 최소 한번 수행하는 `REPEAT`
- `do-while` 문이랑 비슷하다.

`WHILE`

```sql
DELIMITER ;;

CREATE FUNCTION sf_factorial3 (p_max INT)
	RETURNS INT
BEGIN
	DECLARE v_factorial INT DEFAULT 1;
	
	WHILE p_max>1 DO
		SET v_factorial = v_factorial * p_max;
		SET p_max = p_max - 1;
	END WHILE;
	
	RETURN v_factorial;
END ;;

DELIMITER ;
```

- 일반적인 프로그래밍 언어의 `while` 문과 동일하다.

#### 핸들러와 컨디션을 이용한 에러 핸들링

- 안정적이고 견고한 스토어드 프로그램을 작성하려면 반드시 핸들러를 이용해 예외를 처리해야 한다.
- 핸들러가 없다면 `try-catch`가 없는 자바 프로그램과 같다고 볼 수 있다.
- 핸들러는 이미 정의한 컨디션 또는 사용자가 정의한 컨디션을 어떻게 처리할지 정의하는 기능이다.
- 핸들러를 이해하려면 MySQL에서 사용하는 `SQLSTATE`와 에러 번호의 의미와 관계를 알고 있어야 한다.

##### SQLSTATE와 에러 번호(Error NO)

```sql
mysql> SELECT * FROM not_found_table;
ERROR 1146 (42S02): Table 'employees.not_found_table' doesn't exist

-- ERROR (ERROR-NO) (SQL-STATE) : (ERROR-MSG) 형태
```

`ERROR-NO`

- 4자리(현재까지는) 숫자값으로 구성된 에러 코드
- MySQL에서만 유효한 에러 식별 번호
- `1146` 이라는 코드는 '테이블이 존재하지 않는다' 라는 것을 의미하지만 다른 DBMS와 호환되는 에러 코드는 아니다.
- 그외 기타 다양한 에러코드들은 공홈에서 확인 가능하다 [https://dev.mysql.com/doc/mysql-errors/8.0/en/server-error-reference.html](https://dev.mysql.com/doc/mysql-errors/8.0/en/server-error-reference.html)

`SQL-STATE`

- 다섯 글자의 알파벳과 숫자로 구성되고 에러 뿐만 아니라 여러 가지 상태를 의미하는 코드다.
- 이 값은 DBMS 종류가 다르더라도 ANSI SQL 표준을 준수하는 DBMS에서는 모두 똑같은 값과 의미를 가진다.
- 대부분의 MySQL 에러 번호는 특정 `SQL STATE` 값과 매핑되어 있고 매핑되지 않은 `Error No`는 `SQL STATE` 값이 `HY000` 으로 설정된다.
- `SQLSTATE` 값의 앞 2글자 의미
  - `00`: 정상 처리됨(에러 아님)
  - `01`: 경고 메시지
  - `02`: Not Found (`SELECT`나 `CURSOR`에서 결과가 없는 경우에만)
  - 그 이외의 값은 DBMS별로 할당된 각자의 에러 케이스를 의미한다.
- 수많은 `SQL STATE`는 [https://en.wikipedia.org/wiki/SQLSTATE](https://en.wikipedia.org/wiki/SQLSTATE) 에서 확인 가능하다.

`예외 핸들링에서 주의해야할사항`

| ERROR NO | SQL STATE | ERROR NAME    | 설명                                                   |
| -------- | --------- | ------------- | ------------------------------------------------------ |
| 1022     | 23000     | ER_DUP_KEY    | 프라이머리 키 또는 유니크 키 중복 에러 (NDB 클러스터)  |
| 1062     | 23000     | ER_DUP_ENTRY  | 프라이머리 키 또는 유니크 키 중복 에러(InnoDB, MyISAM) |
| 1169     | 23000     | ER_DUP_UNIQUE | 유니크 키 중복 에러(NDB 클러스터)                      |

- 똑같은 에러라고 해도 스토리지 엔진 별로 다른 에러 번호를 가질 수 있다.
- 똑같은 원인에 대해 여러 개의 에러 번호를 가지는 경우 에러 번호 중 하나라도 빠뜨리면 제대로 에러 핸들링을 못할 수 있다.
- 이 경우 `SQL STATE` 값을 사용해서 한번에 핸들링할 수 있다.
- 이뿐만 아니라 다른 에러도 이렇게 중복된 에러 번호를 갖는 경우가 있기 때문에 에러 번호보다는 `SQL STATE`를 핸들러에 사용하는게 좋다.

##### 핸들러

- 스토어드 프로그램 또한 다른 프로그래밍 언어와 같이 여러 가지 에러나 예외 상황에 대한 핸들링이 필수적이다.
- MySQL 스토어드 프로그램에서는 `DECLARE ... HANDLER` 구문을 이용해 예외를 핸들링한다.

```sql
DECLARE handler_type HANDLER
FOR condition_value [, condition_value] ... handelr_statements
```



`handler_type`

- `handler_type`이 `CONTINUE`로 정의되면 `handelr_statements` 를 실행하고 스토어드 프로그램의 마지막 실행 지점으로 다시 돌아가서 나머지 코드를 처리한다.
- `handler_type`이 `EXIT`으로 정의되면 `handelr_statements` 을 실행하고 `BEGIN ... END` 블록을 벗어난다. `EXIT` 핸들러가 정의된다면 이 핸들러의 `handelr_statements` 부분에 함수의 반환 타입에 맞는 적절한 값을 반환하는 코드가 반드시 포함되어 있어야 한다.

`condition_value`

- `condition_value` 에 `SQLSTATE` 로 정의되면 스토어드 프로그램이 실행되는 도중 어떤 이벤트가 발생했을 때 해당 이벤트의 `SQLSTATE`값이 일치할 때 실행되는 핸들러를 정의할 때 사용한다.
- `condition_value` 에 `SQLWARNING` 로 정의되면 스토어드 프로그램에서 코드를 실행하던 중 경고가 발생했을 때 실행되는 핸들러를 정의할 때 사용한다. `SQLWARNING` 키워드는 `SQLSTATE` 값이 "01"로 시작하는 이벤트를 의미한다.
- `condition_value` 에 `NOT FOUND` 로 정의되면 `SELECT` 쿼리 문의 결과 건수가 1건도 없거나 `CURSOR`의 레코드를 마지막까지 읽은 뒤에 실행하는 핸들러를 정의할 때 사용한다. `NOT FOUND` 키워드는 `SQLSTATE` 값이 "02"로 시작하는 이벤트를 의미한다.
- `condition_value` 에 `SQLEXCEPTION` 로 정의되면 경고와 `NOT FOUND`, "00"(정상처리)로 시작하는 `SQLSTATE` 이외의 모든 케이스를 의미하는 키워드다
- MySQL의 에러 코드 값을 직접 명시할 때도 있다. 코드 실행 중 어떤 이벤트가 발생했을 때 `SQLSTATE` 값이 아닌 MySQL의 에러 번호 값을 비교해서 실행되는 핸들러를 정의할 때 사용된다.
- 사용자 정의 `CONDITION`을 생성하고 그 `CONDITION`의 이름을 명시할 수도 있다. 이때는 스토어드 프로그램에서 발생한 이벤트가 정의된 컨디션과 일치하면 핸들러의  처리 내용이 수행된다.
- `condition_value`는 구분자를 이용해 여러 개를 동시에 나열할 수도 있다.
- 값이 "00000"인 `SQLSTATE`와 에러 번호 "0"은 모두 정상적으로 처리됐음을 의미하는 값이라서 `condition_value`에 사용해서는 안된다.

`handelr_statements`

- `handelr_statements`에는 특정 이벤트가 발생했을 때 그 이벤트에 대한 처리 코드를 정의한다.
- `handelr_statements`에는 단순히 명령문 하나만 사용할 수도 있고 `BEGIN ... END`로 감싸서 여러 명령문이 포함된 블록으로 작성할 수도 있다.



<br/>

```sql
DECLARE CONTINUE HANDLER FOR SQLEXCEPTION SET error_flag = 1;
```

- 위 핸들러는 `SQLEXCEPTION`(SQLSTATE가 "00", "01", "02" 이외의 값으로 시작되는 에러)이 발생했을 때 `error_flag` 로컬 변수의 값을 1로 설정하고 마지막으로 실행했던 스토어드 프로그램의 코드로 돌아가서 계속 실행하게 하는 핸들러다.

```sql
DECLARE EXIT HANDLER FOR SQLEXCEPTION
	BEGIN
		ROLLBACK;
		SELECT 'Error occurred - terminating';
	END ;;
```

- 위 핸들러는 `SQLEXCEPTION` 이 발생했을 때  `BEGIN ... END` 을 실행하고 에러가 발생한 코드가 포함된  `BEGIN ... END`을 벗어난다.
- 에러가 발생한 코드가 스토어드 프로그램의 최상위  `BEGIN ... END`이라면 스토어드 프로그램은 종료된다.
- 특별히 스토어드 프로시저에는 위의 예제처럼 결과를 읽거나 사용하지 않는 SELECT 쿼리가 실행되면 MySQL 서버가 이 결과를 즉시 클라이언트로 전송하기 때문에 디버깅 용도로 사용할 수 있지만 스토어드 함수, 트리거, 이벤트에서는 이런 기능을 사용할 수 없다.

```sql
DECLARE CONTINUE HANDLER FOR 1022, 1062 SELECT 'Duplicate ky in index' -- ERROR NO

DECLARE CONTINUE HANDLER FOR 23000 SELECT 'Duplicate ky in index' -- SQLSTATE
```

- 위 핸들러는 에러 번호가 1022나 1062인 예외가 발생했을 때 클라이언트로 'Duplicate ky in index' 메시지를 출력하고 스토어드 프로그램의 원래 실행 지점으로 돌아가서 나머지 코드를 실행한다.
- 하지만 에러 번호보다는 아래와 같이 SQLSTATE 값을 명시하는 핸들러를 사용하는 것이 좀 더 견고한 스토어드 프로그램을 만드는 방법이다.



##### 컨디션

- MySQL의 핸들러는 어떤 조건이 발생했을 때 실행할지를 명시하는 여러가지 방법이 있는데 그 중 하나가 컨디션이다.
- 단순히 에러 번호나 `SQLSTATE` 값 만으로 어떤 조건을 의미하는지 이해하기 어렵기 때문에 이 숫자들이 어떤 의미인지 예측할 수 있는 이름을 만들어 두면 훨씬 더 쉽게 코드를 이해할 수 있다.
- 바로 이러한 조건의 이름을 등록하는 것이 컨디션이다. 위에서 본 `SQLWARNING`이나 `SQLEXCEPTION` 등등 모두 MySQL 서버가 내부적으로 미리 정의해둔 컨디션이라고 볼 수 있다.

```sql
DECLARE condition_name CONDITION FOR condition_value
```

`condition_name`

- `condition_name` 사용자가 부여하려는 이름을 단순 문자열로 입력하면 된다.

`condition_value`

- `condition_value`에 MySQL의 에러 번호를 사용할 때는 `condition_value`에 바로 MySQL의 에러 번호를 입력하면 된다. 에러 코드의 값을 여러개 동시에 명시할 수 없다.
- `condition_value` 에 `SQLSTATE`를 명시하는 경우에는 `SQLSTATE` 키워드를 입력하고 그 뒤에 `SQLSTATE` 값을 입력하면 된다.

```sql
DECLARE no_such_table CONDITION FOR 1051; -- ERROR NO

DECLARE no_such_table CONDITION FOR SQLSTATE '42S02'; -- SQLSTATE
```



##### 컨디션을 사용하는 핸들러 정의

```sql
CREATE FUNCTION sf_testfunc()
	RETURNS BIGINT
BEGIN
	DECLARE dup_key CONDITION FOR 1062;
	DECLARE EXIT HANDLER FOR dup_key
		BEGIN
			RETURN -1;
		END;
		
	INSERT INTO tb_test VALUES (1);
	RETURN 1;
END ;;
```

 

#### 시그널을 이용한 예외 발생

- MySQL의 스토어드 프로그램에서 사용자가 직접 예외나 에러를 발생시키려면 시그널명령을 사용해야 한다.
- 자바와 비교한다면 핸들러는 `catch` 구문, 시그널은 `throw` 구문에 해당하는 기능 정도로 볼 수 있다.
- 시그널 구분은 MySQL 5.5부터 지원된 기능인데 이전에는 아래와 같이 존재하지 않는 테이블을 `SELECT` 한다거나 존재하지 않는 스토어드 프로그램을 호출하는 형태로 작성해서 사용했다.

```sql
CREATE FUNCTION sf_divide_old_style (p_dividend INT, p_divisor INT)
	RETURNS INT
BEGIN
	IF p_divisor IS NULL THEN
		CALL __undef_procedure_divisor_isnull(); -- 에러를 강제로 발생시켜 스토어드 프로그램을 종료
	ELSEIF p_divisor = 0 THEN
		CALL __undef_procedure_divisor_is_0(); -- 에러를 강제로 발생시켜 스토어드 프로그램을 종료
	ELSEIF p_dividend IS NULL THEN
		RETURN 0;
	END IF;
	
	RETURN FLOOR(p_dividend / p_divisor);
END ;;
```



##### 스토어드 프로그램의 BEGIN ... END 블록에서 SIGNAL 사용

```sql
DELIMITER ;;

CREATE FUNCTION sf_divide (p_dividend INT, p_divisor INT) 
         RETURNS INT
         DETERMINISTIC
         SQL SECURITY INVOKER
       BEGIN
         DECLARE null_divisor CONDITION FOR SQLSTATE '45000';
         
         IF p_divisor IS NULL THEN
           SIGNAL null_divisor 
               SET MESSAGE_TEXT='Divisor can not be null', MYSQL_ERRNO=9999;
         ELSEIF p_divisor=0 THEN
           SIGNAL SQLSTATE '45000' 
               SET MESSAGE_TEXT='Divisor can not be 0', MYSQL_ERRNO=9998;
         ELSEIF p_dividend IS NULL THEN
           SIGNAL SQLSTATE '01000' 
               SET MESSAGE_TEXT='Dividend is null, so regarding dividend as 0', MYSQL_ERRNO=9997; 
         END IF;
         
         RETURN FLOOR(p_dividend / p_divisor); 
       END;;

DELIMITER 
```

- `SIGNAL` 명령은 직접 `SQLSTATE` 값을 가질수도 있고 간접적으로 `SQLSTATE`를 가지는 컨디션을 참조해서 에러나 경고를 발생시킬 수도 있다.
- 중요한 것은 항상 `SIGNAL` 명령은 `SQLSTATE`와 직접 또는 간접적으로 연결돼야 한다는 점이다.
- `null_divisor` 의 경우 미리 위에서 정의된 컨디션을 사용해서 간접적으로 `SQLSTATE`와 연결되었다.
- 마지막 `SIGNAL SQLSTATE '01000'` 은 에러가아닌 경고를 나타내는 시그널이다. 경고를 발생시키면 경고 메시지만 출력하고 `RETURN` 문에 해당하는 값이 호출자에게 반환된다.
- `SIGNAL` 명령은 위에서 확인한 것과 같이 에러나 경고가 문법상 아무런 차이가 없다. `SQLSTATE` 값이 에러가 될지 경고가 될지 구분된다. 

![84](https://github.com/sup2is/dev-note/blob/master/db/mysql/images/real-mysql-8/84.png)

> `SELECT sf_divide(NULL, 1);` 의 경우 `SQLSTATE`값이  '01000' 이기 때문에 쿼리가 끝나면 결과와 함께 warning이 나와야 정상인데(책에는 나와있음) 안나와서 검색해보니 아래 글 내용이 있었다. 
>
> [https://bugs.mysql.com/bug.php?id=87442](https://bugs.mysql.com/bug.php?id=87442)
>
> 결론부터 말하면 MySQL 5.5 에서는 위 예제에서 warning이 정상적으로 동작하지만 5.6 이상 버전부터는 `RETURN` 문이 진단영역?(warning 영역?)을 지우기 때문에 스토어드 함수에서는 warning을 받을 수 없다는 내용이 있는 것 같다.
>
> 스토어드 프로시저는 warning이 잘 표시된다.
>
> ```sql
> DELIMITER $$
> 
> CREATE FUNCTION `f`() RETURNS INT(11)
> BEGIN
> 	SIGNAL SQLSTATE '01234';  -- signal a warning
> 	RETURN 5;
> END$$
> 
> CREATE PROCEDURE `p`()
> BEGIN
> 	SIGNAL SQLSTATE '01234';
> END$$
> 
> CREATE PROCEDURE `p2`()
> BEGIN
> 	SIGNAL SQLSTATE '01234';
> 	SELECT RAND() INTO @unused;
> END$$
> 
> DELIMITER ;
> 
> -- 함수 call, warning이 안나옴
> mysql> SELECT f();
> +------+
> | f()  |
> +------+
> |    5 |
> +------+
> 1 row in set (0.00 sec)
> 
> 
> -- 프로시저 call, SHOW WARNINGS; 로 경고메시지 확인 가능
> mysql> CALL p(0);
> Query OK, 0 rows affected, 1 warning (0.01 sec)
> 
> mysql> SHOW WARNINGS;
> +---------+------+------------------------------------------+
> | Level   | Code | Message                                  |
> +---------+------+------------------------------------------+
> | Warning | 1642 | Unhandled user-defined warning condition |
> +---------+------+------------------------------------------+
> 1 row in set (0.00 sec)
> 
> ```



`SQLSTATE의 종류`

| SQLSTATE 클래스(앞 두글자) | 의미              |
| -------------------------- | ----------------- |
| "00"                       | 정상 처리됨       |
| "01"                       | 처리 중 경고 발생 |
| 그 밖의 값                 | 처리 중 오류 발생 |

- 일반적으로 사용하는 `SIGNAL`  명령은 대부분 유저 에러나 예외일 것이므로 그에 해당하는 "45" 로 시작하는 `SQLSTATE` 를 사용할 것을 권장한다.



##### 핸들러 코드에서 SIGNAL 사용

```sql
DELIMITER ;;

CREATE PROCEDURE sp_remove_user (IN p_userid INT) 
         DETERMINISTIC
         SQL SECURITY INVOKER
       BEGIN
         DECLARE v_affectedrowcount INT DEFAULT 0;
         DECLARE EXIT HANDLER FOR SQLEXCEPTION
           BEGIN
             SIGNAL SQLSTATE '45000'
               SET MESSAGE_TEXT='Can not remove user information', MYSQL_ERRNO=9999;
           END;

         -- // 사용자의 정보를 삭제
         DELETE FROM tb_user WHERE user_id=p_userid;
         -- // 위에서 실행된 DELETE 쿼리로 삭제된 레코드 건수를 확인
         -- // ROW_COUNT()는 INSERT, DELETE, UPDATE 쿼리를 통해 수행된 row를 알려줌
         SELECT ROW_COUNT() INTO v_affectedrowcount;
         -- // 삭제된 레코드 건수가 1건이 아닌 경우에는 에러 발생 
         IF v_affectedrowcount<>1 THEN
           SIGNAL SQLSTATE '45000'; 
         END IF;
       END;;
       
DELIMITER ;


mysql> CALL sp_remove_user(12);
ERROR 9999 (45000): Can not remove user information
```

- 이 스토어드 프로시저는 `tb_user` 테이블에서 레코드를 삭제하는 프로시저다. `DELETE` 문을 실행했는데 한 건도 삭제되지 않으면 에러를 발생시키기 위해 핸들러를 사용하고 있다. `SQLEXCEPTION`에 대해 `EXIT HANDLER`가 정의되어 있고 이 핸들러는 발생한 에러의 내용을 무시하고 `SQLSTATE`가 "45000"인 에러를 다시 발생시킨다.
- 이 스토어드 프로그램은 예외가 발생하면 항상 `SQLSTATE` 값("45000") 을 리턴한다.

<br>

- 스토어드 프로그램의 `SIGNAL`이나 `HANDLER` 기능은 프로그램의 가독성을 높이고 예외에 대한 핸들링을 깔끔하게 처리해줄 것이기 때문에 잘 사용한다면 유지보수 측면에서 많은 도움을 줄 수 있다.



#### 커서

- 스토어드 프로그램의 커서는 JDBC 프로그램에서 자주 사용하는 결과 셋(Result Set)이다.
- 하지만 스토어드 프로그램에서 사용하는 커서는 JDBC의 ResultSet에 비해 기능이 상당히 제약적이다.
  - 스토어드 프로그램의 커서는 전(전진) 방향 읽기만 가능하다.
  - 스토어드 프로그램에서는 커서의 칼럼을 바로 업데이트하는 것이 불가능하다.



`DBMS의 센서티브 커서와 인센서티브 커서`

- 센서티브 커서(Sensitive Cursor)
  - 일치하는 레코드에 대한 정보를 실제 레코드의 포인터만으로 유지하는 형태.
  - 센서티브 커서는 커서를 이용해 칼럼의 데이터를 변경하거나 삭제하는 것이 가능하다.
  - 칼럼의 값이 변경돼서 커서를 생성한 `SELECT` 쿼리의 조건에 더는 일치하지 않거나 레코드가 삭제되면 커서에서도 즉시 반영된다.
  - 센서티브 커서는 별도로 임시테이블로 레코드를 복사하지 않기 때문에 커서의 오픈이 빠르다.
- 인센서티 커서(Insensitive Cursor)
  - 일치하는 레코드를 별도의 임시 테이블로 복사해서 가지고 있는 형태다.
  - `SELECT`쿼리에 부합되는 결과를 우선적으로 임시 테이블로 복사해야 하기 때문에 느리다.
  - 임시테이블로 복사된 데이터를 조회하는 것이라 커서를 통해 칼럼의 값을 변경하거나 레코드를 삭제하는 작업이 불가능하다.
  - 하지만 다른 트랜잭션과의 충돌은 발생하지 않는다.



<br>

- MySQL의 스토어드 프로그램에서 정의되는 커서는 어센서티브(Asensitive) 커서인데 어센서티브(Asensitive) 커서는 센서티브 커서와 인센서티브 커서를 혼용하는 방식이다.
- MySQL에서 실제 어떤 커서를 사용하는지 알 수 없으므로 커서를 통해 칼럼을 삭제하거나 변경하는 것이 불가능하다.
- 커서는 일반적인 프로그래밍 언어에서 `SELECT` 쿼리의 결과를 사용하는 방법과 흡사하다.
- 스토어드 프로그램에서도 `SELECT` 쿼리 문장으로 커서를 정의하고, 정의된 커서를 `OPEN` 하면 실제로 쿼리가 MySQL 서버에서 실행되고 결과를 가져온다. 이렇게 오픈된 커서는 `FETCH` 명령으로 레코드 단위로 읽어서 사용할 수 있고 사용이 완료된 후에 CLOSE 명령으로 커서를 닫으면 관련 자원이 모두 해제된다.

```sql
CREATE FUNCTION sf_emp_count(p_dept_no VARCHAR(10)) 
         RETURNS BIGINT
         DETERMINISTIC
         SQL SECURITY INVOKER
       BEGIN
         /* 사원 번호가 20000보다 큰 사원의 수를 누적하기 위한 변수 */
         DECLARE v_total_count INT DEFAULT 0;
         /* 커서에 더 읽어야 할 레코드가 남아 있는지 여부를 위한 플래그 변수 */ 
         DECLARE v_no_more_data TINYINT DEFAULT 0;
         /* 커서를 통해 SELECT된 사원 번호를 임시로 담아 둘 변수 */
         DECLARE v_emp_no INTEGER;
         /* 커서를 통해 SELECT된 사원의 입사 일자를 임시로 담아 둘 변수 */ 
         DECLARE v_from_date DATE;
         /* v_emp_list라는 이름으로 커서 정의 */ 
         DECLARE v_emp_list CURSOR FOR
         SELECT emp_no, from_date FROM dept_emp WHERE dept_no=p_dept_no;
         /* 커서로부터 더 읽을 데이터가 있는지를 나타내는 플래그 변경을 위한 핸들러 */ 
         DECLARE CONTINUE HANDLER FOR NOT FOUND SET v_no_more_data = 1;

         /* 정의된 v_emp_list 커서를 오픈 */ 
         OPEN v_emp_list;
         REPEAT
           /* 커서로부터 레코드를 한 개씩 읽어서 변수에 저장 */ 
           FETCH v_emp_list INTO v_emp_no, v_from_date;
           IF v_emp_no > 20000 THEN
             SET v_total_count = v_total_count + 1; 
           END IF;
         UNTIL v_no_more_data END REPEAT;

         /* v_emp_list 커서를 닫고 관련 자원을 반납 */ 
         CLOSE v_emp_list;

         RETURN v_total_count; 
       END ;;
```

- 위 스토어드 함수는 인자로 전달한 부서의 사원 중에서 사원 번호가 20000보다 큰 사원의 수만 `employees` 테이블에서 카운트해서 반환한다. 
- 예제에서 가장 중요한 부분은 `DECLARE`명령으로 `v_emp_list`라는 커서를 정의하고 커서를 사용하기 위해 `OPEN`과 `FETCH`를 사용하고 커서의 사용이 완료되면 `CLOSE`로 커서를 닫는 부분이다.
- 추가로 커서로부터 더 읽을 데이터가 남아있는지 판단하기 위해 `HANDLER`를 사용한 점이다. 커서로부터 더 읽을 데이터가 없으면 `NOT FOUND` 이벤트가 발생하고 핸들러가 실행되면서 `v_no_more_data` 변수의 값을 1로 변경하고 원래의 위치로 다시 되돌아온다. 반복문 부분에서 `v_no_more_data`를 확인하고 1이되는 경우 `TRUE`이므로 반복문을 종료한다.
- `DECLARE` 명령으로 `CONTINUE`나 `HANDLER`, `CURSOR` 를 정의할때는 반드시 아래 순서로 정의해야 한다.
  - 로컬 변수와 `CONDITION`
  - `CURSOR`
  - `HANDLER`



## 스토어드 프로그램의 보안 옵션

- MySQL 이전 버전까지는 `SUPER`라는 권한이 스토어드 프로그램의 생성, 변경, 삭제 권한과 많이 연결되어 있었지만 8.0부터는 `SUPER` 권한을 오브젝트별 권한으로 세분화 했다.
- 8.0 버전부터는 스토어드 프로그램의 생성 및 변경 권한이 `CREATE ROUTINE`, `ALTER ROUTINE`, `EXECUTE`로 분리되고 트리거나 이벤트의 경우 `TRIGGER`, `EVENT` 권한으로 분리됐다.
- 아래는 많은 사용자가 쉽게 지나치지만 스토어드 프로그램에서 상당히 주의할 옵션에 대해 설명한다.

### DEFINER와 SQL SECURITY 옵션

- 각 스토어드 프로그램을 생성하고 실행하는 권한을 살펴보려면 우선 스토어드 프로그램의 `DEFINER`와 `SQL SECURITY` 옵션을 이해해야 한다.
- 위 옵션들을 놓치고 그냥 지나간다면 보안 관련 문제에 대한 대응 뿐 아니라 스토어드 프로그램의 실행이 제대로 되지 않을 수도 있다.

`DEFINER`

- 스토어드 프로그램이 기본적으로 가지는 옵션
- 해당 스토어드 프로그램의 소유권과 같은 의미를 지닌다.

`SQL SECURTY`

- 이 옵션은 스토어드 프로그램을 실행할 때 누구의 권한으로 실행할지 결정하는 옵션이다.
- `INVOKER` 또는 `DEFINER` 둘 중 하나로 선택할 수 있다.
- `DEFINER` 는 스토어드 프로그램을 생성한 사용자를 의미하고 `INVOKER`는 그 스토어드 프로그램을 호출한 사용자를 의미한다.

<br>

- 예제: `DEFINER`가 'user1'@'%'로 생성된 스토어드 프로그램을 'user2'@'%' 사용자가 실행한다고 가정했을때

|                                          | SQL SECURITY=DEFINER                                         | SQL SECURITY=INVOKER                                         |
| ---------------------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| 스토어드 프로그램을 실행하는 사용자 계정 | 'user1'@'%'                                                  | 'user2'@'%' <- 실제로 스토어드 프로그램을 실행하는 사용자    |
| 실행에 필요한 권한                       | user1에 스토어드 프로그램을 실행할 권한이 있어야 하며, 스토어드 프로그램 내의 각 SQL 문장이 사용하는 테이블에 대해서도 권한을 가지고 있어야 한다. | user2에 스토어드 프로그램을 실행할 권한이 있어야 하며, 스토어드 프로그램 내의 각 SQL 문장이 사용하는 테이블에 대해서도 권한을 가지고 있어야 한다. |

- `DEFINER`는 모든 스토어드 프로그램이 기본적으로 가지는 옵션이지만 `SQL SECURITY` 옵션은 스토어드 프로시저, 스토어드 함수, 뷰만 가질 수 있다.
- 트리거나 이벤트는 자동으로 `DEFINER` 로 설정되므로 항상 `DEFINER` 에 명시된 사용자의 권한으로 항상 실행된다.
- 스토어드 프로그램의 `DEFINER`와 `SQL SECURTY`를 조합해서 복잡한 권한 문제를 어느정도 해결할 수도 있다.
  - 보안이 필요한 db에서 일반사용자에게 꼭 필요한 기능은 스토어드 프로그램으로 개발하고 `DEFINER` 와 `SQL SECURTY`를 적절하게 조절하면 된다. 그럼 일반 사용자는 권한이 전혀 없지만 `DEFINER` 설정으로 인해 사실은 관리자 계정의 권한으로 스토어드 프로그램을 실행할 수 있다.
- 스토어드 프로그램은 위와 같이 보안취약점도 될 수 있기 때문에 `SQL SECURTY`는 `DEFINER` 보다는 `INVOKER`로 설정하는게 좋다.



### DETERMINISTIC과 NOT DETERMINISTIC 옵션

- 위 옵션은 보안과 관련된 옵션이 아니라 성능과 관련된 옵션이다.

`DETERMINISTIC`

- 스토어드 프로그램의 입력이 같다면 시점이나 상황에 관계없이 결과가 항상 같다를 의미하는 키워드

`NOT DETERMINISTIC`

- 입력이 같아도 시점에 따라 결과가 달라질 수도 있음을 의미하는 키워드

<br>

- 일반적으로 일회성으로 실행되는 스토어드 프로시저는 이 옵션의 영향을 거의 받지 않지만 SQL문장에서 반복적으로 호출될 수 있는 스토어드 함수는 이 영향을 많이 받으며 쿼리의 성능을 급격하게 떨어뜨리기도 한다.

```sql
DELIMITER ;;

CREATE FUNCTION sf_getdate1()
	RETURNS DATETIME
	NOT DETERMINISTIC
BEGIN
	RETURN NOW();
END ;;

CREATE FUNCTION sf_getdate2()
	RETURNS DATETIME
	DETERMINISTIC
BEGIN
	RETURN NOW();
END ;;

DELIMITER ;


-- demt_emp의 from_date는 인덱스가 걸려 있다.
EXPLAIN SELECT * FROM dept_emp WHERE from_date > sf_getdate1();
EXPLAIN SELECT * FROM dept_emp WHERE from_date > sf_getdate2();
```

![85](https://github.com/sup2is/dev-note/blob/master/db/mysql/images/real-mysql-8/85.png)

![](https://github.com/sup2is/dev-note/blob/master/db/mysql/images/real-mysql-8/86.png)

- `DETERMINISTIC`으로 정의된 `sf_getdate2()` 함수는 쿼리를 실행하기 위해 딱 한 번만 스토어드 함수를 호출하고 함수의 결과값을 상수화해서 쿼리를 실행한다.
- `NOT DETERMINISTIC` 으로 정의된 `sf_getdate1()` 함수는 `WHERE` 절이 비교를 수행하는 레코드마다 매번 값이 재평가돼야 한다. 이 경우 스토어드 함수는 절대 상수가 될 수 없다.
- 중요한 점은 `NOT DETERMINISTIC`이 스토어드 함수의 디폴트 옵션이라는 것이다. 따라서 이부분을 꼭 인지하고 스토어드 함수를 사용할 때는 `DETERMINISTIC`옵션을 주어야 한다.
- 위에서 본것같은 `sf_getdate1()` 처럼 레코드를 매번 새로운값으로 비교해야 하는 경우는 일반적인 애플리케이션에서 거의 없는 상황이다.



## 스토어드 프로그램의 참고 및 주의사항

### 한글 처리

- 스토어드 프로그램의 소스코드에 한글 문자열 값을 사용해야 한다면 스토어드 프로그램을 생성하는 클라이언트 프로그램이 어떤 문자 집합으로 MySQL 서버에 접속돼 있는지가 중요해진다.

```sql
SHOW VARIABLES LIKE 'character%';

+--------------------------+------------------------------------------------------+
| Variable_name            | Value                                                |
+--------------------------+------------------------------------------------------+
| character_set_client     | utf8mb4                                              |
| character_set_connection | utf8mb4                                              |
| character_set_database   | utf8mb4                                              |
| character_set_filesystem | binary                                               |
| character_set_results    | utf8mb4                                              |
| character_set_server     | utf8mb4                                              |
| character_set_system     | utf8mb3                                              |
| character_sets_dir       | /usr/local/Cellar/mysql/8.0.28/share/mysql/charsets/ |
+--------------------------+------------------------------------------------------+
8 rows in set (0.02 sec)

```

- 스토어드 프로그램이 관여하는 부분은 `character_set_client`과 `character_set_connection` 이다.
- 이 값들이 `latin1` 처럼 다른 값이라면 `utf8mb4` 라는 값으로 설정해주면 된다.

```sql
SET character_set_client = 'utf8mb4'
SET character_set_connection = 'utf8mb4'
SET character_set_results = 'utf8mb4'

-- 위 세개를 한방에! 하지만 커넥션 재접속했을 때는 리셋된다.
SET NAMES 'utf8mb4';

-- 위 세개를 한방에! 하지만 MySQL 클라이언트가 종료되었을 때는 리셋된다.
CHARSET 'utf8mb4';

-- 이런 방법도 있다
CREATE FUNCTION sf_getstring()
	RETURNS VARCHAR(20) CHARACTER SET utf8mb4
BEGIN
	RETURN '한글 테스트';
END ;;

```

- 참고로 MySQL 설정 파일에서 기본 문자셋을 지정할 수 있고 MySQL 8.0 부터는 기본값이 `utf8mb4` 이다.

### 스토어드 프로그램과 세션 변수

- 스토어드 프로그램에서 로컬변수와 세션변수는 둘 다 사용이 가능하다.

```sql
CREATE FUNCTION sf_getsum(p_arg1 INT, p_arg2 INT)
	RETURNS INT
BEGIN
	DECLARE v_sum INT DEFAULT 0;
	SET v_sum = p_arg1 + p_arg2;
	SET @v_sum = v_sum;
	
	RETURN v_sum;
END;;
```

- 세션 변수는 타입이 없기 때문에 데이터 타입에 안전하지 않고 영향을 미치는 범위가 로컬 변수보다 높다. 또한 세션에서 계속 남아 있기 때문에 사용하기 전에 적절하게 초기화하고 사용해야한다.
- 이런 이유때문에 가능하다면 사용자 변수보다는 스토어드 프로그램의 로컬 변수를 사용하는게 좋다.
- 스토어드 프로그램에서 프리페어 스테이트먼트를 실행하려면 세션 변수를 사용할 수밖에 없는데 이런 경우가 아니라면 가능한 한 세션 변수보다는 스토어드 프로그램의 로컬 변수를 사용하자.



### 스토어드 프로시저와 재귀 호출

- 스토어드 프로그램에서도 재귀호출을 사용할 수 있는데 이는 스토어드 프로시저에서만 사용 가능하며 스토어드 함수, 트리거, 이벤트에서는 사용할 수 없다.
- 프로그래밍 언어에서 발생할 수 있는 문제처럼 무한 반복된다면 스택오버플로우가 발생할 수도 있다.
- MySQL에서는 이러한 재귀 호출의 문제를 막기 위해 최대 몇 번까지 재귀 호출을 허용할지를 설정하는 `max_sp_recursion_depth` 라는 시스템 변수가 있다. (기본값 0) 이 값을 변경하지 않으면 MySQL에서 재귀호출을 사용할 수 없다.
- 아래 프로시저와 같이 필요한 설정 값을 스토어드 프로시저의 내부에서 변경하는 것도 좋은 방법이다.

```sql
CREATE PROCEDURE sp_getfactorial(IN p_max INT, OUT p_sum INT)
BEGIN
	SET max_sp_recursion_depth=50; -- 최대 재귀 호출 횟수 50회
	
	...
	
END ;;
```



### 중첩된 커서 사용

- 일반적으로는 하나의 커서를 열고 사용이 끝나면 닫고 다시 필요하다면 새로운 커서를 여는 방식을 많이 사용하지만 중첩된 루프 안에서 두 개의 커서를 동시에 열어서 사용할 때도 있다.

```sql
CREATE PROCEDURE sp_updateemployeehiredate() 
         DETERMINISTIC
         SQL SECURITY INVOKER
       BEGIN
         -- // 첫 번째 커서로부터 읽은 부서 번호를 저장 
         DECLARE v_dept_no CHAR(4);
         -- // 두 번째 커서로부터 읽은 사원 번호를 저장 
         DECLARE v_emp_no INT;
         -- // 커서를 끝까지 읽었는지 여부를 나타내는 플래그를 저장 
         DECLARE v_no_more_rows BOOLEAN DEFAULT FALSE;
         -- // 부서 정보를 읽는 첫 번째 커서 
         DECLARE v_dept_list CURSOR FOR SELECT dept_no FROM departments;

         -- // 부서별 사원 1명을 읽는 두 번째 커서 
         DECLARE v_emp_list CURSOR FOR SELECT emp_no 
                                       FROM dept_emp 
                                       WHERE dept_no=v_dept_no LIMIT 1;

         -- // 커서의 레코드를 끝까지 다 읽은 경우에 대한 핸들러
         DECLARE CONTINUE HANDLER FOR NOT FOUND SET v_no_more_rows := TRUE;

         OPEN v_dept_list;
         LOOP_OUTER: LOOP 
           -- // 외부 루프 시작 ❶ 
           FETCH v_dept_list INTO v_dept_no;
           IF v_no_more_rows THEN 
             CLOSE v_dept_list; 
             LEAVE loop_outer;
           END IF;

           OPEN v_emp_list;
           LOOP_INNER: LOOP
             FETCH v_emp_list INTO v_emp_no;
             -- // 레코드를 모두 읽었으면 커서 종료 및 내부 루프를 종료 
             IF v_no_more_rows THEN
               -- // 반드시 no_more_rows를 FALSE로 다시 변경해야 한다.
               -- // 그렇지 않으면 내부 루프 때문에 외부 루프까지 종료돼 버린다. 
               SET v_no_more_rows := FALSE;
               CLOSE v_emp_list;
               LEAVE loop_inner;
             END IF;
           END LOOP loop_inner;
         END LOOP loop_outer; 
       END;;
```

- `no_more_rows` 라는 로컬 변수로 두 개의 반복 루프를 제어하다보니 이런 이해하기 힘든 로직이 나올 수 있다.
- 이처럼 반복 루프가 여러번 중첩되어 커서가 사용될 때는 `LOOP_OUTER` 와 `LOOP_INNER`를 서로 다른 `BEGIN ... END` 블록으로 구분해서 작성하면 쉽고 깔끔하게 해결할 수 있다.

```sql
CREATE PROCEDURE sp_updateemployeehiredate1()
         DETERMINISTIC
         SQL SECURITY INVOKER
       BEGIN
         -- // 첫 번째 커서로부터 읽은 부서 번호를 저장 
         DECLARE v_dept_no CHAR(4);
         -- // 커서를 끝까지 읽었는지를 나타내는 플래그를 저장 
         DECLARE v_no_more_depts BOOLEAN DEFAULT FALSE;
         -- // 부서 정보를 읽는 첫 번째 커서 
         DECLARE v_dept_list CURSOR FOR SELECT dept_no FROM departments;
         -- // 부서 커서의 레코드를 끝까지 다 읽은 경우에 대한 핸들러 
         DECLARE CONTINUE HANDLER FOR NOT FOUND SET v_no_more_depts := TRUE;
       
         OPEN v_dept_list;
         LOOP_OUTER: LOOP                    -- // 외부 루프 시작
           FETCH v_dept_list INTO v_dept_no;
           IF v_no_more_depts THEN 
              -- // 레코드를 모두 읽었으면 커서 종료 및 외부 루프 종료
             CLOSE v_dept_list;
             LEAVE loop_outer; 
           END IF;
       
           BLOCK_INNER: BEGIN -- // 내부 프로시저 블록 시작
             -- // -----------------------------------------------------------------
             -- // 두 번째 커서로부터 읽은 사원 번호 저장 
             DECLARE v_emp_no INT;
             -- // 사원 커서를 끝까지 읽었는지 여부를 위한 플래그 저장 
             DECLARE v_no_more_employees BOOLEAN DEFAULT FALSE;
             -- // 부서별 사원 1명을 읽는 두 번째 커서 
             DECLARE v_emp_list CURSOR FOR SELECT emp_no 
                                           FROM dept_emp 
                                           WHERE dept_no=v_dept_no LIMIT 1;
             -- // 사원 커서의 레코드를 끝까지 다 읽은 경우에 대한 핸들러
             DECLARE CONTINUE HANDLER FOR NOT FOUND SET v_no_more_employees := TRUE;

             OPEN v_emp_list;
             LOOP_INNER: LOOP              -- // 내부 루프 시작
               FETCH v_emp_list INTO v_emp_no;
               -- // 레코드를 모두 읽었으면 커서 종료 및 내부 루프를 종료 
               IF v_no_more_employees THEN
                 CLOSE v_emp_list;
                 LEAVE loop_inner; 
               END IF;
             END LOOP loop_inner;          -- // 내부 루프 종료
           -- // ----------------------------------------------------------------- 
           END block_inner;                -- // 내부 프로시저 블록 종료
         END LOOP loop_outer;              -- // 외부 루프 종료 
       END;;
```

- 중첩된 커서를 각 프로시저 블록으로 작성하면 커서별로 이벤트 핸들러를 생성해서 사용할 수 있다.

