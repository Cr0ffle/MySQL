# Chapter 15. 데이터 타입

- 칼럼의 데이터 타입과 길이를 선정할 때 가장 주의해야 할 사항
    1. 저장되는 값의 성격에 맞는 최적의 타입을 선정
    2. 가변 길이 칼럼은 최적의 길이를 지정
    3. 조인 조건으로 사용되는 칼럼은 똑같은 데이터 타입을 선정

## 1. 문자열(`CHAR` 와 `VARCHAR`)

---

### 1-1. 저장 공간

---

- `CHAR` vs `VARCHAR`
    - 공통점: 문자열 저장할 수 있는 데이터 타입
    - 차이점: `CHAR`는 고정길이, `VARCHAR`는 가변길이
        - 고정길이는 실제 입력되는 칼럼 값의 길이에 따라 사용하는 저장 공간의 크기가 변하지 않는다.
            - `CHAR`타입은 이미 저장 공간의 크기가 고정적이다. 실제 저장된 값의 유효 크기가 얼미인지 별도로 저장할 필요가 없으므로 추가로 공간이 필요하지 않다.
        - 가변 길이는 최대로 저장할 수 있는 값의 길이는 제한돼 있지만, 그 이하 크기의 값이 저장되면 그만큼 저장공간이 줄어든다.
            - 하지만 `VARCHAR` 타입은 저장된 값의 유효 크기가 얼마인지를 별도로 저장해 둬야 하므로 1~2바이트의 저장 공간이 추가로 더 필요하다.
            - 255 바이트 이하dl면 1바이트만 사용하고, 타입의 길이가 256바이트 이상으로 설정되면 2바이트를 길이를 저장하는 데 사용한다 → `VARCHAR` 타입의 최대 길이는 2바이트로 표현할 수 있는 이상은 사용할 수 없다.
            
            ❗즉 `VARCHAR` 타입의 최대 길이는 65,536 바이트 이상으로 설정할 수 없다.
            
    
    👉 문자열 길이가 항상 일정하다면 `CHAR`를 사용하고 가변적이라면 `VARCHAR`를 사용하는 것이 일반적!****
    

- `CHAR` 타입과 `VARCHAR` 타입 결정 시 가장 중요한 기준
    - 저장되는 문자열의 길이가 대개 비슷한가?
    - 칼럼의 값이 자주 변경되는가?
        
        ```sql
        CREATE TABLE tb_test (
          fd1 INT,
          fd2 CHAR(10),
          fd3 DATETIME
        );
        
        INSERT INTO tb_test (fd1, fd2, fd3) VALUES (1, 'ABCD', '2011-06-27 11:02:11');
        ```
        
        - 레코드 1건 저장하면 내부적으로 디스크
            
            ![15.1 `CHAR` 타입이 저장된 상태](./images/Untitled.png)
            
            15.1 `CHAR` 타입이 저장된 상태
            
            - fd1 칼럼은 `INTEGER` 타입이므로 고정 길이로 4바이트를 사용
            - fd3 또한 `DATETIME`이므로 고정 길이로 8바이트를 시용
            - fd2 칼럼은 정확히 10바이트를 사용하면서 앞쪽의 4바이트만 유효한 값으로 채워졌고 나머지는 공백 문자로 채워져 있다.
            
        - fd2 칼럼만 `CHAR(10)` 대신 `VARCHAR(10)`으로 변경
            
            ![15.2 `VARCHAR` 타입의 칼럼이 저장된 상태](./images/Untitled%201.png)
            
            15.2 `VARCHAR` 타입의 칼럼이 저장된 상태
            
            - 첫 번째 바이트에는 저장된 칼럼 값의 유효한 바이트 수 (4)
            - 두 번째 바이트부터 다섯 번째 바이트까지 실제 칼럼 값이 저장된다.
        - fd2 칼럼의 값이 변경될 때 어떤 현상이 발생? (“ABCDE"로 `UPDATE`)
            - `CHAR(10)`타입: fd2 칼럼을 위해 공간이 10바이트가 준비돼 있으므로 그냥 변경되는 칼럼의 값을 업데이트만 하면 된다.
            - `VARCHAR(10)`타입: fd2 칼럼에 4바이트밖에 저장할 수 없는 구조로 만들어져 있다. 그래서 길이가 더 큰 값으로 변경될 때는 레코드 자체를 다른 공간으로 옮겨서(Row migration) 저장해야 한다.
        - 값이 2~3바이트씩 차이가 나더라도 지주 변경될 수 있는 부서 번호나 게시물의 상태 값 등은 `CHAR` 타입을 사용하는 것이 좋다.
            - 자주 변경돼도 레코드가 물리적으로 다른 위치로 이동시키거나 분리하지 않아도 되기 때문이다.
            - 레코드의 이동이나 분리는 `CHAR` 타입으로 인해 발생하는 2~3바이트 공간 낭비보다 더 큰 공간이나 자원을 낭비하게 만든다.

- MySQL 에서 `CHAR`나 `VARCHAR` 뒤에 지정하는 숫자는 그 칼럼의 바이트 크기가 아니라 문자의 수를 의미한다.
    - `CHAR(10)` 타입을 사용하더라도 이 칼럼이 실제로 디스크나 메모리에서 사용하는 공간은 각각 달라진다.
        - 일반적으로 영어를 포함한 서구권 언어는 각 문자가 1바이트를 사용하므로 10바이트를 사용한다.
        - 한국어나 일본어와 같은 아시아권 언어는 각 문자가 최대 2바이트를 사용하므로 20바이트를 사용한다.
        - UTF-8과 같은 유니코드는 최대 4바이트까지 사용하므로 40바이트까지 사용할 수 있다.
            - MySQL 5.5 이전 버전에선 utf8 문자셋은 한 글자당 최대 3바이트까지만 지원됐다.
                - SMP, SIP 그 이후 플레인 문자(4바이트 저장 공간 사용)는 저장 불가능
            - MySQL 5.5 부터 `utf8mb4` 지원되기 시작 → 유니코드에서 지원하는 대부분의 문자를 지원****

### 1-2. 저장 공간과 스키마 변경(Online DDL)

---

- `VARCHAR` 데이터 타입의 길이를 늘리는 작업은 경우에 따라 읽기잠금을 걸고 레코드 복사작업이 필요할 수도 있다.
    
    ```sql
    CREATE TABLE test (
        id INT PRIMARY KEY,
        value VARCHAR(60)
    ) DEFAULT CHARSET=utf8mb4;
    
    ALTER TABLE test MODIFY value VARCHAR(63), ALGORITHM=INPLACE, LOCK=NONE;
    Query OK, 0 rows affected (0.00 sec)
    Records: 0  Duplicates: 0  Warnings: 0
    
    # VARCHAR 타입의 칼럼이 가지는 길이 저장 공간의 크기 때문
    ALTER TABLE test MODIFY value VARCHAR(64), ALGORITHM=INPLACE, LOCK=NONE;
    ERROR 1846 (0A000): ALGORITHM=INPLACE is not supported. Reason: Cannot change column type INPLACE. Try ALGORITHM=COPY.
    
    ALTER TABLE test MODIFY value VARCHAR(64), ALGORITHM=COPY, LOCK=SHARED;
    Query OK, 0 rows affected (0.02 sec)
    Records: 0  Duplicates: 0  Warnings: 0
    ```
    
    - `VARCHAR(60)` 칼럼은 최대 길이가 240(60*4) 바이트이기 때문에 문자열 값의 길이를 저장하는 공간은 1바이트면 된다.
    - `VARCHAR(64)` 타입은 저장할 수 있는 문자열의 크기가 최대 256(64*4) 바이트까지 가능하기 때문에 문자열 길이를 저장하는 공간의 크기가 2바이트로 변경되어야 한다.
    
    👉 문자열 길이를 저장하는 공간의 크기가 바뀌게 되면 MySQL 서버는 스키마 변경을 하는 동안 읽기 잠금(`LOCK=SHARED`)을 걸어서 아무도 데이터를 변경하지 못하도록 막고 테이블의 레코드를 복사하는 방식으로 처리한다.
    

❗`VARCHAR` 타입의 길이가 크게 변경될 것으로 예상된다면 길이 저장 공간의 크기가 바뀌지 않도록 미리 조금 크게 설계하는 것이 좋다.****

### 1-3. 문자 집합(캐릭터 셋)

---

- 문자 집합은 문자열을 저장하는 `CHAR`, `VARCHAR`, `TEXT` 타입의 칼럼에만 설정할 수 있다.
- MySQL 에서 최종적으로는 칼럼 단위로 문자 집합을 관리하지만 관리의 편의를 위해 MySQL 서버와 DB, 그리고 테이블 단위로 기본 문자 집합을 설정할 수 있는 기능을 제공한다.
- MySQL 서버에서 사용 가능한 문자 집합은 `SHOW CHARACTER SET` 명령으로 확인해 볼 수 있다.
    
    ```sql
    SHOW CHARACTER SET;
    # Default collation: 칼럼에 콜레이션은 명시하지 않고 문자 집합만 지정했을 때 설정되는 콜레이션
    +----------+---------------------------------+---------------------+--------+
    | Charset  | Description                     | **Default collation**   | Maxlen |
    +----------+---------------------------------+---------------------+--------+
    | armscii8 | ARMSCII-8 Armenian              | armscii8_general_ci |      1 |
    | ascii    | US ASCII                        | ascii_general_ci    |      1 |
    | big5     | Big5 Traditional Chinese        | big5_chinese_ci     |      2 |
    | binary   | Binary pseudo charset           | binary              |      1 |
    | cp1250   | Windows Central European        | cp1250_general_ci   |      1 |
    | cp1251   | Windows Cyrillic                | cp1251_general_ci   |      1 |
    | cp1256   | Windows Arabic                  | cp1256_general_ci   |      1 |
    | cp1257   | Windows Baltic                  | cp1257_general_ci   |      1 |
    | cp850    | DOS West European               | cp850_general_ci    |      1 |
    | cp852    | DOS Central European            | cp852_general_ci    |      1 |
    | cp866    | DOS Russian                     | cp866_general_ci    |      1 |
    | cp932    | SJIS for Windows Japanese       | cp932_japanese_ci   |      2 |
    | dec8     | DEC West European               | dec8_swedish_ci     |      1 |
    | eucjpms  | UJIS for Windows Japanese       | eucjpms_japanese_ci |      3 |
    | euckr    | EUC-KR Korean                   | euckr_korean_ci     |      2 |
    | gb18030  | China National Standard GB18030 | gb18030_chinese_ci  |      4 |
    | gb2312   | GB2312 Simplified Chinese       | gb2312_chinese_ci   |      2 |
    | gbk      | GBK Simplified Chinese          | gbk_chinese_ci      |      2 |
    | geostd8  | GEOSTD8 Georgian                | geostd8_general_ci  |      1 |
    | greek    | ISO 8859-7 Greek                | greek_general_ci    |      1 |
    | hebrew   | ISO 8859-8 Hebrew               | hebrew_general_ci   |      1 |
    | hp8      | HP West European                | hp8_english_ci      |      1 |
    | keybcs2  | DOS Kamenicky Czech-Slovak      | keybcs2_general_ci  |      1 |
    | koi8r    | KOI8-R Relcom Russian           | koi8r_general_ci    |      1 |
    | koi8u    | KOI8-U Ukrainian                | koi8u_general_ci    |      1 |
    | latin1   | cp1252 West European            | latin1_swedish_ci   |      1 |
    | latin2   | ISO 8859-2 Central European     | latin2_general_ci   |      1 |
    | latin5   | ISO 8859-9 Turkish              | latin5_turkish_ci   |      1 |
    | latin7   | ISO 8859-13 Baltic              | latin7_general_ci   |      1 |
    | macce    | Mac Central European            | macce_general_ci    |      1 |
    | macroman | Mac West European               | macroman_general_ci |      1 |
    | sjis     | Shift-JIS Japanese              | sjis_japanese_ci    |      2 |
    | swe7     | 7bit Swedish                    | swe7_swedish_ci     |      1 |
    | tis620   | TIS620 Thai                     | tis620_thai_ci      |      1 |
    | ucs2     | UCS-2 Unicode                   | ucs2_general_ci     |      2 |
    | ujis     | EUC-JP Japanese                 | ujis_japanese_ci    |      3 |
    | utf16    | UTF-16 Unicode                  | utf16_general_ci    |      4 |
    | utf16le  | UTF-16LE Unicode                | utf16le_general_ci  |      4 |
    | utf32    | UTF-32 Unicode                  | utf32_general_ci    |      4 |
    | utf8     | UTF-8 Unicode                   | utf8_general_ci     |      3 |
    | utf8mb4  | UTF-8 Unicode                   | utf8mb4_0900_ai_ci  |      4 |
    +----------+---------------------------------+---------------------+--------+
    ```
    
    - 한국에서 MySQL 을 사용한다면 대부분 `euckr` 이나 `utf8mb4` 만으로도 충분할 것이다.
        - `euckr`: 한국어 전용으로 사용되는 문자 집합이며, 모든 글자는 1~2 바이트를 사용한다.
        - `utf8mb4`: 다국어 문자를 포함할 수 있는 칼럼에 사용하기에 적합하다.
            - 일반적으로 디스크에 저장할 때는 한 글자를 저장하기 위해 1~4 바이트까지 사용한다.
            - `utf8mb4` 문자 집합을 사용하는 문자열 값이 메모리에 기록될 때는 실제 문자열의 길이와 관계없이 문자당 4바이트로 공간이 할당되는 경우도 있다.

- MySQL 에서 설정 가능한 문자 집합 관련 변수
    - `character_set_system`
        - MySQL 서버가 식별자를 저장할 때 사용하는 문자 집합이다.
        - 기본적으로 `utf8`로 설정되며, 사용자가 설정하거나 변경할 필요가 없다.
    - `character_set_server`
        - MySQL 서버의 기본 문자 집합이다.
        - DB, 테이블, 칼럼에 아무런 문자 집합이 설정되지 않을 때 이 시스템 변수에 명시된 문자 집합이 기본으로 사용된다.(디폴트: `utf8mb4`)
    - `character_set_database`
        - MySQL DB 의 기본 문자 집합이다.
        - DB 를 생성할 때 아무런 문자 집합이 명시되지 않았다면 이 시스템 변수에 명시된 문자 집합이 기본값으로 사용된다.
        - 이 변수가 정의되지 않으면 `character_set_server` 시스템 변수에 명시된 문자 집합이 기본으로 사용된다.(디폴트: `utf8mb4`)
    - `character_set_filesystem`
        - `LOAD DATA INFILE ...` 또는 `SELECT ... INTO OUTFILE` 실행할 때 인자로 지정되는 파일의 이름을 해석할 때 사용되는 문자 집합
        - 파일의 내용을 읽을 때 사용하는 문자 집합이 아니라 파일의 이름을 찾을 때 사용하는 문자 집합! (디폴트: `binary`)
    - `character_set_client`
        - MySQL 클라이언트가 보낸 SQL 문장은 `character_set_client` 에 설정된 문자 집합으로 인코딩해서 MySQL 서버로 전송한다.
        - 이 값은 각 커넥션에서 임의의 문자 집합으로 변경해서 사용할 수 있다.(디폴트: `utf8mb4`)
    - `character_set_connection`
        - MySQL 서버가 클라이언트로부터 받은 SQL 문장을 처리하기 위해 `character_set_connection` 의 문자 집합으로 변환한다.
        - 클라이언트로부터 전달받은 숫자 값을 문자열로 변환할때도 `character_set_connection`에 설정된 문자 집합이 사용된다.(디폴트: `utf8mb4`)
    - `character_set_results`
        - MySQL 서버가 쿼리의 처리 결과를 클라이언트로 보낼때 사용하는 문자 집합을 설정하는 시스템 변수(디폴트: `utf8mb4`)

- 1️⃣ 클라이언트로 부터 쿼리 요청시 문자 집합 변환
    - MySQL 서버는 받은 문자열 데이터를 `character_set_connection` 에 정의된 문자 집합으로 변환한다.
    - 인트로듀서: SQL 문장에서 별도로 문자 집합을 설정하는 지정자
        - 문자열 리터럴 앞에 언더스코어 기호(`_`)와 문자 집합의 이름을 붙여서 표현한다.
        
        ```sql
        SELECT emp_no, first_name FROM employees WHERE first_name = _latin1'Matt';
        ```
        
- 2️⃣ 처리 결과를 클라이언트로 전송할 때의 문자 집합 변환
    - MySQL 서버는 쿼리의 결과를 `character_set_results` 변수에 설정된 문자 집합으로 변환해 클라이언트로 전송한다.
        - 결과 셋에 포함된 칼럼의 값이나 칼럼명과 같은 메타데이터도 모두 `character_set_results`로 인코딩되어 클라이언트로 전송된다.
    - 변환전 문자 집합과 변환해야 할 문자 집합이 똑같다면 별도의 문자 집합 변환 작업은 모두 생략한다.

### 1-4. 콜레이션(Collation)

---

- 콜레이션은 문자열 칼럼의 값에 대한 비교나 정렬 순서를 위한 규칙을 의미한다.
    - 비교나 정렬 작업 에서 영문 대소문자를같은 것으로 처리할지 아니면 더 크거나 작은 것으로 판단할지에 대한 규칙을 정의하는 것이다.
    - 문자열 칼럼의 값을 비교하거나 정렬하는 기준이 된다.
        
        → 각 문자열 칼럼의 값을 비교하거나 정렬할 때는 항상 문자집합뿐 아니라 콜레이션의 일치 여부에 따라 결과가 달라지며, 쿼리의 성능 또한 상당한 영향을 받는다.
        
- 1️⃣ 콜레이션 이해
    - 문자 집합은 2개 이상의 콜레이션을 가지고 있는데, 하나의 문자 집합에 속한 콜레이션은 다른 문자집합과 공유해서 사용할 수 없다.
    - 테이블이나 칼럼에 문자 집합만 지정하면 해당 문자 집합의 디폴트 콜레이션이 해당 칼럼의 콜레이션으로 지정된다.
        - 반대로 칼럼의 콜레이션만 지정하면, 그 콜레이션이 소속된 문자 집합이 묵시적으로 그 칼럼의 문자 집합으로 사용된다.
    - `SHOW COLLATION` 명령: MySQL 서버에서 시용 가능한 콜레이션의 목록 확인
        
        ```sql
        SHOW COLLATION;
        +----------------------------+----------+-----+---------+----------+---------+---------------+
        | Collation                  | Charset  | Id  | Default | Compiled | Sortlen | Pad_attribute |
        +----------------------------+----------+-----+---------+----------+---------+---------------+
        | armscii8_bin               | armscii8 |  64 |         | Yes      |       1 | PAD SPACE     |
        | armscii8_general_ci        | armscii8 |  32 | Yes     | Yes      |       1 | PAD SPACE     |
        | ascii_bin                  | ascii    |  65 |         | Yes      |       1 | PAD SPACE     |
        | ascii_general_ci           | ascii    |  11 | Yes     | Yes      |       1 | PAD SPACE     |
        | big5_bin                   | big5     |  84 |         | Yes      |       1 | PAD SPACE     |
        | big5_chinese_ci            | big5     |   1 | Yes     | Yes      |       1 | PAD SPACE     |
        | binary                     | binary   |  63 | Yes     | Yes      |       1 | NO PAD        |
        | cp1250_bin                 | cp1250   |  66 |         | Yes      |       1 | PAD SPACE     |
        | cp1250_croatian_ci         | cp1250   |  44 |         | Yes      |       1 | PAD SPACE     |
        | cp1250_czech_cs            | cp1250   |  34 |         | Yes      |       2 | PAD SPACE     |
        | cp1250_general_ci          | cp1250   |  26 | Yes     | Yes      |       1 | PAD SPACE     |
        | cp1250_polish_ci           | cp1250   |  99 |         | Yes      |       1 | PAD SPACE     |
        | cp1251_bin                 | cp1251   |  50 |         | Yes      |       1 | PAD SPACE     |
        | cp1251_bulgarian_ci        | cp1251   |  14 |         | Yes      |       1 | PAD SPACE     |
        | cp1251_general_ci          | cp1251   |  51 | Yes     | Yes      |       1 | PAD SPACE     |
        | cp1251_general_cs          | cp1251   |  52 |         | Yes      |       1 | PAD SPACE     |
        | cp1251_ukrainian_ci        | cp1251   |  23 |         | Yes      |       1 | PAD SPACE     |
        | cp1256_bin                 | cp1256   |  67 |         | Yes      |       1 | PAD SPACE     |
        | cp1256_general_ci          | cp1256   |  57 | Yes     | Yes      |       1 | PAD SPACE     |
        | cp1257_bin                 | cp1257   |  58 |         | Yes      |       1 | PAD SPACE     |
        | cp1257_general_ci          | cp1257   |  59 | Yes     | Yes      |       1 | PAD SPACE     |
        | cp1257_lithuanian_ci       | cp1257   |  29 |         | Yes      |       1 | PAD SPACE     |
        | cp850_bin                  | cp850    |  80 |         | Yes      |       1 | PAD SPACE     |
        | cp850_general_ci           | cp850    |   4 | Yes     | Yes      |       1 | PAD SPACE     |
        | cp852_bin                  | cp852    |  81 |         | Yes      |       1 | PAD SPACE     |
        | cp852_general_ci           | cp852    |  40 | Yes     | Yes      |       1 | PAD SPACE     |
        | cp866_bin                  | cp866    |  68 |         | Yes      |       1 | PAD SPACE     |
        | cp866_general_ci           | cp866    |  36 | Yes     | Yes      |       1 | PAD SPACE     |
        | cp932_bin                  | cp932    |  96 |         | Yes      |       1 | PAD SPACE     |
        | cp932_japanese_ci          | cp932    |  95 | Yes     | Yes      |       1 | PAD SPACE     |
        | dec8_bin                   | dec8     |  69 |         | Yes      |       1 | PAD SPACE     |
        | dec8_swedish_ci            | dec8     |   3 | Yes     | Yes      |       1 | PAD SPACE     |
        | eucjpms_bin                | eucjpms  |  98 |         | Yes      |       1 | PAD SPACE     |
        | eucjpms_japanese_ci        | eucjpms  |  97 | Yes     | Yes      |       1 | PAD SPACE     |
        | euckr_bin                  | euckr    |  85 |         | Yes      |       1 | PAD SPACE     |
        | euckr_korean_ci            | euckr    |  19 | Yes     | Yes      |       1 | PAD SPACE     |
        | gb18030_bin                | gb18030  | 249 |         | Yes      |       1 | PAD SPACE     |
        | gb18030_chinese_ci         | gb18030  | 248 | Yes     | Yes      |       2 | PAD SPACE     |
        | gb18030_unicode_520_ci     | gb18030  | 250 |         | Yes      |       8 | PAD SPACE     |
        | gb2312_bin                 | gb2312   |  86 |         | Yes      |       1 | PAD SPACE     |
        | gb2312_chinese_ci          | gb2312   |  24 | Yes     | Yes      |       1 | PAD SPACE     |
        | gbk_bin                    | gbk      |  87 |         | Yes      |       1 | PAD SPACE     |
        | gbk_chinese_ci             | gbk      |  28 | Yes     | Yes      |       1 | PAD SPACE     |
        | geostd8_bin                | geostd8  |  93 |         | Yes      |       1 | PAD SPACE     |
        | geostd8_general_ci         | geostd8  |  92 | Yes     | Yes      |       1 | PAD SPACE     |
        | greek_bin                  | greek    |  70 |         | Yes      |       1 | PAD SPACE     |
        | greek_general_ci           | greek    |  25 | Yes     | Yes      |       1 | PAD SPACE     |
        | hebrew_bin                 | hebrew   |  71 |         | Yes      |       1 | PAD SPACE     |
        | hebrew_general_ci          | hebrew   |  16 | Yes     | Yes      |       1 | PAD SPACE     |
        | hp8_bin                    | hp8      |  72 |         | Yes      |       1 | PAD SPACE     |
        | hp8_english_ci             | hp8      |   6 | Yes     | Yes      |       1 | PAD SPACE     |
        | keybcs2_bin                | keybcs2  |  73 |         | Yes      |       1 | PAD SPACE     |
        | keybcs2_general_ci         | keybcs2  |  37 | Yes     | Yes      |       1 | PAD SPACE     |
        | koi8r_bin                  | koi8r    |  74 |         | Yes      |       1 | PAD SPACE     |
        | koi8r_general_ci           | koi8r    |   7 | Yes     | Yes      |       1 | PAD SPACE     |
        | koi8u_bin                  | koi8u    |  75 |         | Yes      |       1 | PAD SPACE     |
        | koi8u_general_ci           | koi8u    |  22 | Yes     | Yes      |       1 | PAD SPACE     |
        | latin1_bin                 | latin1   |  47 |         | Yes      |       1 | PAD SPACE     |
        | latin1_danish_ci           | latin1   |  15 |         | Yes      |       1 | PAD SPACE     |
        | latin1_general_ci          | latin1   |  48 |         | Yes      |       1 | PAD SPACE     |
        | latin1_general_cs          | latin1   |  49 |         | Yes      |       1 | PAD SPACE     |
        | latin1_german1_ci          | latin1   |   5 |         | Yes      |       1 | PAD SPACE     |
        | latin1_german2_ci          | latin1   |  31 |         | Yes      |       2 | PAD SPACE     |
        | latin1_spanish_ci          | latin1   |  94 |         | Yes      |       1 | PAD SPACE     |
        | latin1_swedish_ci          | latin1   |   8 | Yes     | Yes      |       1 | PAD SPACE     |
        | latin2_bin                 | latin2   |  77 |         | Yes      |       1 | PAD SPACE     |
        | latin2_croatian_ci         | latin2   |  27 |         | Yes      |       1 | PAD SPACE     |
        | latin2_czech_cs            | latin2   |   2 |         | Yes      |       4 | PAD SPACE     |
        | latin2_general_ci          | latin2   |   9 | Yes     | Yes      |       1 | PAD SPACE     |
        | latin2_hungarian_ci        | latin2   |  21 |         | Yes      |       1 | PAD SPACE     |
        | latin5_bin                 | latin5   |  78 |         | Yes      |       1 | PAD SPACE     |
        | latin5_turkish_ci          | latin5   |  30 | Yes     | Yes      |       1 | PAD SPACE     |
        | latin7_bin                 | latin7   |  79 |         | Yes      |       1 | PAD SPACE     |
        | latin7_estonian_cs         | latin7   |  20 |         | Yes      |       1 | PAD SPACE     |
        | latin7_general_ci          | latin7   |  41 | Yes     | Yes      |       1 | PAD SPACE     |
        | latin7_general_cs          | latin7   |  42 |         | Yes      |       1 | PAD SPACE     |
        | macce_bin                  | macce    |  43 |         | Yes      |       1 | PAD SPACE     |
        | macce_general_ci           | macce    |  38 | Yes     | Yes      |       1 | PAD SPACE     |
        | macroman_bin               | macroman |  53 |         | Yes      |       1 | PAD SPACE     |
        | macroman_general_ci        | macroman |  39 | Yes     | Yes      |       1 | PAD SPACE     |
        | sjis_bin                   | sjis     |  88 |         | Yes      |       1 | PAD SPACE     |
        | sjis_japanese_ci           | sjis     |  13 | Yes     | Yes      |       1 | PAD SPACE     |
        | swe7_bin                   | swe7     |  82 |         | Yes      |       1 | PAD SPACE     |
        | swe7_swedish_ci            | swe7     |  10 | Yes     | Yes      |       1 | PAD SPACE     |
        | tis620_bin                 | tis620   |  89 |         | Yes      |       1 | PAD SPACE     |
        | tis620_thai_ci             | tis620   |  18 | Yes     | Yes      |       4 | PAD SPACE     |
        | ucs2_bin                   | ucs2     |  90 |         | Yes      |       1 | PAD SPACE     |
        | ucs2_croatian_ci           | ucs2     | 149 |         | Yes      |       8 | PAD SPACE     |
        | ucs2_czech_ci              | ucs2     | 138 |         | Yes      |       8 | PAD SPACE     |
        | ucs2_danish_ci             | ucs2     | 139 |         | Yes      |       8 | PAD SPACE     |
        | ucs2_esperanto_ci          | ucs2     | 145 |         | Yes      |       8 | PAD SPACE     |
        | ucs2_estonian_ci           | ucs2     | 134 |         | Yes      |       8 | PAD SPACE     |
        | ucs2_general_ci            | ucs2     |  35 | Yes     | Yes      |       1 | PAD SPACE     |
        | ucs2_general_mysql500_ci   | ucs2     | 159 |         | Yes      |       1 | PAD SPACE     |
        | ucs2_german2_ci            | ucs2     | 148 |         | Yes      |       8 | PAD SPACE     |
        | ucs2_hungarian_ci          | ucs2     | 146 |         | Yes      |       8 | PAD SPACE     |
        | ucs2_icelandic_ci          | ucs2     | 129 |         | Yes      |       8 | PAD SPACE     |
        | ucs2_latvian_ci            | ucs2     | 130 |         | Yes      |       8 | PAD SPACE     |
        | ucs2_lithuanian_ci         | ucs2     | 140 |         | Yes      |       8 | PAD SPACE     |
        | ucs2_persian_ci            | ucs2     | 144 |         | Yes      |       8 | PAD SPACE     |
        | ucs2_polish_ci             | ucs2     | 133 |         | Yes      |       8 | PAD SPACE     |
        | ucs2_romanian_ci           | ucs2     | 131 |         | Yes      |       8 | PAD SPACE     |
        | ucs2_roman_ci              | ucs2     | 143 |         | Yes      |       8 | PAD SPACE     |
        | ucs2_sinhala_ci            | ucs2     | 147 |         | Yes      |       8 | PAD SPACE     |
        | ucs2_slovak_ci             | ucs2     | 141 |         | Yes      |       8 | PAD SPACE     |
        | ucs2_slovenian_ci          | ucs2     | 132 |         | Yes      |       8 | PAD SPACE     |
        | ucs2_spanish2_ci           | ucs2     | 142 |         | Yes      |       8 | PAD SPACE     |
        | ucs2_spanish_ci            | ucs2     | 135 |         | Yes      |       8 | PAD SPACE     |
        | ucs2_swedish_ci            | ucs2     | 136 |         | Yes      |       8 | PAD SPACE     |
        | ucs2_turkish_ci            | ucs2     | 137 |         | Yes      |       8 | PAD SPACE     |
        | ucs2_unicode_520_ci        | ucs2     | 150 |         | Yes      |       8 | PAD SPACE     |
        | ucs2_unicode_ci            | ucs2     | 128 |         | Yes      |       8 | PAD SPACE     |
        | ucs2_vietnamese_ci         | ucs2     | 151 |         | Yes      |       8 | PAD SPACE     |
        | ujis_bin                   | ujis     |  91 |         | Yes      |       1 | PAD SPACE     |
        | ujis_japanese_ci           | ujis     |  12 | Yes     | Yes      |       1 | PAD SPACE     |
        | utf16le_bin                | utf16le  |  62 |         | Yes      |       1 | PAD SPACE     |
        | utf16le_general_ci         | utf16le  |  56 | Yes     | Yes      |       1 | PAD SPACE     |
        | utf16_bin                  | utf16    |  55 |         | Yes      |       1 | PAD SPACE     |
        | utf16_croatian_ci          | utf16    | 122 |         | Yes      |       8 | PAD SPACE     |
        | utf16_czech_ci             | utf16    | 111 |         | Yes      |       8 | PAD SPACE     |
        | utf16_danish_ci            | utf16    | 112 |         | Yes      |       8 | PAD SPACE     |
        | utf16_esperanto_ci         | utf16    | 118 |         | Yes      |       8 | PAD SPACE     |
        | utf16_estonian_ci          | utf16    | 107 |         | Yes      |       8 | PAD SPACE     |
        | utf16_general_ci           | utf16    |  54 | Yes     | Yes      |       1 | PAD SPACE     |
        | utf16_german2_ci           | utf16    | 121 |         | Yes      |       8 | PAD SPACE     |
        | utf16_hungarian_ci         | utf16    | 119 |         | Yes      |       8 | PAD SPACE     |
        | utf16_icelandic_ci         | utf16    | 102 |         | Yes      |       8 | PAD SPACE     |
        | utf16_latvian_ci           | utf16    | 103 |         | Yes      |       8 | PAD SPACE     |
        | utf16_lithuanian_ci        | utf16    | 113 |         | Yes      |       8 | PAD SPACE     |
        | utf16_persian_ci           | utf16    | 117 |         | Yes      |       8 | PAD SPACE     |
        | utf16_polish_ci            | utf16    | 106 |         | Yes      |       8 | PAD SPACE     |
        | utf16_romanian_ci          | utf16    | 104 |         | Yes      |       8 | PAD SPACE     |
        | utf16_roman_ci             | utf16    | 116 |         | Yes      |       8 | PAD SPACE     |
        | utf16_sinhala_ci           | utf16    | 120 |         | Yes      |       8 | PAD SPACE     |
        | utf16_slovak_ci            | utf16    | 114 |         | Yes      |       8 | PAD SPACE     |
        | utf16_slovenian_ci         | utf16    | 105 |         | Yes      |       8 | PAD SPACE     |
        | utf16_spanish2_ci          | utf16    | 115 |         | Yes      |       8 | PAD SPACE     |
        | utf16_spanish_ci           | utf16    | 108 |         | Yes      |       8 | PAD SPACE     |
        | utf16_swedish_ci           | utf16    | 109 |         | Yes      |       8 | PAD SPACE     |
        | utf16_turkish_ci           | utf16    | 110 |         | Yes      |       8 | PAD SPACE     |
        | utf16_unicode_520_ci       | utf16    | 123 |         | Yes      |       8 | PAD SPACE     |
        | utf16_unicode_ci           | utf16    | 101 |         | Yes      |       8 | PAD SPACE     |
        | utf16_vietnamese_ci        | utf16    | 124 |         | Yes      |       8 | PAD SPACE     |
        | utf32_bin                  | utf32    |  61 |         | Yes      |       1 | PAD SPACE     |
        | utf32_croatian_ci          | utf32    | 181 |         | Yes      |       8 | PAD SPACE     |
        | utf32_czech_ci             | utf32    | 170 |         | Yes      |       8 | PAD SPACE     |
        | utf32_danish_ci            | utf32    | 171 |         | Yes      |       8 | PAD SPACE     |
        | utf32_esperanto_ci         | utf32    | 177 |         | Yes      |       8 | PAD SPACE     |
        | utf32_estonian_ci          | utf32    | 166 |         | Yes      |       8 | PAD SPACE     |
        | utf32_general_ci           | utf32    |  60 | Yes     | Yes      |       1 | PAD SPACE     |
        | utf32_german2_ci           | utf32    | 180 |         | Yes      |       8 | PAD SPACE     |
        | utf32_hungarian_ci         | utf32    | 178 |         | Yes      |       8 | PAD SPACE     |
        | utf32_icelandic_ci         | utf32    | 161 |         | Yes      |       8 | PAD SPACE     |
        | utf32_latvian_ci           | utf32    | 162 |         | Yes      |       8 | PAD SPACE     |
        | utf32_lithuanian_ci        | utf32    | 172 |         | Yes      |       8 | PAD SPACE     |
        | utf32_persian_ci           | utf32    | 176 |         | Yes      |       8 | PAD SPACE     |
        | utf32_polish_ci            | utf32    | 165 |         | Yes      |       8 | PAD SPACE     |
        | utf32_romanian_ci          | utf32    | 163 |         | Yes      |       8 | PAD SPACE     |
        | utf32_roman_ci             | utf32    | 175 |         | Yes      |       8 | PAD SPACE     |
        | utf32_sinhala_ci           | utf32    | 179 |         | Yes      |       8 | PAD SPACE     |
        | utf32_slovak_ci            | utf32    | 173 |         | Yes      |       8 | PAD SPACE     |
        | utf32_slovenian_ci         | utf32    | 164 |         | Yes      |       8 | PAD SPACE     |
        | utf32_spanish2_ci          | utf32    | 174 |         | Yes      |       8 | PAD SPACE     |
        | utf32_spanish_ci           | utf32    | 167 |         | Yes      |       8 | PAD SPACE     |
        | utf32_swedish_ci           | utf32    | 168 |         | Yes      |       8 | PAD SPACE     |
        | utf32_turkish_ci           | utf32    | 169 |         | Yes      |       8 | PAD SPACE     |
        | utf32_unicode_520_ci       | utf32    | 182 |         | Yes      |       8 | PAD SPACE     |
        | utf32_unicode_ci           | utf32    | 160 |         | Yes      |       8 | PAD SPACE     |
        | utf32_vietnamese_ci        | utf32    | 183 |         | Yes      |       8 | PAD SPACE     |
        | utf8mb4_0900_ai_ci         | utf8mb4  | 255 | Yes     | Yes      |       0 | NO PAD        |
        | utf8mb4_0900_as_ci         | utf8mb4  | 305 |         | Yes      |       0 | NO PAD        |
        | utf8mb4_0900_as_cs         | utf8mb4  | 278 |         | Yes      |       0 | NO PAD        |
        | utf8mb4_0900_bin           | utf8mb4  | 309 |         | Yes      |       1 | NO PAD        |
        | utf8mb4_bin                | utf8mb4  |  46 |         | Yes      |       1 | PAD SPACE     |
        | utf8mb4_croatian_ci        | utf8mb4  | 245 |         | Yes      |       8 | PAD SPACE     |
        | utf8mb4_cs_0900_ai_ci      | utf8mb4  | 266 |         | Yes      |       0 | NO PAD        |
        | utf8mb4_cs_0900_as_cs      | utf8mb4  | 289 |         | Yes      |       0 | NO PAD        |
        | utf8mb4_czech_ci           | utf8mb4  | 234 |         | Yes      |       8 | PAD SPACE     |
        | utf8mb4_danish_ci          | utf8mb4  | 235 |         | Yes      |       8 | PAD SPACE     |
        | utf8mb4_da_0900_ai_ci      | utf8mb4  | 267 |         | Yes      |       0 | NO PAD        |
        | utf8mb4_da_0900_as_cs      | utf8mb4  | 290 |         | Yes      |       0 | NO PAD        |
        | utf8mb4_de_pb_0900_ai_ci   | utf8mb4  | 256 |         | Yes      |       0 | NO PAD        |
        | utf8mb4_de_pb_0900_as_cs   | utf8mb4  | 279 |         | Yes      |       0 | NO PAD        |
        | utf8mb4_eo_0900_ai_ci      | utf8mb4  | 273 |         | Yes      |       0 | NO PAD        |
        | utf8mb4_eo_0900_as_cs      | utf8mb4  | 296 |         | Yes      |       0 | NO PAD        |
        | utf8mb4_esperanto_ci       | utf8mb4  | 241 |         | Yes      |       8 | PAD SPACE     |
        | utf8mb4_estonian_ci        | utf8mb4  | 230 |         | Yes      |       8 | PAD SPACE     |
        | utf8mb4_es_0900_ai_ci      | utf8mb4  | 263 |         | Yes      |       0 | NO PAD        |
        | utf8mb4_es_0900_as_cs      | utf8mb4  | 286 |         | Yes      |       0 | NO PAD        |
        | utf8mb4_es_trad_0900_ai_ci | utf8mb4  | 270 |         | Yes      |       0 | NO PAD        |
        | utf8mb4_es_trad_0900_as_cs | utf8mb4  | 293 |         | Yes      |       0 | NO PAD        |
        | utf8mb4_et_0900_ai_ci      | utf8mb4  | 262 |         | Yes      |       0 | NO PAD        |
        | utf8mb4_et_0900_as_cs      | utf8mb4  | 285 |         | Yes      |       0 | NO PAD        |
        | utf8mb4_general_ci         | utf8mb4  |  45 |         | Yes      |       1 | PAD SPACE     |
        | utf8mb4_german2_ci         | utf8mb4  | 244 |         | Yes      |       8 | PAD SPACE     |
        | utf8mb4_hr_0900_ai_ci      | utf8mb4  | 275 |         | Yes      |       0 | NO PAD        |
        | utf8mb4_hr_0900_as_cs      | utf8mb4  | 298 |         | Yes      |       0 | NO PAD        |
        | utf8mb4_hungarian_ci       | utf8mb4  | 242 |         | Yes      |       8 | PAD SPACE     |
        | utf8mb4_hu_0900_ai_ci      | utf8mb4  | 274 |         | Yes      |       0 | NO PAD        |
        | utf8mb4_hu_0900_as_cs      | utf8mb4  | 297 |         | Yes      |       0 | NO PAD        |
        | utf8mb4_icelandic_ci       | utf8mb4  | 225 |         | Yes      |       8 | PAD SPACE     |
        | utf8mb4_is_0900_ai_ci      | utf8mb4  | 257 |         | Yes      |       0 | NO PAD        |
        | utf8mb4_is_0900_as_cs      | utf8mb4  | 280 |         | Yes      |       0 | NO PAD        |
        | utf8mb4_ja_0900_as_cs      | utf8mb4  | 303 |         | Yes      |       0 | NO PAD        |
        | utf8mb4_ja_0900_as_cs_ks   | utf8mb4  | 304 |         | Yes      |      24 | NO PAD        |
        | utf8mb4_latvian_ci         | utf8mb4  | 226 |         | Yes      |       8 | PAD SPACE     |
        | utf8mb4_la_0900_ai_ci      | utf8mb4  | 271 |         | Yes      |       0 | NO PAD        |
        | utf8mb4_la_0900_as_cs      | utf8mb4  | 294 |         | Yes      |       0 | NO PAD        |
        | utf8mb4_lithuanian_ci      | utf8mb4  | 236 |         | Yes      |       8 | PAD SPACE     |
        | utf8mb4_lt_0900_ai_ci      | utf8mb4  | 268 |         | Yes      |       0 | NO PAD        |
        | utf8mb4_lt_0900_as_cs      | utf8mb4  | 291 |         | Yes      |       0 | NO PAD        |
        | utf8mb4_lv_0900_ai_ci      | utf8mb4  | 258 |         | Yes      |       0 | NO PAD        |
        | utf8mb4_lv_0900_as_cs      | utf8mb4  | 281 |         | Yes      |       0 | NO PAD        |
        | utf8mb4_persian_ci         | utf8mb4  | 240 |         | Yes      |       8 | PAD SPACE     |
        | utf8mb4_pl_0900_ai_ci      | utf8mb4  | 261 |         | Yes      |       0 | NO PAD        |
        | utf8mb4_pl_0900_as_cs      | utf8mb4  | 284 |         | Yes      |       0 | NO PAD        |
        | utf8mb4_polish_ci          | utf8mb4  | 229 |         | Yes      |       8 | PAD SPACE     |
        | utf8mb4_romanian_ci        | utf8mb4  | 227 |         | Yes      |       8 | PAD SPACE     |
        | utf8mb4_roman_ci           | utf8mb4  | 239 |         | Yes      |       8 | PAD SPACE     |
        | utf8mb4_ro_0900_ai_ci      | utf8mb4  | 259 |         | Yes      |       0 | NO PAD        |
        | utf8mb4_ro_0900_as_cs      | utf8mb4  | 282 |         | Yes      |       0 | NO PAD        |
        | utf8mb4_ru_0900_ai_ci      | utf8mb4  | 306 |         | Yes      |       0 | NO PAD        |
        | utf8mb4_ru_0900_as_cs      | utf8mb4  | 307 |         | Yes      |       0 | NO PAD        |
        | utf8mb4_sinhala_ci         | utf8mb4  | 243 |         | Yes      |       8 | PAD SPACE     |
        | utf8mb4_sk_0900_ai_ci      | utf8mb4  | 269 |         | Yes      |       0 | NO PAD        |
        | utf8mb4_sk_0900_as_cs      | utf8mb4  | 292 |         | Yes      |       0 | NO PAD        |
        | utf8mb4_slovak_ci          | utf8mb4  | 237 |         | Yes      |       8 | PAD SPACE     |
        | utf8mb4_slovenian_ci       | utf8mb4  | 228 |         | Yes      |       8 | PAD SPACE     |
        | utf8mb4_sl_0900_ai_ci      | utf8mb4  | 260 |         | Yes      |       0 | NO PAD        |
        | utf8mb4_sl_0900_as_cs      | utf8mb4  | 283 |         | Yes      |       0 | NO PAD        |
        | utf8mb4_spanish2_ci        | utf8mb4  | 238 |         | Yes      |       8 | PAD SPACE     |
        | utf8mb4_spanish_ci         | utf8mb4  | 231 |         | Yes      |       8 | PAD SPACE     |
        | utf8mb4_sv_0900_ai_ci      | utf8mb4  | 264 |         | Yes      |       0 | NO PAD        |
        | utf8mb4_sv_0900_as_cs      | utf8mb4  | 287 |         | Yes      |       0 | NO PAD        |
        | utf8mb4_swedish_ci         | utf8mb4  | 232 |         | Yes      |       8 | PAD SPACE     |
        | utf8mb4_tr_0900_ai_ci      | utf8mb4  | 265 |         | Yes      |       0 | NO PAD        |
        | utf8mb4_tr_0900_as_cs      | utf8mb4  | 288 |         | Yes      |       0 | NO PAD        |
        | utf8mb4_turkish_ci         | utf8mb4  | 233 |         | Yes      |       8 | PAD SPACE     |
        | utf8mb4_unicode_520_ci     | utf8mb4  | 246 |         | Yes      |       8 | PAD SPACE     |
        | utf8mb4_unicode_ci         | utf8mb4  | 224 |         | Yes      |       8 | PAD SPACE     |
        | utf8mb4_vietnamese_ci      | utf8mb4  | 247 |         | Yes      |       8 | PAD SPACE     |
        | utf8mb4_vi_0900_ai_ci      | utf8mb4  | 277 |         | Yes      |       0 | NO PAD        |
        | utf8mb4_vi_0900_as_cs      | utf8mb4  | 300 |         | Yes      |       0 | NO PAD        |
        | utf8mb4_zh_0900_as_cs      | utf8mb4  | 308 |         | Yes      |       0 | NO PAD        |
        | utf8_bin                   | utf8     |  83 |         | Yes      |       1 | PAD SPACE     |
        | utf8_croatian_ci           | utf8     | 213 |         | Yes      |       8 | PAD SPACE     |
        | utf8_czech_ci              | utf8     | 202 |         | Yes      |       8 | PAD SPACE     |
        | utf8_danish_ci             | utf8     | 203 |         | Yes      |       8 | PAD SPACE     |
        | utf8_esperanto_ci          | utf8     | 209 |         | Yes      |       8 | PAD SPACE     |
        | utf8_estonian_ci           | utf8     | 198 |         | Yes      |       8 | PAD SPACE     |
        | utf8_general_ci            | utf8     |  33 | Yes     | Yes      |       1 | PAD SPACE     |
        | utf8_general_mysql500_ci   | utf8     | 223 |         | Yes      |       1 | PAD SPACE     |
        | utf8_german2_ci            | utf8     | 212 |         | Yes      |       8 | PAD SPACE     |
        | utf8_hungarian_ci          | utf8     | 210 |         | Yes      |       8 | PAD SPACE     |
        | utf8_icelandic_ci          | utf8     | 193 |         | Yes      |       8 | PAD SPACE     |
        | utf8_latvian_ci            | utf8     | 194 |         | Yes      |       8 | PAD SPACE     |
        | utf8_lithuanian_ci         | utf8     | 204 |         | Yes      |       8 | PAD SPACE     |
        | utf8_persian_ci            | utf8     | 208 |         | Yes      |       8 | PAD SPACE     |
        | utf8_polish_ci             | utf8     | 197 |         | Yes      |       8 | PAD SPACE     |
        | utf8_romanian_ci           | utf8     | 195 |         | Yes      |       8 | PAD SPACE     |
        | utf8_roman_ci              | utf8     | 207 |         | Yes      |       8 | PAD SPACE     |
        | utf8_sinhala_ci            | utf8     | 211 |         | Yes      |       8 | PAD SPACE     |
        | utf8_slovak_ci             | utf8     | 205 |         | Yes      |       8 | PAD SPACE     |
        | utf8_slovenian_ci          | utf8     | 196 |         | Yes      |       8 | PAD SPACE     |
        | utf8_spanish2_ci           | utf8     | 206 |         | Yes      |       8 | PAD SPACE     |
        | utf8_spanish_ci            | utf8     | 199 |         | Yes      |       8 | PAD SPACE     |
        | utf8_swedish_ci            | utf8     | 200 |         | Yes      |       8 | PAD SPACE     |
        | utf8_tolower_ci            | utf8     |  76 |         | Yes      |       1 | PAD SPACE     |
        | utf8_turkish_ci            | utf8     | 201 |         | Yes      |       8 | PAD SPACE     |
        | utf8_unicode_520_ci        | utf8     | 214 |         | Yes      |       8 | PAD SPACE     |
        | utf8_unicode_ci            | utf8     | 192 |         | Yes      |       8 | PAD SPACE     |
        | utf8_vietnamese_ci         | utf8     | 215 |         | Yes      |       8 | PAD SPACE     |
        +----------------------------+----------+-----+---------+----------+---------+---------------+
        ```
        
    - 콜레이션의 이름은 2개 또는 3개 파트로 구분돼 있다.
        - 3개의 파트로 구성된 콜레이선 이름
            - 첫 번째 파트는 문자집합의 이름이다.
            - 두 번째 파트는 해당문지집합의 하위 분류를 나타낸다.
            - 세 번째 파트는 대문자나 소문자의 구분 여부를 나타낸다.
                - 즉, 세 번째 파트가 `ci`이면 대소문자를 구분하지 않는 콜레이션(Case Insensitive}을 의미
                - `cs`이면 대소문자를 별도의 문자로 구분하는 콜레이션(Case Sensitive)이다.
        - 2개의 파트로 구성된 콜레이션 이름
            - 첫 번째 파트는 마찬가지로 문자 집합의 이름이다.
            - 두 번째 파트는 항상 `bin` 키워드가 사용된다.
                - 이진 데이터(binary)를 의미
                - 이진 데이터로 관리되는 문자열 칼럼은 별도의 콜레이션을 가지지 않는다.
                - 콜레이션이 `xx_bin`이라면 비교 및 정렬은 실제 문자 데이터의 바이트 값을 기준으로 수행된다.
    - 조인을 수행하는 양쪽 테이블의 칼럼이 문자 집합이나 콜레이션이 다르다면 비교 작업에서 콜레이션의 변환이 필요하기 때문에 인덱스를 효율적으로 이용하지 못할 때가 많으므로 주의해야 한다.
    - 테이블을 생성할 때 문자 집합이나 콜레이션을 적용하는 방법
        
        ```sql
        # utf8의 기본 콜레이션인 utf8_general_ci가 기본 콜레이션이 된다.
        CREATE DATABASE db test CHARACTER SET=utf8;
        
        CREATE TABLE tb_member(
          member_id VARCHAR(20) NOT NULL COLLATE latin1_general_cs,
          member_name VARCHAR(20) NOT NULL COLLATE utf8_bin,
          member_email VARCHAR(100) NOT NULL,
        ...
        );
        ```
        
        - tb_member 테이블을 생성하면서 member_id 칼럼의 콜레이선을 `latin1_general_cs`로 설정했다.
            - 숫자나 영문 알파벳, 그리고 키보드의 특수 문자 위주로만 저장할 수 있고 `_cs` 계열의 콜레이션이므로 대소문자 구분을 하는 정렬이나 비교를 수행한다.
        - member_name 칼럼은 콜레이션이 `utf8_bin`으로 설정됐으므로 한글이나 다른 나라의 언어를 사용할 수 있지만 `_bin` 계열의 콜레이션이 사용됐으므로 대소문자를 구분하는 정렬과 비교를 수행한다.
        - member_email 칼럼은 아무런 문자집합이나 콜레이선을 정의하지 않았으므로 DB 의 기본 문자 집합과 콜레이션을 그대로 사용한다.
            - `utf8_general_ci` 콜레이션을 사용하고 비교나 정렬 시 대소문자를 구분하지 않는다.
    - `SHOW CREATE TABLE` 명령: 테이블의 구조를 확인할 수 있다.
        
        ```sql
        SHOW CREATE TABLE tb_test;
        +---------+----------------------------------------------------------------------------------------------------------------------------------------------+
        | Table   | Create Table                                                                                                                                 |
        +---------+----------------------------------------------------------------------------------------------------------------------------------------------+
        | tb_test | CREATE TABLE `tb_test` (
          `age` int DEFAULT NULL,
          KEY `ix_age` (`age`)
        ) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci |
        +---------+----------------------------------------------------------------------------------------------------------------------------------------------+
        ```
        
    - `information_schema`.`columns`: 각 칼럼의 문자 집합이나 콜레이션을 정확히 확인할 수 있다.
        
        ```sql
        SELECT table_name, column_name, column_type, character_set_name, collation_name
          FROM information_schema.columns
         WHERE table_schema='test'
           AND table_name='tb_test';
        +------------+-------------+-------------+--------------------+----------------+
        | TABLE_NAME | COLUMN_NAME | COLUMN_TYPE | CHARACTER_SET_NAME | COLLATION_NAME |
        +------------+-------------+-------------+--------------------+----------------+
        | tb_test    | age         | int         | NULL               | NULL           |
        +------------+-------------+-------------+--------------------+----------------+
        ```
        
- 2️⃣ `utf8mb4` 문자 집합의 콜레이션
    - 최근에는 응용 프로그램의 다국어 지원이 필수적이어서 대부분은 `utf8mb4` 문자 집합을 사용할 것이다.
    - 콜레이션 이름의 숫자 값은 모두 콜레이션의 비교 알고리즘 버전이다.
    - 콜레이션의 이름에 로캘(Locale)이 포함돼 있는지 여부로 언어에 종속적인 콜레이션과 언어에 비종속적인 콜레이션으로 구분할 수 있다 👉 범용 응용 프로그램일 경우 `utf8mb4_0900_ai_ci` 콜레이션으로도 충분할 것이다.
        
        
        | 콜레이션 | 언어 | 표기 |
        | --- | --- | --- |
        | utf8mb4_0900_ai_ci | N/A | 없음 |
        | utf8mb4_zh_0900_as_cs | 중국어 | zh |
        | utf8mb4_la_0900_ai_ci | 클래식 라틴 | la 또는 roman |
        | utf8mb4_de_pb_0900_ai_ci | 독일 전화번호 안내 책자 순서 | de_pb 또는 german2 |
        | utf8mb4_ja_0900_as_cs | 일본어 | ja |
        | utf8mb4_ro_0900_ai_ci | 로마어 | ro 또는 romanian |
        | utf8mb4_ru_0900_ai_ci | 러시아어 | ru |
        | utf8mb4_es_0900_ai_ci | 현대 스페인어 | es 또는 spanish |
        | utf8mb4_vi_0900_ai_ci | 베트남 | vi 또는 vietnamese |
        - 언어 비종속적인 콜레이션은 문자 셋의 기본 정렬 순서에 의해 정렬 및 비교가 수행되며,
        - 언어 종속적인 콜레이션은 해당 언어에서 정의한 정렬 순서에 의해 정렬 및 비교가 수행된다.
    - MySQL 8.0 으로 업그레이드하는 경우 `utf8mb4` 문자 집합의 콜레이션에 주의해야 한다.
        - MySQL 5.7 버전에서는 `utf8mb4` 문자 집합의 기본 콜레이션은 `utf8mb4_general_ci` 였는데, MySQL 8.0 버전부터는 `utf8mb4_0900_ai_ci`로 변경됐다.
        - MySQL 5.7 버전부터 존재하던 테이블은 이미 `utf8mb4_general_ci` 콜레이션을 사용하고 있기 때문에 조인할 때 에러가 발생하거나 성능이 심각하게 떨어진다.
        - `default_collation_for_utf8mb4` 시스템 변수에 `utf8mb4_generai_ci`로 설정하면 되지만, 이 시스템 변수는 일시적으로 제공되는 기능이므로 영구적으로 사용하기에는 불안할 수 있다.
        
        ❗MySQL 8.0 으로 새로 시작하는 응용 프로그램이라면 `utf8mb4_0900_ai_ci` 콜레이션을 기본으로 사용할 것을 권장한다.
        

### 1-5. 비교 방식

---

- MySQL 에서 문자열 칼럼을 비교하는 방식은 `CHAR`와 `VARCHAR`가 거의 같다.
    - MySQL 에서 지원하는 대부분의 문자 집합과 콜레이션에서 `CHAR` 타입이나 `VARCHAR` 타입을 비교할 때 공백 문자를 뒤에 붙여서 두 문자열의 길이를 동일하게 만든 후 비교를 수행한다.
- 하지만 `utf8mb4` 문자 집합이 UCA 버전 9.0.0 을 지원하면서 문자열 뒤에 붙어있는 공백 문자들에 대한 비교 방식이 달라졌다.
    - `utf8mb4_bin` 콜레이션을 사용하는 경우 문자열 뒤에 붙어있는 공백은 비교에 영향을 미치지 않는다.
    - `utf8mb4_0900_bin` 콜레이션을 사용하는 경우 문자열 뒤의 공백이 비교 결과에 영향을 미친다.
- 문자열 뒤의 공백이 비교 결과에 영향을 미치는지 아닌지는 `information_schema`.`COLLATIONS` 뷰에서 `PAD_ATTRIBUTE` 칼럼의 값으로 판단할 수 있다.
    
    ```sql
    SELECT * FROM information_schema.COLLATIONS WHERE collation_name LIKE 'utf8mb4%';
    +----------------------------+--------------------+-----+------------+-------------+---------+---------------+
    | COLLATION_NAME             | CHARACTER_SET_NAME | ID  | IS_DEFAULT | IS_COMPILED | SORTLEN | **PAD_ATTRIBUTE** |
    +----------------------------+--------------------+-----+------------+-------------+---------+---------------+
    | utf8mb4_0900_ai_ci         | utf8mb4            | 255 | Yes        | Yes         |       0 | NO PAD        |
    | utf8mb4_0900_as_ci         | utf8mb4            | 305 |            | Yes         |       0 | NO PAD        |
    | utf8mb4_0900_as_cs         | utf8mb4            | 278 |            | Yes         |       0 | NO PAD        |
    | utf8mb4_0900_bin           | utf8mb4            | 309 |            | Yes         |       1 | NO PAD        |
    | utf8mb4_bin                | utf8mb4            |  46 |            | Yes         |       1 | PAD SPACE     |
    | utf8mb4_croatian_ci        | utf8mb4            | 245 |            | Yes         |       8 | PAD SPACE     |
    | utf8mb4_cs_0900_ai_ci      | utf8mb4            | 266 |            | Yes         |       0 | NO PAD        |
    | utf8mb4_cs_0900_as_cs      | utf8mb4            | 289 |            | Yes         |       0 | NO PAD        |
    | utf8mb4_czech_ci           | utf8mb4            | 234 |            | Yes         |       8 | PAD SPACE     |
    | utf8mb4_danish_ci          | utf8mb4            | 235 |            | Yes         |       8 | PAD SPACE     |
    | utf8mb4_da_0900_ai_ci      | utf8mb4            | 267 |            | Yes         |       0 | NO PAD        |
    | utf8mb4_da_0900_as_cs      | utf8mb4            | 290 |            | Yes         |       0 | NO PAD        |
    | utf8mb4_de_pb_0900_ai_ci   | utf8mb4            | 256 |            | Yes         |       0 | NO PAD        |
    | utf8mb4_de_pb_0900_as_cs   | utf8mb4            | 279 |            | Yes         |       0 | NO PAD        |
    | utf8mb4_eo_0900_ai_ci      | utf8mb4            | 273 |            | Yes         |       0 | NO PAD        |
    | utf8mb4_eo_0900_as_cs      | utf8mb4            | 296 |            | Yes         |       0 | NO PAD        |
    | utf8mb4_esperanto_ci       | utf8mb4            | 241 |            | Yes         |       8 | PAD SPACE     |
    | utf8mb4_estonian_ci        | utf8mb4            | 230 |            | Yes         |       8 | PAD SPACE     |
    | utf8mb4_es_0900_ai_ci      | utf8mb4            | 263 |            | Yes         |       0 | NO PAD        |
    | utf8mb4_es_0900_as_cs      | utf8mb4            | 286 |            | Yes         |       0 | NO PAD        |
    | utf8mb4_es_trad_0900_ai_ci | utf8mb4            | 270 |            | Yes         |       0 | NO PAD        |
    | utf8mb4_es_trad_0900_as_cs | utf8mb4            | 293 |            | Yes         |       0 | NO PAD        |
    | utf8mb4_et_0900_ai_ci      | utf8mb4            | 262 |            | Yes         |       0 | NO PAD        |
    | utf8mb4_et_0900_as_cs      | utf8mb4            | 285 |            | Yes         |       0 | NO PAD        |
    | utf8mb4_general_ci         | utf8mb4            |  45 |            | Yes         |       1 | PAD SPACE     |
    | utf8mb4_german2_ci         | utf8mb4            | 244 |            | Yes         |       8 | PAD SPACE     |
    | utf8mb4_hr_0900_ai_ci      | utf8mb4            | 275 |            | Yes         |       0 | NO PAD        |
    | utf8mb4_hr_0900_as_cs      | utf8mb4            | 298 |            | Yes         |       0 | NO PAD        |
    | utf8mb4_hungarian_ci       | utf8mb4            | 242 |            | Yes         |       8 | PAD SPACE     |
    | utf8mb4_hu_0900_ai_ci      | utf8mb4            | 274 |            | Yes         |       0 | NO PAD        |
    | utf8mb4_hu_0900_as_cs      | utf8mb4            | 297 |            | Yes         |       0 | NO PAD        |
    | utf8mb4_icelandic_ci       | utf8mb4            | 225 |            | Yes         |       8 | PAD SPACE     |
    | utf8mb4_is_0900_ai_ci      | utf8mb4            | 257 |            | Yes         |       0 | NO PAD        |
    | utf8mb4_is_0900_as_cs      | utf8mb4            | 280 |            | Yes         |       0 | NO PAD        |
    | utf8mb4_ja_0900_as_cs      | utf8mb4            | 303 |            | Yes         |       0 | NO PAD        |
    | utf8mb4_ja_0900_as_cs_ks   | utf8mb4            | 304 |            | Yes         |      24 | NO PAD        |
    | utf8mb4_latvian_ci         | utf8mb4            | 226 |            | Yes         |       8 | PAD SPACE     |
    | utf8mb4_la_0900_ai_ci      | utf8mb4            | 271 |            | Yes         |       0 | NO PAD        |
    | utf8mb4_la_0900_as_cs      | utf8mb4            | 294 |            | Yes         |       0 | NO PAD        |
    | utf8mb4_lithuanian_ci      | utf8mb4            | 236 |            | Yes         |       8 | PAD SPACE     |
    | utf8mb4_lt_0900_ai_ci      | utf8mb4            | 268 |            | Yes         |       0 | NO PAD        |
    | utf8mb4_lt_0900_as_cs      | utf8mb4            | 291 |            | Yes         |       0 | NO PAD        |
    | utf8mb4_lv_0900_ai_ci      | utf8mb4            | 258 |            | Yes         |       0 | NO PAD        |
    | utf8mb4_lv_0900_as_cs      | utf8mb4            | 281 |            | Yes         |       0 | NO PAD        |
    | utf8mb4_persian_ci         | utf8mb4            | 240 |            | Yes         |       8 | PAD SPACE     |
    | utf8mb4_pl_0900_ai_ci      | utf8mb4            | 261 |            | Yes         |       0 | NO PAD        |
    | utf8mb4_pl_0900_as_cs      | utf8mb4            | 284 |            | Yes         |       0 | NO PAD        |
    | utf8mb4_polish_ci          | utf8mb4            | 229 |            | Yes         |       8 | PAD SPACE     |
    | utf8mb4_romanian_ci        | utf8mb4            | 227 |            | Yes         |       8 | PAD SPACE     |
    | utf8mb4_roman_ci           | utf8mb4            | 239 |            | Yes         |       8 | PAD SPACE     |
    | utf8mb4_ro_0900_ai_ci      | utf8mb4            | 259 |            | Yes         |       0 | NO PAD        |
    | utf8mb4_ro_0900_as_cs      | utf8mb4            | 282 |            | Yes         |       0 | NO PAD        |
    | utf8mb4_ru_0900_ai_ci      | utf8mb4            | 306 |            | Yes         |       0 | NO PAD        |
    | utf8mb4_ru_0900_as_cs      | utf8mb4            | 307 |            | Yes         |       0 | NO PAD        |
    | utf8mb4_sinhala_ci         | utf8mb4            | 243 |            | Yes         |       8 | PAD SPACE     |
    | utf8mb4_sk_0900_ai_ci      | utf8mb4            | 269 |            | Yes         |       0 | NO PAD        |
    | utf8mb4_sk_0900_as_cs      | utf8mb4            | 292 |            | Yes         |       0 | NO PAD        |
    | utf8mb4_slovak_ci          | utf8mb4            | 237 |            | Yes         |       8 | PAD SPACE     |
    | utf8mb4_slovenian_ci       | utf8mb4            | 228 |            | Yes         |       8 | PAD SPACE     |
    | utf8mb4_sl_0900_ai_ci      | utf8mb4            | 260 |            | Yes         |       0 | NO PAD        |
    | utf8mb4_sl_0900_as_cs      | utf8mb4            | 283 |            | Yes         |       0 | NO PAD        |
    | utf8mb4_spanish2_ci        | utf8mb4            | 238 |            | Yes         |       8 | PAD SPACE     |
    | utf8mb4_spanish_ci         | utf8mb4            | 231 |            | Yes         |       8 | PAD SPACE     |
    | utf8mb4_sv_0900_ai_ci      | utf8mb4            | 264 |            | Yes         |       0 | NO PAD        |
    | utf8mb4_sv_0900_as_cs      | utf8mb4            | 287 |            | Yes         |       0 | NO PAD        |
    | utf8mb4_swedish_ci         | utf8mb4            | 232 |            | Yes         |       8 | PAD SPACE     |
    | utf8mb4_tr_0900_ai_ci      | utf8mb4            | 265 |            | Yes         |       0 | NO PAD        |
    | utf8mb4_tr_0900_as_cs      | utf8mb4            | 288 |            | Yes         |       0 | NO PAD        |
    | utf8mb4_turkish_ci         | utf8mb4            | 233 |            | Yes         |       8 | PAD SPACE     |
    | utf8mb4_unicode_520_ci     | utf8mb4            | 246 |            | Yes         |       8 | PAD SPACE     |
    | utf8mb4_unicode_ci         | utf8mb4            | 224 |            | Yes         |       8 | PAD SPACE     |
    | utf8mb4_vietnamese_ci      | utf8mb4            | 247 |            | Yes         |       8 | PAD SPACE     |
    | utf8mb4_vi_0900_ai_ci      | utf8mb4            | 277 |            | Yes         |       0 | NO PAD        |
    | utf8mb4_vi_0900_as_cs      | utf8mb4            | 300 |            | Yes         |       0 | NO PAD        |
    | utf8mb4_zh_0900_as_cs      | utf8mb4            | 308 |            | Yes         |       0 | NO PAD        |
    +----------------------------+--------------------+-----+------------+-------------+---------+---------------+
    ```
    
    - `PAD SPACE` 라고 표시된 콜레이션에서는 비교 대ㅑ상 문자열의 길이가 같아지도록 문자열 뒤에 공백을 추가해서 비교를 수행한다.
    - `NO PAD`의 경우 별도로 문자열의 길이를 일치시키지 않고 그대로 비교한다.

👉 MySQL 서버에서 지원하는 대부분의 콜레이션은 `PAD SPACE`, `utf8mb4_0900`으로 시작하는 콜레이션만 `NO PAD`다.

- 문자열 비교의 경우 예외적으로 `LIKE`를 사용한 문자열 패턴 비교에서는 공백 문자가 유효 문자로 취급된다.
    
    ```sql
    SELECT 'ABC   ' LIKE 'ABC' AS is_same_pattern;
    +-----------------+
    | is_same_pattern |
    +-----------------+
    |               0 |
    +-----------------+
    ```
    

### 1-6. 문자열 이스케이프 처리

---

- MySQL 에서 SQL 문장에 사용하는 문자열은 프로그래밍 언어처럼 '`\`' 를 이용해 이스케이프 처리를 하는 것이 가능하다.
    
    
    | 이스케이프 표기 | 의미 |
    | --- | --- |
    | \0 | 아스키 NULL 문자 |
    | \' | 홑따옴표 |
    | \" | 쌍따옴표 |
    | \b | 백스페이스 문자 |
    | \n | 개행 문자 |
    | \r | 캐리지 리턴 문자 |
    | \t | 탭 문자 |
    | \\ | 백슬래시 문자 |
    | \% | 퍼센트 문자 |
    | \_ | 언더 스코어 문자 |
- MySQL 에서는 다른 DBMS 에서와 같이 홑따옴표와 쌍따옴표의 경우에는 두 번 연속으로 표시해서 이스케이프 처리할 수도 있다.

## 2. 숫자

---

- 숫자를 저장하는 타입은 값의 정확도에 따라 참값(Exact Value)과 근삿값 타입으로 나눌 수 있다.
    - 참값 : 소수점 이하 값의 유무와 관계없이 정확히 그 값을 유지하는 것. `INTEGER`를 포함해 `INT`로 끝나는 타입과 `DECIMAL`이 있다.
    - 근삿값 : 흔히 부동 소수점이라고 불리는 값. `FLOAT`, `DOUBLE` 이 있다.
- 값이 저장되는 포맷에 따라 십진 표기법(`DECIMAL`)과 이진 표기법으로 나눌 수 있다.
    - 이진 표기법: 흔히 프로그래밍 언어에서 사용하는 정수나 실수 타입을 의미한다.
        - 256 까지의 숫자를 표현할 수 있기 때문에 숫자 값을 적은 메모리나 디스크 공간에 저장할 수 있다.
        - MySQL 의 `INTEGER나 BIGINT` 등 대부분 숫자 타입은 모두 이진 표기법을 사용한다.
    - 십진 표기법(`DECIMAL`): 각 자릿값을 표현하기 위해 4비트나 한 바이트를 사용해 표기하는 방법
        - MySQL 의 십진 표기법을 사용하는 타입은 `DECIMAL` 뿐이며, 정확하게 소수점까지 관리돼야 하는 값을 저장할 때 사용한다.

- DBMS 에서는 근삿값은 저장할 때와 조회할 때의 값이 정확히 일치하지 않고, 유효 자릿수를 넘어서는 소수점 이하의 값은 계속 바뀔 수 있다.
    - 특히 `STATEMENT` 포맷을 사용하는 복제에서는 소스 서버와 레플리카 서버 간 데이터 차이가 발생할 수도 있다.
- MySQL 에서 `FLOAT`나 `DOUBLE` 같은 부동 소수점 타입은 잘 사용하지 않는다.
- 십진 표기법을 사용하는 `DECIMAL` 타입은 이진 표기법을 사용하는 타입보다 저장 공간을 2배 이상 필요로 한다.****
- 매우 큰 숫자 값이나 고정 소수점을 저장해야 하는 것이 아니라면 일반적으로 `INTEGER`나 `BIGINT` 타입을 자주 사용한다.

### 2-1. 정수

---

- `DECIMAL`타입을 제외하고 정수를 저장하는 데 사용할 수 있는 데이터 타입으로는 5가지가 있다.
    - 저장 가능한 숫자 값의 범위만 다를 뿐 다른 차이는 거의 없다.

| 데이터 타입 | 저장 공간 | 최솟값 (Signed) | 최솟값 (Unsigned) | 최댓값 (Signed) | 최댓값 (Unsigned) |
| --- | --- | --- | --- | --- | --- |
| TINYINT | 1 | -128 | 0 | 127 | 255 |
| SMALLINT | 2 | -32768 | 0 | 32767 | 65535 |
| MEDIUMINT | 3 | -8388608 | 0 | 8388607 | 16777215 |
| INT | 4 | -2147483648 | 0 | 2147483647 | 4294967295 |
| BIGINT | 8 | -263 | 0 | 263-1 | 264-1 |
- 정수타입은 `UNSIGNED` 옵션을 사용할 수 있다.
    - 정수 칼럼을 생성할 때 `UNSIGNED` 옵션을 명시하지 않으면 기본적으로 음수와 양수를 동시에 저장할 수 있는 숫자 타입(`SIGNED`)이 된다.
- 정수 타입에서 `UNSIGNED` 옵션은 조인할 때 인덱스의 사용 여부까지 영향을 미치지는 않는다.
- 외래키로 사용하는 칼럼이나 조인의 조건이 되는 칼럼은 `SIGNED` 나 `UNSIGNED` 옵션을 일치시키는 것이 좋다.

### 2-2. 부동 소수점

---

- MySQL 에서는 부동 소수점을 저장하기 위해 `FLOAT`과 `DOUBLE` 타입을 사용할 수 있다.
- 부동 소수점은 근삿값을 저장하는 방식이라서 동등 비교(Equal)는 사용할 수 없다.
    - `FLOAT`: 일반적으로 정밀도를 명시하지 않으면 4바이트를 사용해 유효 자릿수를 8개까지 유지하며, 정밀도가 명시된 경우 최대 8바이트 까지 저장공간을 사용할 수 있다.
        
        ```sql
        CREATE TABLE tb_float (fd1 FLOAT);
        INSERT INTO tb_float VALUES (0.1);
        
        SELECT * FROM tb_float WHERE fd1=0.1;
        Empty set (0.00 sec)
        ```
        
    - `DOUBLE`: 8바이트의 저장 공간을 필요로 하며 최대 유효 자릿수를 16개까지 유지할 수 있다.

- ❗만약 부동 소수점 값을 저장해야 한다면 유효 소수점의 자릿수만큼 10을 곱해서 정수로 만들어 그 값을 정수 타입의 칼럼에 저장하는 방법도 생각해볼 수 있다.
    
    ```sql
    CREATE TABLE tb_location (
      latitude INT UNSIGNED,
      longitude INT UNSIGNED
    ...
    );
    
    INSERT INTO tb_location (latitude, longitude, .. ) VALUES (37.1422 * 10000, 131 .5208 * 10000, ..);
    
    SELECT latitude/10000 AS latitude, longitude/10000 AS longitude
      FROM tb_location
     WHERE latitude=37.1422 * 10000
       AND longitude=131.5208 * 10000;
    ```
    

### 2-3. `DECIMAL`

---

- MySQL 에서 소수점 이하의 값까지 정확하게 관리하려면 `DECIMAL` 타입을 이용해야 한다.
    - 숫자 하나를 저장하는데 1/2 바이트가 필요하다.
    
    → `DECIMAL`로 저장하는(숫자의 자릿수)/2 의 결괏값을 올림 처리한 만큼의 바이트수가 필요하다.
    
- 미세한 차이지만 `DECIMAL` 보다는 `BIGINT` 타입이 값을 곱하는 연산시 더 빠르다.
- 단순히 정수를 관리하고자 한다면 `INTEGER`나 `BIGINT`를 사용하는 것이 좋다.

### 2-4. 정수 타입의 칼럼을 생성할 때의 주의사항

---

- `FLOAT`나 `DOUBLE` 타입은 저장 공간의 크기가 고정이므로 정밀도를 조절한다고 해서 저장 공간의 크기가 바뀌는 것은 아니다.
    - 하지만 `DECIMAL` 타입은 저장 공간의 크기가 가변적인 데이터 타입이어서 정밀도는 저장 가능한 자릿수를 결정함과 동시에 저장 공간의 크기까지 제한한다.
- MySQL 5.7 버전까지는 부동 소수점이나 고정 소수점이 아닌 정수 타입을 생성할 때도 똑같이 `BIGINT(10)`과 같이 크기를 명시할 수 있는 문법을 지원했다.
    - 정수 타입 뒤에 명시되는 괄호는 화면에 표시할 자릿수를 의미할 뿐 저장 가능한 값을 제한하는 용도가 아니다.
- MySQL 8.0 부터는 이렇게 정수 타입에 화면 표시 자릿수를 사용하는 기능은 제거됐다.
    - `(10)`은 무시된다.

### 2-5. 자동 증가(`AUTO_INCREMENT`) 옵션 사용

---

- 테이블의 PK 를 구성하는 칼럼의 크기가 너무 크거나 PK 로 사용할 만한 칼럼이 없을 때는 숫자 타입 칼럼에 자동 증가 옵션을 사용해 인조키를 생성할 수 있다.
- MySQL 서버의 `auto_increment_increment` 와 `auto_increment_offset` 시스템 설정을 이용해 `AUTO_INCREMENT` 칼럼의 자동 증가값이 얼마가 될지 변경할 수 있다.
    - 일반적으로 이 두 시스템 변숫값은 모두 1로 사용된다.
- `AUTO_INCREMENT` 옵션을 사용한 칼럼은 반드시 그 테이블에서 PK 나 유니크 키의 일부로 정의해야 한다.

- PK 나 유니크키가 여러 칼럼으로 구성되면 `AUTO_INCREMENT` 속성의 칼럼값이 증가하는 패턴이 스토리지 엔진마다 달라진다.
    - MyISAM: `AUTO_INCREMENT` 옵션이 사용된 칼럼이 PK 나 유니크 키의 아무 위치에나 사용될 수 있다.
    - InnoDB: `AUTO_INCREMENT` 칼럼으로 시작되는 인덱스를 생성해야 한다.
        - `AUTO_INCREMENT` 칼럼을 PK 의 뒤쪽에 배치하면 오류가 발생한다.
            
            ```sql
            CREATE TABLE tb_autoinc_innodb (
              fdpk1 INT,
              fdpk2 INT AUTO_INCREMENT,
              PRIMARY KEY(fdpk1, fdpk2)
            ) ENGINE=INNODB;
            ERROR 1075 (42000): Incorrect table definition; there can be only one auto column and it must be defined as a key
            ```
            

- `AUTO_INCREMENT` 칼럼은 테이블당 하나만 사용할 수 있다.
- `AUTO_INCREMENT` 칼럼의 현재 증가 값은 테이블의 메타 정보에 저장돼 있는데, 이는 `SHOW CREATE TABLE` 명령으로 조회할 수 있다.

## 3. 날짜와 시간

---

- MySQL 에서는 날짜만 저장하거나 시간만 따로 저장할 수도 있으며, 날짜와 시간을 합쳐서 하나의 칼럼에 저장할 수 있게 여러 가지 타입을 지원한다 → `DATE` 와 `DATETIME` 타입이 많이 사용된다.
- MySQL 5.6 버전부터 `TIME` 타입과 `DATETIME`, `TIMESTAMP` 타입은 밀리초 단위의 데이터를 저장할 수 있게 됐다.
    - 칼럼의 저장 공간 크기는 밀리초 단위를 몇 자리까지 저장하느냐에 따라 달라진다.
    
    | 데이터 타입 | MySQL 5.6.4 이전 | MySQL 5.6.4 부터 |
    | --- | --- | --- |
    | YEAR | 1 바이트 | 1 바이트 |
    | DATE | 3 바이트 | 3 바이트 |
    | TIME | 3 바이트 | 3 바이트 + (밀리초 단위 저장 공간) |
    | DATETIME | 8 바이트 | 5 바이트 + (밀리초 단위 저장 공간) |
    | TIMESTAMP | 4 바이트 | 4 바이트 + (밀리초 단위 저장 공간) |
- 밀리초 단위는 2자리당 1바이트씩 공간이 더 필요하다. 그래서 MySQL 8.0 에서는 마이크로초까지 저장 가능한 `DATETIME(6)`타입은 8바이트(5 + 3)를 사용한다.
    
    
    | 밀리초 단위 자릿수 | 저장 공간 |
    | --- | --- |
    | 없음 | 0 바이트 |
    | 1,2 | 1 바이트 |
    | 3,4 | 2 바이트 |
    | 5,6 | 3 바이트 |
- `NOW()` 함수를 이용해 현재 시간을 가져올 때도 가져올 밀리초의 자릿수를 명시해야 한다.
    - `NOW()`로 현재 시간을 가져오면 자동으로 `NOW(0)`으로 실행되어 밀리초 단위는 0으로 반환된다.
- MySQL 의 날짜 타입은 칼럼 자체에 타임존 정보가 저장되지 않으므로 `DATETIME`이나 `DATE` 타입은 현재 DBMS 커넥션의 타임존과 관계없이 클라이언트로부터 입력된 값을 그대로 저장하고 조회할 때도 변환없이 그대로 출력한다.
- 하지만 `TIMESTAMP`는 항상 UTC 타임존으로 저장되므로 타임존이 달라져도 값이 자동으로 보정된다.

- MySQL 서버의 칼럼이 `TIMESTAMP`이든 `DATETIME`이든 관계없이, JDBC 드라이버는 날짜 및 시간 정보를 MySQL 타임존에서 JVM 의 타임존으로 변환해서 출력한다.
- ORM 을 사용하는 경우에는 `DATETIME`타입의 칼럼값을 어떤 JDBC API 를 이용해서 페치하는지, 타임존 변환이 기대하는 대로 자동 변환하는지를 실제 응용 프로그램으로 테스트해 볼 것을 권장한다.
- 😵 타임존 관련 설정은 한 번 문제가 되기 시작하면 해결하기가 매우 어려운 문제가 될 수도 있다.
    
    ```sql
    SHOW VARIABLES LIKE '%time_zone%';
    +------------------+--------+
    | Variable_name    | Value  |
    +------------------+--------+
    | system_time_zone | KST    | # 나의 맥북
    | time_zone        | SYSTEM |
    +------------------+--------+
    ```
    
    - `system_time_zone`: MySQL 서버의 타임존. 일반적으로 이 값은 운영체제의 타임존을 그대로 상속받는다.
    - `time_zone`: MySQL 서버로 연결하는 모든 클라이언트 커넥션의 기본 타임존을 의미한다.
    - RDS
        
        ![Untitled](./images/Untitled%202.png)
        
        ```sql
        SHOW VARIABLES LIKE '%time_zone%';
        +------------------+------------+
        | Variable_name    | Value      |
        +------------------+------------+
        | system_time_zone | UTC        |
        | time_zone        | Asia/Seoul |
        +------------------+------------+
        ```
        

### 3-1. 자동 업데이트

---

- MySQL 5.6 이전 버전까지는 `TIMESTAMP` 타입의 칼럼은 레코드의 다른 칼럼 데이터가 변경될 때마다 시간이 자동 업데이트되고, `DATETIME` 은 그렇지 않은 차이를 가지고 있었다.
- MySQL 5.6 버전부터는 `TIMESTAMP`와 `DATETIME` 칼럼 모두 `INSERT`와 `UPDATE` 문장이 실행될 때마다 해당 시점으로 자동 업데이트되게 하려면 테이블을 생성할 때 칼럼 정의 뒤에 다음 옵션을 정의해야 한다.
    
    ```sql
    CREATE TABLE tb_autoupdate (
        id BIGINT NOT NULL AUTO_INCREMENT,
        title VARCHAR(20),
        created_at_ts TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
        updated_at_ts TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
        created_at_dt DATETIME DEFAULT CURRENT_TIMESTAMP,
        updated_at_dt DATETIME DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
        PRIMARY KEY (id)
    );
    ```
    
    - `DEFAULT CURRENT_TIMESTAMP`: 레코드가 `INSERT`될 떄의 시점을 자동으로 업데이트 한다.
    - `ON UPDATE CURRENT_TIMESTAMP`: 레코드가 `UPDATE`될 때의 시점을 자동으로 업데이트 한다.

👉 MySQL 5.6 버전부터는 `DATETIME` 타입과 `TIMESTAMP` 타입 사이에 커넥션의 `time_zone` 시스템 변수의 타임존으로 저장할지, UTC 로 저장할지의 차이만 남고 모든 것이 같아졌다.

## 4. `ENUM`과 `SET`

---

- `ENUM`과 `SET`은 모두 문자열 값을 MySQL 내부적으로 숫자 값으로 매핑해서 관리하는 타입이다.****

### 4-1. `ENUM`

---

- `ENUM` 타입은 테이블의 구조(메타 데이터)에 나열된 목록 중 하나의 값을 가질 수 있다.
- `ENUM` 타입의 가장 큰 용도는 코드화된 값을 관리하는 것이다.
    
    ```sql
    CREATE TABLE tb_enum (fd_enum ENUM ('PROCESSING', 'FAILURE', 'SUCCESS'));
    INSERT INTO tb_enum VALUES ('PROCESSING'), ('FAILURE');
    SELECT * FROM tb_enum;
    +------------+
    | fd_enum    |
    +------------+
    | PROCESSING |
    | FAILURE    |
    +------------+
    ```
    
- MySQL 서버가 실제로 값을 디스크나 메모리에 저장할 때는 사용자로부터 요청된 문자열이 아니라 그 값에 매핑된 정숫값을 사용한다.
- `ENUM` 타입에 사용할 수 있는 최대 아이템의 개수는 65,535개이며, 아이템의 개수가 255개 미만이면 `ENUM` 타입은 저장 공간으로 1바이트를 사용하고, 그 이상인 경우에는 2바이트까지 사용한다.
- `ENUM` 타입은 저장해야 하는 아이템 값(문자열)이 길면 길수록 저장 공간을 더 많이 절약할 수 있다.

- `ENUM` 타입의 가장 큰 단점: 칼럼에 저장되는 문자열 값이 테이블의 구조(메타 정보)가 되면서 기존 `ENUM` 타입에 새로운 값을 추가해야 한다면 테이블의 구조를 변경해야 한다.
    - 예전 버전의 MySQL 서버에서는 항상 테이블을 리빌드해야 했다.
    - 하지만 MySQL 5.6 버전부터는 새로 추가하는 아이템이 `ENUM` 타입의 제일 마지막으로 추가되는 형태라면 테이블의 구조(메타데이터) 변경만으로 즉시 완료된다.
- 기존 `ENUM` 타입의 아이템들이 순서가 변경되거나 중간에 새로운 아이템이 추가되는 경우에는 `COPY` 알고리즘과 읽기 잠금까지 필요하다.
- `ENUM` 타입은 테이블 구조에 정의된 코드 값만 사용할 수 있게 강제한다는 장점도 있지만, 더 큰 장점은 데이터베이스 서버의 디스크 저장 공간의 크기를 줄여준다는 것이다.

### 4-2. `SET`

---

- `SET` 타입도 테이블의 구조에 정의된 아이템을 정숫값으로 매핑해서 저장하는 방식

👉 `SET`과 `ENUM`의 가장 큰 차이: 하나의 칼럼에 1개 이상의 값을 저장할 수 있다.

- MySQL 서버는 내부적으로 BIT-OR 연산을 거쳐 1개 이상의 선택된 값을 저장한다.
    - 여러 개의 값을 저장할 수는 있지만 실제 여러 개의 값을 저장하는 공간을 가지는 것이 아니다.
    - 그래서 각 아이템값에 매핑되는 정숫값은 1씩 증가하는 정숫값이 아니라 2n 의 값을 갖게 된다.
- `SET` 타입은 아이템 값의 멤버 수가 8개 이하이면 1바이트의 저장 공간을 사용하며, 9개에서 16개 이하이면 2바이트를 사용하고 똑같은 방식으로 최대 8바이트까지 저장 공간을 사용하게 된다.
    
    ```sql
    CREATE TABLE tb_set (
        fd_set SET('TENNIS', 'SOCCER', 'GOLF', 'TABLE-TENNIS', 'BASKETBALL', 'BILLIARD')
    );
    INSERT INTO tb_set (fd_set) VALUES ('SOCCER'), ('GOLF,TENNIS');
    Query OK, 2 rows affected (0.00 sec)
    Records: 2  Duplicates: 0  Warnings: 0
    
    SELECT * FROM tb_set;
    +-------------+
    | fd_set      |
    +-------------+
    | SOCCER      |
    | TENNIS,GOLF |
    +-------------+
    ```
    

- 동등 비교(Equal)를 수행하려면 칼럼에 저장된 순서대로 문자열을 나열해야만 검색할 수 있다.
    
    ```sql
    SELECT * FROM tb_set WHERE fd_set='tennis,golf';
    +-------------+
    | fd_set      |
    +-------------+
    | TENNIS,GOLF |
    +-------------+
    ```
    
- 인덱스가 있더라도 동등 비교 조건을 제외하고 `FIND_IN_SET()` 함수나 `LIKE`를 사용하는 쿼리는 인덱스를 사용할 수 없다.
- `FIND_IN_SET()` 함수: `SET` 타입의 칼럼에 대해 하나의 특정 값을 포함하고 있는지 확인
    
    ```sql
    SELECT * FROM tb_set WHERE FIND_IN_SET('tennis', fd_set) >= 1;
    +-------------+
    | fd_set      |
    +-------------+
    | TENNIS,GOLF |
    +-------------+
    ```
    
- `ENUM`과 마찬가지로 정의된 아이템 중간에 새로운 아이템을 추가하는 경우 테이블의 읽기 잠금과 리빌드 작업이 필요하다.

## 5. `TEXT`와 `BLOB`

---

- MySQL 에서 대량의 데이터를 저장하려면 `TEXT` 나 `BLOB` 타입을 사용해야 한다.
    - 두 타입은 많은 부분에서 거의 똑같은 설정이나 방식으로 작동한다.
- 유일한 차이점: `TEXT` 타입은 문자열을 저장하는 대용량 칼럼이라서 문자 집합이나 콜레이션을 가진다.
    - `BLOB` 타입은 이진 데이터 타입이라서 별도의 문자 집합이나 콜레이션을 가지지 않는다.

- 내부적으로 저장 가능한 최대 길이에 따라 4가지 타입으로 구분한다.
    
    
    | 데이터 타입 | 필요 저장 공간 (L=저장하고자 하는 데이터의 바이트 수) | 최대 저장 가능한 바이트 수 |
    | --- | --- | --- |
    | TINYTEXT, TINYBLOB | L+1 바이트 | 2^8-1 (255) |
    | TEXT, BLOB | L+2 바이트 | 2^16-1 (65,535) |
    | MEDIUMTEXT, MEDIUMBLOB | L+3 바이트 | 2^24-1 (16777,215) |
    | LONGTEXT, LONGBLOB | L+4 바이트 | 2^32-1 (4,294,967,295) |
- `TEXT`, `BLOB` 칼럼 모두 사용할 때 주의하고 너무 남용해서는 안된다. 주로 다음과 같은 상황에서 사용하는 것이 좋다.
    - 칼럼 하나에 저장되는 문자열이나 이진 값의 길이가 예측할 수 없이 클 때 사용한다.
    - 레코드의 전체 크기가 64KB 를 넘어서는 경우
- MySQL 에서 인덱스 레코드의 모든 칼럼은 최대 제한 크기를 가지고 있다.
    - MyISAM 은 1000 바이트
    - `REDUNDANT` 또는 `COMPACT` 로우 포맷을 사용하는 InnoDB 경우: 767 바이트
    - `DYNAMIC` 또는 `COMPRESSED` 로우 포맷을 사용하는 InnoDB 경우: 3072 바이트
- `BLOB`이나 `TEXT` 칼럼으로 정렬을 수행할 때, MySQL 서버의 `max_sort_length` 시스템 변수에 설정된 길이까지만 정렬을 수행한다.
    - 일반적으로 이 설정값은 1024 바이트
    - `TEXT` 타입의 정렬을 더 빠르게 실행하려면 이 값을 줄여서 설정하는 것이 좋다.
- MySQL 8.0 버전부터 TempTable 은 `TEXT`나 `BLOB` 타입을 지원하지만 MEMORY 스토리지 엔진은 `TEXT`나 `BLOB`을 지원하지 않는다.
    - 가능하면 `tmp_mem_storage_engine` 을 `TempTable` 로 설정해서 `BLOB` 타입이나 `TEXT` 타입을 포함하는 결과도 메모리를 사용할 수 있게 하는 것이 좋다.
- `BLOB`이나 `TEXT` 타입의 칼럼이 포함된 SQL 실행시 문장이 매우 길어질 수 있다.
    - MySQL 서버의 `max_allowed_packet` 시스템 변수에 설정된 값 보다 큰 SQL 문장은 전송되지 못하고 오류가 발생할 수도 있다.
    - MySQL 서버의 `max_allowed_packet` 시스템 변수를 필요한 만큼 충분히 늘려서 설정하는 것이 좋다.
- `ROW_FORMAT` 옵션: `BLOB`과 `TEXT` 타입 칼럼의 데이터가 어떻게 저장될지를 결정하는 요소
    - 별도로 지정되지 않으면 `innodb_default_row_format` 시스템 변수에 설정된 값을 적용한다.
    - 설정하지 않으면 기본으로 최신 `ROW_FORMAT`인 `dynamic`이 설정된다.

## 6. `JSON` 타입

---

### 6-1. 저장 방식

---

- MySQL 은 내부적으로 `JSON` 타입의 값을 `BLOB` 타입에 저장한다.
    - 사용자가 입력한 값 그대로 저장하는 것이 아니라 바이너리 포맷인 `BSON` 타입으로 변환해서 저장한다.
    - 공간 효율이 높은 편이다.
- MySQL 서버에서 매우 큰 용량의 `JSON` 도큐먼트가 저장되면 MySQL 서버는 16KB 단위로 여러 개의 데이터 페이지로 나뉘어 저장된다.
    - MySQL 5.7 버전까지는 `BLOB` 페이지들이 단순 링크드 리스트처럼 관리됐다.
        - `JSON` 필드의 부분 업데이트를 효율적으로 처리할 수 없다.
    - MySQL 8.0 버전부터는 `BLOB` 페이지들의 인덱스를 관리하고 각 인덱스는 실제 `BLOB` 데이터를 가진 페이지들의 링크를 갖도록 개선됐다.

### 6-2. 부분 업데이트 성능

---

- MySQL 8.0 버전부터는 `JSON` 타입에 대해 부분 업데이트 기능을 제공한다.
- `JSON_SET()`, `JSON_REPLACE()`, `JSON_REMOVE()` 함수를 이용해 특정 필드 값을 변경하거나 삭제하는 경우에만 작동한다.
    
    ```sql
    UPDATE tb_json
       SET fd=JSON_SET(fd, '$.user_id', '12345')
    WHERE id=2;
    ```
    
- `JSON_SET()` 함수를 이용한 `JSON` 칼럼의 변경 작업이 “부분 업데이트”로 처리됐는지 확인할 수 있는 명확한 방법은 없다 → `JSON_STORAGE_SIZE()` 함수와 `JSON_STORAGE_FREE()` 함수를 이용하면 대략 예측할 수 있다.
- 부분 업데이트 기능은 특정 조건에서는 매우 빠른 업데이트 성능을 보여준다.

### 6-3. `JSON` 타입 콜레이션과 비교

---

- `JSON` 칼럼에 저장되는 데이터와 `JSON` 칼럼으로부터 가공되어 나온 결과값은 모두 `utf8mb4` 문자 집합과 `utf8mb4_bin` 콜레이션을 가진다.
- `utf8mb4_bin` 콜레이션은 바이너리 콜레이션 이기 때문에 대소문자 구분은 물론 액센트 문자 등도 구분해서 비교한다.

### 6-4. `JSON` 컬럼 선택

---

- `JSON` 데이터를 저장해야 한다면 당연히 `BLOB`이나 `TEXT` 칼럼보다는 `JSON` 칼럼을 선택하는 것이 좋다.

❗성능을 중심으로 판단한다면 `JSON` 칼럼보다는 성능적인 이점을 가지고 있는 정규화된 칼럼을 추천한다.

## 7. 가상 칼럼(파생 칼럼)

---

- 다른 DBMS 에서는 가상 컬럼 (Virtual Column) 이라는 이름으로 사용되지만 MySQL 에서는 Generated Column 이라는 이름으로 소개되고 있다.
- MySQL 서버의 가상 컬럼은 크게 가상 컬럼과 스토어드 컬럼으로 구분할 수 있다.
    
    ```sql
    # 가상 칼럼 사용 예제
    CREATE TABLE tb_virtual_column (
        id INT NOT NULL AUTO_INCREMENT,
        price DECIMAL (10, 2) NOT NULL DEFAULT '0.00',
        quantity INT NOT NULL DEFAULT 1,
        total_price DECIMAL (10, 2) AS (quantity * price) VIRTUAL
    );
    
    # 스토어드 칼럼 사용 예제
    CREATE TABLE tb_stored_column (
        id INT NOT NULL AUTO_INCREMENT,
        price DECIMAL (10, 2) NOT NULL DEFAULT '0.00',
        quantity INT NOT NULL DEFAULT 1,
        total_price DECIMAL (10, 2) AS (quantity * price) STORED    
    );
    ```
    
- 가상 칼럼과 스토어드 컬럼 모두 칼럼의 정의 뒤에 `AS` 절로 계산식을 정의한다.
    - `STORED` 키워드가 사용되면 스토어드 칼럼으로 생성된다. (그 이외에는 항상 가상 컬럼)

- 가상 칼럼의 표현식은 입력이 동일하면 결과가 항상 동일한 `DETERMINISTIC` 표현식만 사용할 수 있다.
- MySQL 8.0 버전까지는 가상 칼럼의 표현식에 서브쿼리나 스토어드 프로그램을 사용할 수는 없다.

- 가상 칼럼(Virtual Column)
    - 칼럼 값이 디스크에 저장되지 않음
        - 가상 칼럼에 인덱스를 생성하게 되면 테이블의 레코드는 가상 칼럼을 포함하지 않지만 해당 인덱스는 계산된 값을 저장한다.
    - 칼럼의 구조 변경은 테이블 리빌드를 필요로 하지 않음
        - 인덱스가 생성된 가상 칼럼의 경우 필요하다면 인덱스의 리빌드 작업이 필요하다.
    - 칼럼의 값은 레코드가 읽히기 전 또는 `BEFORE` 트리거 실행 직후에 계산되어 만들어짐
- 스토어드 칼럼(Stored Column)
    - 칼럼의 값이 물리적으로 디스크에 저장됨
    - 칼럼의 구조 변경은 다른 일반 테이블과 같이 필요 시 테이블 리빌드 방식으로 처리됨
    - `INSERT`와 `UPDATE` 시점에만 칼럼의 값이 계산됨

👉 가장 큰 차이는 계산된 칼럼의 값이 디스크에 실제로 저장되는지 여부!

- 함수 기반의 인덱스(Function Based Index)
    - MySQL 8.0 버전부터 도입됨
    - 가상 칼럼에 인덱스를 생성하는 방식으로 작동한다.

- 가상 칼럼 vs 스토어드 칼럼 - 어떤 것을 선택해야 할지?
    - CPU 사용량을 조금 높여서 디스크 부하를 낮출 것이냐
        - 계산 과정이 빠른 반면 상대적으로 결과가 많은 저장 공간을 차지한다 → 가상 칼럼을 선택!
            - 가상 칼럼은 데이터를 조회하는 시점에 매번 계산된다.
    - 디스크 사용량을 조금 높여서 CPU 사용량을 낮출 것이냐
        - 계산하는 과정이 복잡하고 시간이 오래 걸린다 → 스토어드 칼럼을 선택!
