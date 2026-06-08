# MyBatis Patterns 레퍼런스

이 프로젝트의 **MyBatis 데이터 접근 패턴**을 정의한다.

`development` 에이전트는 Mapper/Entity 구현 시, `test-qa` 에이전트는 데이터 레이어 검증 시 이 문서를 참조한다.

---

## 프로젝트 MyBatis 설정

| 항목 | 값 |
|---|---|
| MapperScan 패키지 | `com.kakaohealthcare.mydata.repository.mybatis` |
| XML 위치 | `classpath:com/kakaohealthcare/mydata/**/mybatis/*.xml` |
| SqlSessionFactory | `mybatisSqlSessionFactory` |
| TransactionManager | `transactionManagerMybatis` |
| Config 클래스 | `DataSourceMybatisConfig` |

### 공유 JAR 의존성

Mapper 인터페이스와 XML의 상당수가 `libs/` 공유 JAR에 포함되어 있다:
- `mydata-share-repository` — 공유 Mapper 인터페이스/XML
- `mydata-model-plain` — 도메인 모델 (Entity)

**로컬에 Mapper를 추가할 때:** 동일 패키지 패턴(`com.kakaohealthcare.mydata.repository.mybatis`)을 따라야 MapperScan이 인식한다.

---

## 1. Mapper 인터페이스 패턴

### 기본 구조
```java
package com.kakaohealthcare.mydata.repository.mybatis;

import org.apache.ibatis.annotations.Mapper;
import org.apache.ibatis.annotations.Param;
import java.util.List;

@Mapper
public interface DiagnosticReportMapper {

    void insert(DiagnosticReport report);

    void insertBatch(@Param("list") List<DiagnosticReport> reports);

    DiagnosticReport findByReportId(@Param("reportId") String reportId);

    void upsert(DiagnosticReport report);

    int deleteByUserId(@Param("userId") String userId);
}
```

### 원칙
- `@Mapper` 어노테이션 필수
- 여러 파라미터는 `@Param`으로 이름 지정
- 메서드명은 동사 시작: `insert`, `find`, `update`, `delete`, `upsert`, `count`
- 기존 공유 JAR의 Mapper 네이밍 패턴을 먼저 확인하고 따른다

---

## 2. XML Mapper 패턴

### 파일 위치
```
src/main/resources/com/kakaohealthcare/mydata/repository/mybatis/
    └── DiagnosticReportMapper.xml
```

### 기본 구조
```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
    "http://mybatis.org/dtd/mybatis-3-mapper.dtd">

<mapper namespace="com.kakaohealthcare.mydata.repository.mybatis.DiagnosticReportMapper">

    <resultMap id="diagnosticReportMap" type="com.kakaohealthcare.mydata.model.DiagnosticReport">
        <id property="id" column="id"/>
        <result property="userId" column="user_id"/>
        <result property="reportId" column="report_id"/>
        <result property="type" column="type"/>
        <result property="value" column="value"/>
        <result property="createdAt" column="created_at"/>
    </resultMap>

    <insert id="insert" parameterType="DiagnosticReport">
        INSERT INTO diagnostic_report (user_id, report_id, type, value, created_at)
        VALUES (#{userId}, #{reportId}, #{type}, #{value}, #{createdAt})
    </insert>

    <select id="findByReportId" resultMap="diagnosticReportMap">
        SELECT * FROM diagnostic_report
        WHERE report_id = #{reportId}
    </select>

</mapper>
```

### 원칙
- **namespace**: Java 인터페이스의 FQCN과 일치
- **resultMap**: `id` + `result` 매핑 (Java 필드명 ↔ DB 컬럼명)
- **parameterType**: 생략 가능하나 복잡한 경우 명시
- 기존 XML 구조/네이밍을 확인 후 동일하게 작성

---

## 3. SQL 파라미터 바인딩

### `#{}` vs `${}` 규칙

| 구문 | 용도 | Injection | 사용 |
|---|---|---|---|
| `#{param}` | 값 바인딩 (PreparedStatement) | **안전** | **기본값** |
| `${param}` | 문자열 치환 (직접 삽입) | **위험** | 원칙적 금지 |

**상세 규칙과 예제: [보안 체크리스트](./security-checklist.md) §1 참조**

### `${}` 허용되는 유일한 경우
- 동적 테이블명/컬럼명 (화이트리스트 검증 필수)
- ORDER BY 절의 동적 정렬 컬럼 (화이트리스트 검증 필수)

---

## 4. 동적 SQL 패턴

### if 태그
```xml
<select id="findByCondition" resultMap="reportMap">
    SELECT * FROM diagnostic_report
    WHERE user_id = #{userId}
    <if test="type != null">
        AND type = #{type}
    </if>
    <if test="startDate != null">
        AND created_at >= #{startDate}
    </if>
</select>
```

### foreach (배치 삽입)
```xml
<insert id="insertBatch">
    INSERT INTO diagnostic_report (user_id, report_id, type, value)
    VALUES
    <foreach collection="list" item="item" separator=",">
        (#{item.userId}, #{item.reportId}, #{item.type}, #{item.value})
    </foreach>
</insert>
```

### where / set 태그
```xml
<update id="updateSelective">
    UPDATE diagnostic_report
    <set>
        <if test="type != null">type = #{type},</if>
        <if test="value != null">value = #{value},</if>
    </set>
    WHERE report_id = #{reportId}
</update>
```

### 원칙
- `<if test="...">` 조건은 **null 체크 우선**
- `<foreach>` 사용 시 리스트가 비어있을 때 SQL 에러 방지 (호출 전 empty 체크)
- `<where>` / `<set>` 태그로 불필요한 AND/콤마 자동 처리

---

## 5. Upsert (중복 시 업데이트) 패턴

### MySQL ON DUPLICATE KEY UPDATE
```xml
<insert id="upsert">
    INSERT INTO diagnostic_report (user_id, report_id, type, value)
    VALUES (#{userId}, #{reportId}, #{type}, #{value})
    ON DUPLICATE KEY UPDATE
        type = VALUES(type),
        value = VALUES(value),
        updated_at = NOW()
</insert>
```

### 복합키 Upsert
```xml
<insert id="upsertByCompositeKey">
    INSERT INTO diagnostic_report (user_id, report_id, type, value)
    VALUES (#{userId}, #{reportId}, #{type}, #{value})
    ON DUPLICATE KEY UPDATE
        type = #{type},
        value = #{value},
        updated_at = NOW()
</insert>
```

### 원칙
- `ON DUPLICATE KEY UPDATE`에는 **변경 가능한 컬럼만** 나열
- PK/UK 컬럼은 UPDATE 절에 넣지 않는다
- `VALUES()` 또는 재바인딩 (`#{field}`) 중 기존 패턴을 따른다

---

## 6. 트랜잭션 경계

### 규칙
- **한 Bundle 저장 = 하나의 트랜잭션** (FHIR 파싱 결과)
- Saver 레이어의 저장 메서드에 `@Transactional(transactionManager = "transactionManagerMybatis")` 적용
- 부분 실패 시 **전체 롤백** (데이터 정합성 우선)

### 1:N 저장 순서
```java
@Transactional(transactionManager = "transactionManagerMybatis")
public void save(ParseResult result) {
    // 1. 부모 먼저
    encounterMapper.insertBatch(result.getEncounters());
    // 2. 자식 나중 (FK 참조)
    observationMapper.insertBatch(result.getObservations());
    reportMapper.insertBatch(result.getReports());
}
```

### 주의
- `@Transactional`을 Service에 걸면 Mapper 호출이 하나의 트랜잭션으로 묶임
- 잘못된 위치에 걸면 **Mapper 호출마다 별도 트랜잭션** → 부분 실패 시 정합성 깨짐

---

## 7. N+1 문제 방지

### 나쁜 예
```java
// N+1: 유저 목록 조회 후 각각 리포트 조회
List<User> users = userMapper.findAll();
for (User user : users) {
    List<Report> reports = reportMapper.findByUserId(user.getId());  // N번 실행
}
```

### 좋은 예
```java
// 한 번에 조회 (JOIN 또는 IN)
List<UserWithReports> result = userMapper.findAllWithReports();

// 또는 ID 모아서 IN 쿼리
List<String> userIds = users.stream().map(User::getId).toList();
List<Report> reports = reportMapper.findByUserIds(userIds);
```

### XML에서 IN 쿼리
```xml
<select id="findByUserIds" resultMap="reportMap">
    SELECT * FROM diagnostic_report
    WHERE user_id IN
    <foreach collection="userIds" item="id" open="(" separator="," close=")">
        #{id}
    </foreach>
</select>
```

---

## 8. 인덱스 고려사항

### 규칙
- `WHERE`, `JOIN`, `ORDER BY`에 자주 쓰이는 컬럼에 인덱스
- 복합키 테이블의 PK 구조를 쿼리가 활용하는지 확인 (선두 컬럼 순서)
- 개발 플랜에 DDL 변경이 있으면 인덱스 설계도 포함

### 체크 포인트
- 새 쿼리가 기존 인덱스를 활용하는가? (`EXPLAIN` 확인)
- 풀 테이블 스캔이 발생하지 않는가?

---

## 9. QA 검증 시 데이터 레이어 체크 항목

| 항목 | 기준 | 위반 시 심각도 |
|---|---|---|
| SQL Injection | `#{}` 사용, `${}` 화이트리스트 검증 | Critical |
| 트랜잭션 경계 | Bundle 단위, Saver에 `@Transactional` | Major |
| 1:N 저장 순서 | 부모 먼저, 자식 나중 | Major |
| N+1 쿼리 | `for`문 안에서 단건 조회 | Major |
| 인덱스 활용 | 풀 테이블 스캔 방지 | Minor |
| Mapper namespace | Java 인터페이스 FQCN과 일치 | Minor |
| foreach 빈 리스트 | 호출 전 empty 체크 | Minor |

---

## 참조

이 문서를 읽는 에이전트:
- **development.md** — Mapper/Entity 구현 시 §1~§8 참조
- **test-qa.md** — QA 체크리스트의 "데이터" 항목에서 §9 체크 항목 적용
- **development-plan.md** — Mapper 설계, DDL 변경 계획 시

관련 레퍼런스:
- [보안 체크리스트](./security-checklist.md) §1 — SQL Injection 상세
- [FHIR 패턴](./fhir-patterns.md) §11 — Saver 레이어 연계
