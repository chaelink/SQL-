### 핵심노트 Chapter 6
소트 튜닝 종류
- Sort Runs : Sort Area가 찰 때마다 Temp 영역에 저장해 둔 중간 단계의 집합
- Optimal 소트 : 정렬 대상 집합이 크지 않아 Sort Area 내에서 작업을 마무리하는 것
- Onepass 소트 : 실행계획에 나타난 하나의 소트 오퍼레이션에 대해 정렬 대상 집합을 **디스크에 한번만 기록하고 작업을 마치는 것**
- Multipass 소트 : 디스크에 여러 번 기록하는 것

<br>

소트 오퍼레이션 종류
- Sort Aggregate : sum, min, max 등 집계 계산 / Sort Area를 가장 적게 사용
- Sort Order By : 전체 데이터 정렬 / Sort Area를 가장 많이 사용
- Sort Group By : 전체 데이터를 정렬하지는 않음 / 결과집합 건수만큼의 Sort Area 사용
- Hash group By : Sort 알고리즘 대신 Hash 알고리즘을 사용한다는 점만 다름
- Sort Unique

<br>

Direct Path I/O
- (Sort Area 부족) -> Sort Area에 정렬된 데이터를 Temp 테이블스페이스에 쓰고 이를 다시 읽어 들일 때 사용
- I/o call이 완료될 때까지 대기 발생
    - direct path write temp
    - direct path read temp

<br>

커밋
- 커밋 발생 시, 네트워크를 경유한 db call 발생

<br>

배치 프로그램
= 부분범위처리 불가
- 병렬 처리 가능
- nologging 옵션은 insert 시에만 사용 가능
- Array Processing 가능

<br>

데이터베이스 Call
- 애플리케이션 커서를 캐싱하지 않는 한, 바인드 변수를 사용해도 Parse Call은 매번 일어남
- DML은 Parse call 단계를 제외하면 모든 I/O가 Execute call 단계에서 발생
- select 문은 대부분 i.o가 Fetch Call 단계에서 발생
- Parse Call 단계에서 Recursive Call 발생 가능 (하드 파싱)
- Execute Call 단계에서 Recursive Call 발생 가능
- Fetch Call 단계에서 Recursive Call 발생 가능

<br>

Array Processing
- 성능 효과의 핵심 부분 : 데이터베이스 call 감소
- java 프로그램은 네트워크를 경유하므로, PL/SQL 보다 데이터베이스 부하가 더 큼, 따라서 Array Processing 적용 후의 성능 효과도 더 큼
- Array 단위를 늘리면 성능이 좋아지지만 개선율 감소, 적정 크기로 설정해야 함
- ArraySize를 작게 설정하거나, 조건절을 충족하는 데이터가 많아서 빨리 채울수록 응답속도가 빠름

---
### 헷갈리는 내용
- Sort Group By, Hash group By 차이
    -  Hash Group By는 해시 함수를 사용하여 메모리(Hash Area)에서 빠르게 그룹핑하여 대용량 데이터에 유리하지만 정렬 순서를 보장하지 않음
    -  Sort Group By는 데이터를 먼저 정렬(Sort Area)한 후 그룹핑하여 정렬된 결과를 얻을 수 있으나 데이터 양이 많으면 성능 저하
- GROUP BY에서 소트 연산 생략(부분 범위 처리) 가능하게 인덱스 짜는 방법
    - group by 컬럼이 선두 컬럼인 인덱스를 사용하면 가능 
- "Top N STOPKEY 알고리즘" 작동 가능한 SQL 쿼리 작성 법
    - 인라인 뷰 안쪽에 ORDER BY 명시
    - 바로 바깥쪽에 ROWNUM <= 조건 명시
- 소트 연산 생략이 가능하도록 하는 인덱스 구성 공식
    - '=' 연산자로 사용한 조건절 컬럼 선정
    - ORDER BY 절에 기술한 컬럼 추가
    - '=' 연산자가 아닌 조건절 컬럼은 데이터 분포를 고려해 추가 여부 결정 

- 커밋, 서버 프로새스가 변경한 블록이 커밋 시점에 바로 데이터파일에 기록되지는 않는다. ?
- 친절한 sql 튜닝에서 `INSERT INTO SELECT` 내용 다시 확인하기
- 페이징 처리 기법을 사용하면 데이터베이스 Call 부하를 줄인다.
    - 왜?, 페이징 지정 값까지만 출력하고 멈추기 때문에? 
- I/O Call을 정확히 어디로 날리는 건지
- 




---



소트 연산 수행 과정
- PGA에 할당한 sort area에서 수행하며 pga가 부족하면 temp스페이스 사용
- 메모리 소트 : pga 내에서 소트 완료하는 경우
- 디스크 소트 : temp 테이블 스페이스까지 사용하여 소트를 완료하는 경우

SGA(System Global Area)
- 오라클 인스턴스 시작 시 할당, 모든 사용자 세션이 공유하는 메모리 영역
- db 내에 단 하나만 존재
- 주요 구성 요소 
    - Shared Pool : SQL문, 데이터 딕셔너리 정보 공유
    - Database Buffer Cache : 디스크에서 읽은 데이터 블록 저장
    - Redo Log Buffer : 변경된 데이터의 로그 정보 저장

PGA(Program Global Area)
- 개별 서버 프로세스에게 할당되는 전용 메모리 공간
- 사용자 세션마다 독립적으로 존재
- 주요 용도
    - 세션 정보 관리
    - 소트 작업
    - 조인 워크스페이스 

버퍼캐시
- 디스크에서 한 번 읽어온 데이터를 메모리에 임시로 저장하여,
- 이후 동일한 데이터 요청 시 느린 디스크 io 대신 빠른 메모리 접근을 가능하게 함

버퍼캐시 vs 일반캐시 차이
- 버퍼 캐시 : 디스크io 등 특정 입출력 효율화를 위해 데이터를 일시 보관 후 삭제
- 일반 캐시 : 자주 쓰는 데이터의 재사용을 위해 복사본을 지속적으로 저장







