- - Real MySQL 8.0

    

    # #6 데이터 압축

    - MySQL 서버에서 디스크에 저장된 데이터 파일의 크기는 일반적으로 쿼리의 처리 성능과도 직결되지만 백업 및 복구 시간과도 밀접하게 연결된다.
    - 디스크 데이터 파일이 크면 클수록 비용, 속도문제, 성능문제 등으로 많은 DBMS가 이런 문제점들을 해결하기 위해 데이터 압축 기능을 제공한다.

    

    ## 페이지 압축

    - MySQL 서버에서는 디스크에 저장하는 시점에 데이터 페이지가 압축되어 저장되고, 반대로 MySQL 서버가 디스크에서 데이터 페이지를 읽어올 때 압축이 해제된다.
      - 버퍼 풀에 데이터 페이지가 적재되면 InnoDB 스토리지 엔진은 압축이 해제된 상태로만 데이터 페이지를 관리한다.
      - 이때 문제가 되는게 예를 들어 16kb 데이터 페이지를 압축한 결과가 용량이 얼마나 될지 예측이 불가능하고 하나의 테이블은 동일한 크기의 페이지로 통일돼야 한다는 것
    - 그래서 페이지 압축 기능은 운영체제별로 특정 버전의 파일 시스템에서만 지원되는 펀치홀이라는 기능을 사용한다.

    `펀치홀 동작방식`

    1. 16kb 페이지를 압축 (압축 결과를 7kb로 가정)
    2. MySQL 서버는 디스크에 압축된 결과 7kb를 기록 (이때 MySQL 서버는 압축 데이터 7kb에 9kb의 빈 데이터를 기록)
    3. 디스크에 데이터를 기록한 후, 7kb 이후의 공간 9kb에 대해 펀치 홀을 생성
    4. 파일 시스템은 7kb만 남기고 나머지지 디스크의 9kb공간은 다시 운영체제로 반납

    `펀치홀 단점`

    - 펀치홀 기능은 운영체제뿐만 아니라 하드웨어 자체에서도 해당 기능을 지원해야 한다.
    - 아직 파일 시스템 관련 명령어가 펀치홀을 지원하지 못한다.
      - ex: 펀치홀이 적용되어 1gb로 압축되었다고 하더라도 `cp` 명령어로 복사하면 펀치홀이 다시 채워져 원본 크기인 10gb가 될 수 있다.
    - 이런한 이유로 실제 페이지 압축은 많이 사용되지 않는다.

    

    `페이지 압축을 이용하기`

    ```sql
    // 테이블 생성시
    mysql> CREATE TABLE t1 (c1 INT) COMPRESSION="zlib"
    
    // 테이블 변경시
    mysql> ALTER TABLE t1 COMPRESSION="zlib"
    mysql> OPTIMIZE TABLE t1;
    ```

    

    ## 테이블 압축

    - 테이블 압축은 운영체제나 하드웨어에 대한 제약이 없기 때문에 일반적으로도 활용도가 높다.
    - 하지만 아래와 같은 단점이 있다.

    `테이블 압축의 단점`

    - 버퍼 풀 공간 활용률이 낮다.
    - 쿼리 처리 성능이 낮다.
    - 빈번한 데이터 변경시 압축률이 떨어진다.

    

    ### 압축 테이블 생성

    - 테이블 압축을 사용하기 위한 전제 조건으로 압축을 사용하려는 테이블이 별도의 테이블 스페이스를 사용해야 한다.
      - `innodb_file_per_table` 시스템 변수가 ON으로 설정된 상태에서 테이블이 생성돼야 한다.
    - 테이블 압축을 사용하는 테이블은 테이블을 생성할 때 `ROW_FORMAT=COMPRESSED` 옵션을 명시해야 한다.
    - `KEY_BLOCK_SIZE`(2n, n 값은 2 이상)을 이용해 압축된 페이지의 타깃 크기를 명시한다.
      - InnoDB 스토리지엔진의 페이지크기가 16kb라면 4kb 또는 8kb만 설정이 가능하다.
    - 페이지크기가 32kb 또는 64kb인 경우에는 테이블 압축을 적용할 수 없다.

    ```sql
    mysql> SET GLOBAL innodb_file_per_table=ON;
    
    -- // ROW_FORMAT 옵션과 KEY_BLOCK_SIZE 옵션을 모두 명시
    mysql> CREATE TABLE compressed_table (
      c1 INT PRIMARY KEY
    )
    ROW_FORMAT=COMPRESSED
    KEY_BLOCK_SIZE=8;
    
    -- //KEY_BLOCK_SIZE 옵션만 명시
    mysql> CREATE TABLE compressed_table (
      c1 INT PRIMARY KEY
    )
    KEY_BLOCK_SIZE=8;
    
    -- //위와 같은 경우에는 자동으로 ROW_FORMAT=COMPRESSED 옵션이 추가되어 생성된다.
    -- //KEY_BLOCK_SIZE의 옵션값은 kb
    
    ```

    

    `InnoDB 스토리지 엔진이 압축을 적용하는 방법`

    - InnoDB 스토리지 엔진의 데이터 페이지 크기가 16kb, `KEY_BLOCK_SIZE`= 8 이라고 가정
      1. 16kb의 데이터 페이지를 압축
         1. 압축된 결과가 8kb 이하이면 그대로 디스크에 저장 (압축 완료)
         2. 압축된 결과가 8kb를 초과하면 원본 페이지를 스플릿해서 2개의 페이지에 8kb씩 저장

      2. 나뉜 페이지 각각에 대해 1번 단계를 반복 실행  

    - 테이블 압축 방식에서 가장 중요한 것은 원본 데이터 페이지의 압축 결과가 목표크기(`KEY_BLOCK_SIZE`) 보다 작거나 같을 때까지 반복해서 페이지를 스플릿하는 것이다.
    - 목표 크기가 잘못 설정되면 MySQL 서버의 처리 성능이 급격히 떨어질 수 있음을 주의!

    

    ### KEY_BLOCK_SIZE 결정

    - `KEY_BLOCK_SIZE` 가 테이블 압축에서 가장 중요하기 때문에 4kb, 8kb로 테이블을 생성해서 샘플데이터를 저장해보고 적절한지 판단하는 것이 좋다.
      - 최소한 테이블의 데이터 페이지가 10개 정도는 되어야 유의미한 테스트가 된다.

    

    `테스트해보기 (KEY_BLOCK_SIZE=4)`

    ```sql
    CREATE TABLE `employees_comp4k` (
      `emp_no` int NOT NULL,
      `birth_date` date NOT NULL,
      `first_name` varchar(14) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci NOT NULL,
      `last_name` varchar(16) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci NOT NULL,
      `gender` enum('M','F') CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci NOT NULL,
      `hire_date` date NOT NULL,
      PRIMARY KEY (`emp_no`),
      KEY `ix_hiredate` (`hire_date`),
      KEY `ix_gender_birthdate` (`gender`,`birth_date`),
      KEY `ix_firstname` (`first_name`)
    ) ROW_FORMAT=COMPRESSED
    KEY_BLOCK_SIZE=4;
    
    -- // 인덱스별로 압축 실행 횟수와 성공횟수를 기록하기 위한 설정
    SET GLOBAL innodb_cmp_per_index_enabled=ON;
    
    -- // employees 테이블의 데이터를 압축 테스트 테이블로 저장
    INSERT INTO employees_comp4k SELECT * FROM employees;
    
    -- // 인덱스 별로 압축 횟수와 성공 횟수, 압축 실패율을 조회
    SELECT table_name, index_name, compress_ops, compress_ops_ok, (compress_ops-compress_ops_ok)/compress_ops * 100 as compression_failure_pct
    FROM information_schema.INNODB_CMP_PER_INDEX;
    ```

    

    ![1](https://github.com/sup2is/dev-note/raw/master/db/mysql/images/real-mysql-8/1.png)

    - PK는 전체 18653번을 압축을 실행하고 그 중에서 13478번 성공했다.
    - 5175(18653 - 13478)번 실패했는데 압축의 결과가 4kb를 초과해서 데이터 페이지를 스플릿해서 다시 압축을 실행했다는 의미다.
    - PK 압축 실패율 외에도 나머지 인덱스 2개도 압축 실패율이 상대적으로 높은편인데 `KEY_BLOCK_SIZE` 는 압축 실패율을 3~5% 사이로 둘 수 있게 설정하는게 좋다.

    `테스트해보기 (KEY_BLOCK_SIZE=8)`

    ![2](https://github.com/sup2is/dev-note/raw/master/db/mysql/images/real-mysql-8/2.png)

    - `KEY_BLOCK_SIZE` 가 8kb여도 PK 키의 압축 실패율이 높기때문에 이 결과를 기준으로 압축을 적용하면 압축 실패율이 꽤 높아서 InnoDB 버퍼풀에서 디스크로 기록되기 전에 압축하는 과정에서 시간이 꽤 걸릴 수 있다.
    - 성능에 민감한 서비스라면 이 테이블은 압축을 적용하지 않는 것이 좋다고 판단할 수 있다.

    `압축된 모습`

    ![3](https://github.com/sup2is/dev-note/raw/master/db/mysql/images/real-mysql-8/3.png)

    - `KEY_BLOCK_SIZE` 가 4kb나 8kb나 압축후 결과가 크게 차이안나므로 이 경우 8kb를 사용하는게 좋다.
    - 압축 실패율이 높다고해서 압축을 사용하지 말라는 뜻은 아니다. 스플릿이 일어나더라도 데이터파일의 크기가 큰 폭으로 줄어든다면 큰 손해는 아니다. 하지만 데이터가 매우 빈번하게 조회되고 변경된다면 압축 실패율이 낮더라도 압축을 고려하지 않는게 좋다.
    - 테이블 압축은 zlib을 이용해 압축을 실행하는데 이 압축 알고리즘은 많은 cpu 자원을 소모한다.

    

    ### 압축된 페이지의 버퍼 풀 적재 및 사용

    - InnoDB 스토리지 엔진은 디스크에서 읽은 상태 그대로의 데이터 페이지 목록을 관리하는 LRU 리스트와 압축된 페이지들의 압축 해제 버전인 Unzip_LRU 리스트를 별도로 관리한다.
    - MySQL 서버에는 압축된 테이블과 압축되지 않은 테이블이 공존하므로 결국 LRU 리스트는 다음과 같이 두 개의 페이지를 모두 가질 수 있다.
      - 압축이 적용되지 않은 테이블의 데이터 페이지
      - 압축이 적용된 테이블의 압축된 데이터 페이지
    - InnoDB 스토리지 엔진은 압축된 테이블에 대해서 버퍼 풀의 공간을 이중으로 사용함으로써 메모리를 낭비하는 효과를 가진다.
    - 하지만 압축 및 압축 해제 작업은 CPU를 상대적으로 많이 소모하기 때문에 Unzip_LRU 리스트를 별도로 관리하고 있다가 MySQL 서버로 유입되는 요청 패턴에 따라 적절히 다음과 같은 처리를 수행한다.
      - InnoDB 버퍼 풀의 공간이 필요한 경우엔 LRU 리스트에서 원본 데이터 페이지는 유지하고 Unzip_LRU 리스트에서 압축 해제된 버전은 제거해서 버퍼풀의 공간을 확보한다.
      - 압축된 데이터 페이지가 자주 사용되는 경우에는 Unzip_LRU 리스트에 압축 해제된 페이지를 계속 유지하면서 압축 및 압축 해제 작업을 최소화한다.
      - 압축된 데이터 페이지가 사용되지 않아서 LRU 리스트에서 제거되는 경우에는 Unzip_LRU 리스트에서도 함께 제거된다.

    

    `InnoDB 스토리지 엔진의 어댑티브 알고리즘`

    - CPU 사용량이 높은 서버에는 가능하면 압축과 압축 해제를 피하기 위해 Unzip_LRU 비율을 높여서 유지한다.
    - Disk IO 사용량이 높은 서버에는 가능하면 Unzip_LRU 리스트의 비율을 낮춰서 InnoDB 버퍼 풀의 공간을 더 확보하도록 작동한다.

    

    ### 테이블 압축 관련 설정

    - 테이블 압축과 연관된 시스템 변수

    `innodb_com_per_index_enabled`

    - 테이블 압축이 사용된 테이블의 모든 인덱스별로 압축 성공 및 압축 실행 횟수를 수집하도록 설정한다.
    - OFF 일 경우 테이블 단위의 압축 성공 및 압축 실행 횟수만 수집된다.
    - 테이블 단위로 수집된 정보는 `infomation_schema.INNODB_CMP`
    - 인덱스 단위로 수집된 정보는 `infomation_schema.INNODB_CMP_PER_INDEX`

    `innodb_compression_level`

    - 이 변수를 사용해서 압축률을 설정할 수 있다.
    - 0~9의 값
    - 값이 작을수록 압축 속도는 빨라지지만 저장 공간은 커질 수 있다. 반대로 값이 커질수록 속도는 느려질 수 있지만 압축률은 높아진다.
    - 기본값은 6
    - 압축속도가 빨라진다 = CPU 자원을 적게 사용한다.

    `innodb_compression_failure_threshold_pct와 innodb_compression_pad_pct_max`

    - 테이블 단위로 압축 실패율이 `innodb_compression_failure_threshold_pct` 보다 커지면 압축을 실행하기 전 원본 데이터 페이지의 끝에 의도적으로 빈 패딩 공간을 추가한다.
    - 추가된 빈 공간은 압축률을 높여서 압축 결과가 `KEY_BLOCK_SIZE` 보다 작아지게 만드는 효과를 낸다.
    - 이 패딩 공간은 압축 실패율이 높아질수록 계속 증가된 크기를 갖는데 추가할 수 있는 패딩공간의 최대 공간을 `innodb_compression_pad_pct_max` 으로 설정할 수 있다. 
    - `innodb_compression_pad_pct_max` 는 % 값이고 전체 데이터 페이지 크기 대비 패딩 공간의 비율을 의미한다.

    `innodb_log_compressed_pages`

    - MySQL 서버가 비정상적으로 종료됐다가 다시 시작되는 경우 압축 알고리즘의 버전 차이가 있더라도 복구 과정이 실패하지 않도록 InnoDB 스토리지 엔진은 압축된 데이터 페이지를 그대로 리두 로그에 기록한다. 
    - 이는 압축 알고리즘을 업그레이드할 때 도움이 되지만, 데이터 페이지를 통째로 리두 로그에 저장하는 것은 리두 로그 증가량에 상당한 영향력을 미칠 수 있다.
    - 압축을 적용한 후 리두 로그 용량이 매우 빠르게 증가한다거나 버퍼 풀로부터 더티 페이지가 한꺼번에 많이 기록되는 패턴으로 바뀌었다면 `innodb_log_compressed_pages` 을 OFF로 바꾼뒤 모니터링해보는 것이 좋다.
    - 기본값은 ON, ON으로 유지시키는것을 권장한다.
