---
title: "DnDn — 건설 현장관리 ERP 시스템"
permalink: /project/dndn/
layout: single
live_demo_pending: true
author_profile: false
sidebar:
  nav: "projects_nav"
toc: true
toc_label: "목차"
toc_sticky: true
---

<div class="port-meta-bar">
  <div class="port-meta-item">
    <div class="port-meta-label">기간</div>
    <div class="port-meta-value">2026.04 — 2026.06</div>
  </div>
  <div class="port-meta-item">
    <div class="port-meta-label">구성</div>
    <div class="port-meta-value">BE · FE · APP 3개 레포</div>
  </div>
  <div class="port-meta-item">
    <div class="port-meta-label">아키텍처</div>
    <div class="port-meta-value">Multimodule · MSA</div>
  </div>
  <div class="port-meta-item">
    <div class="port-meta-label">GitHub</div>
    <div class="port-meta-value">
      <a href="https://github.com/beyond-sw-camp/be24-fin-Intelli_J-DnDn-BE" target="_blank">BE</a> &nbsp;·&nbsp;
      <a href="https://github.com/beyond-sw-camp/be24-fin-Intelli_J-DnDn-APP" target="_blank">APP</a> &nbsp;·&nbsp;
      <a href="https://github.com/beyond-sw-camp/be24-fin-Intelli_J-DnDn-FE" target="_blank">FE</a>
    </div>
  </div>
</div>

## 프로젝트 개요

- 건설 현장의 인력·근태·안전·공정·문서·ESG 데이터를 실시간으로 수집·통합 관리하는 ERP 플랫폼
- 현장 소장·관리자가 근로자 현황, 게이트 혼잡도, 공정 진행률, 일일 보고를 대시보드 한 화면에서 모니터링
- MSA 기반 Multimodule 구조(dndn-core, document-management, gateway, discovery)와 외부 HR 연동·피로도 산정 배치 파이프라인

## 담당 기능

- **인증/인가** — RBAC+ABAC 혼합 인가 구현 (dndn-core / auth 패키지)
- **인력 배치** — 현장·공종별 인력 배치 API, 피로도 기반 구역 재배정 로직 (dndn-core / staffing 패키지)
- **근무자 관리** — 근로자 프로필·출결·사고 이력 관리 API (dndn-core / worker 패키지)
- **Mobile 연동** — Ionic Vue 앱 ↔ dndn-core REST API 연동 설계 및 구현
- **인력 데이터 조회 자동화** — 외부 HR 수집 → 검증 → 운영 테이블 동기화 7 Step 배치 파이프라인 (dndn-batch 모듈 전체)
- **Kubernetes 환경 관리 및 운영** — Jenkins를 활용한 CI/CD, Blue/Green 무중단 배포

## 기술 스택

<div class="port-tech-groups">
  <div class="port-tech-group">
    <div class="ptg-label">Backend</div>
    <div class="ptg-badges">
      <span class="port-badge pb-b">Java</span>
      <span class="port-badge pb-b">Spring Boot</span>
      <span class="port-badge pb-b">Spring Security</span>
      <span class="port-badge pb-b">Spring Batch</span>
    </div>
  </div>
  <div class="port-tech-group">
    <div class="ptg-label">Database</div>
    <div class="ptg-badges">
      <span class="port-badge pb-g">MariaDB</span>
      <span class="port-badge pb-g">JPA</span>
    </div>
  </div>
  <div class="port-tech-group">
    <div class="ptg-label">Infra</div>
    <div class="ptg-badges">
      <span class="port-badge pb-o">Kubernetes</span>
      <span class="port-badge pb-o">Istio</span>
      <span class="port-badge pb-o">Docker</span>
      <span class="port-badge pb-o">Jenkins</span>
    </div>
  </div>
  <div class="port-tech-group">
    <div class="ptg-label">Architecture</div>
    <div class="ptg-badges">
      <span class="port-badge pb-r">MSA</span>
    </div>
  </div>
  <div class="port-tech-group">
    <div class="ptg-label">Monitoring</div>
    <div class="ptg-badges">
      <span class="port-badge pb-s">Prometheus</span>
      <span class="port-badge pb-s">Grafana</span>
    </div>
  </div>
</div>

---

## 기술 의사결정

### 1. RBAC + ABAC 혼합 인가 설계

역할 기반 접근 제어(RBAC)만으로 설계를 시작했을 때, "근로자는 자신이 배치된 현장의 출결·서류 데이터만 열람 가능하다"는 요구사항을 표현하기 어려웠습니다. 모든 근로자에게 동일한 `ROLE_WORKER`를 부여하면 타 현장 데이터에도 접근 가능해지는 허점이 생기고, 이를 RBAC 안에서 해결하려면 현장마다 별도 역할을 생성해야 해 역할이 폭발적으로 증가합니다.

순수 ABAC(속성 기반)로만 구성하면, 기본적인 역할 구분(관리자/고용주/근로자)마저 모든 요청마다 속성 평가 연산이 필요해 불필요한 오버헤드가 발생합니다.

<div class="port-decision">
<strong>혼합 전략</strong>: RBAC로 역할 수준의 1차 접근 통제를 처리합니다(현장 생성은 관리자·고용주만 가능, 근로자는 조회만 가능). 역할 체크를 통과한 요청에 한해서만 ABAC로 리소스 소유권을 검증합니다(요청 파라미터의 현장 ID가 자신이 배치된 현장 ID와 일치하는지 확인). 역할 체크가 실패하면 속성 평가 없이 즉시 거부되므로, 연산 비용이 높은 ABAC 평가를 최소한의 요청에만 적용합니다.
</div>

---

### 2. K8s CronJob 트리거 전환

초기 구현은 `@Scheduled(cron = "0 30 0 * * *")`로 매일 00:30에 외부 HR 시스템의 인력 정보를 DB에 동기화했습니다. 이 방식은 세 가지 문제를 만들었습니다.

**HTTP 요청 라이프사이클에 처리가 묶임**: 프론트에서 수동 트리거를 시도할 경우 처리 시간이 길어져 HTTP timeout이 발생했습니다. 클라이언트 입장에서는 실패처럼 보이지만 백엔드는 계속 실행 중인 불투명한 상태가 연출됐습니다.

**API 서버와 JVM 공유**: 배치 처리가 API 서버와 같은 JVM에서 동작하므로, 야간 대량 처리 중 API 응답 지연이 유발됐습니다. 배치 부하가 증가할수록 서비스 전체의 응답성에 영향을 미쳤습니다.

**실행 이력 부재**: 성공·실패 여부를 로그로만 파악해야 했고, 특정 현장 처리 중 예외가 발생하면 이후 모든 현장이 중단되는 격리 부재 문제도 있었습니다.

<div class="port-decision">
K8s CronJob으로 전환하면서 배치를 <strong>독립 Pod</strong>에서 실행하도록 분리했습니다. 배치 실패가 API 서버에 영향을 주지 않고, <code>concurrencyPolicy: Forbid</code>로 이전 실행이 남아있으면 다음 실행을 건너뛰며, <code>backoffLimit: 2</code>로 실패 시 자동 재시도합니다.<br><br>
K8s CronJob에서 curl로 백엔드 API를 호출하는 방식(Architecture C)도 검토했습니다. 이는 별도 배치 프로젝트 없이 단순하지만, 백엔드 JVM을 공유하므로 부하 분리가 되지 않고 Spring Batch 메타 테이블 기반 실행 이력도 남지 않습니다. <strong>부하 분리와 실행 이력이 운영 핵심 요건</strong>이었기 때문에 독립 배치 프로젝트(Architecture A)를 채택했습니다.
</div>

---

### 3. Spring Batch Tasklet + Chunk 혼합 방식 적용

초기에는 API 서버(dndn-core) 안에서 Spring의 `@Scheduled(cron = "0 30 0 * * *")`로 `WorkerSyncScheduler.syncAllSites()`를 호출했습니다. 스케줄링 자체는 가볍지만, 처리가 HTTP 요청·API JVM과 묶여 있고 실행 이력·현장 단위 격리가 없었습니다. 이를 해결하기 위해 별도 `dndn-batch` 프로젝트로 분리하고 Spring Batch를 도입했습니다.

<div class="port-decision">
Spring Batch는 <code>BATCH_JOB_EXECUTION</code>·<code>BATCH_STEP_EXECUTION</code> 메타 테이블에 Job·Step 실행 이력을 자동 기록하고, chunk 단위 트랜잭션·파티셔닝 병렬 처리·실패 격리를 프레임워크 수준에서 제공합니다. K8s CronJob이 독립 Pod에서 Job을 기동하므로, 배치 부하와 API 응답성이 분리됩니다.
</div>

**@Scheduled → Spring Batch 전환 과정**

| 단계 | 구조 | 한계 |
|------|------|------|
| ① 단일 Chunk Step | `workerSyncStep` · chunk(1) = 현장 1개 = 트랜잭션 1개 | 사전 검증·사후 알림 삽입 지점 없음 |
| ② Tasklet 추가 | `validationStep` + Chunk + `highRiskNotificationStep` | 검증·고위험 알림을 1회성 Tasklet으로 분리 |
| ③ Multi-Thread Step | `TaskExecutor` 4스레드 · synchronized Reader | 단일 `StepExecution` 공유 → Reader 락 경합, null read 빈 chunk 커밋 |
| ④ Partitioning | `SiteCodePartitioner` + SlaveStep × 현장 수 | 현장별 독립 `StepExecution`, 실패 격리, Reader 경합 제거 |
| ⑤ syncChunk 통합 | chunk(50) · Worker 50명 = 1청크 = 1 `@Transactional` | upsert·출결·서류·피로도를 동일 트랜잭션으로 묶어 상태 불일치 방지 |
| ⑥ rosterCleanupStep | 병렬 Step 전 직렬 DELETE | `worker_document` gap lock 데드락 구조적 해소 |
| ⑦ 스테이징 7 Step | 수집 → 검증 → 운영 반영 3단계 분리 | API 수집 실패 시 운영 테이블 보존 |

**Tasklet vs Chunk 선택 기준**

| 기준 | Tasklet | Chunk-Oriented |
|------|---------|---------------|
| 처리 단위 | `execute()` 1회 실행 | Reader → Processor → Writer 반복 |
| 적합한 경우 | 검증, DDL, 집계, 알림 등 1회성 작업 | 근로자·현장 등 대량 데이터 반복 처리 |
| 재시도 단위 | Step 전체 | chunk 단위 |
| 트랜잭션 | Step 트랜잭션 1개 | chunk마다 커밋 |

검증·스테이징 DDL·운영 테이블 사전 삭제·고위험 알림은 반복 단위가 없으므로 Tasklet을, HR 데이터 수집·운영 반영은 chunk(50) Chunk Step을 사용했습니다.

**Tasklet Step 역할**

- **validationStep** — 활성 현장 수 확인. 대상이 없으면 WARN 후 진행(향후 FAILED 전환 가능).
- **tempTableCreateStep / tempTableDropStep** — `tmp_worker_sync_stage` CREATE·TRUNCATE. Step 2·7이 동일 `TempTableTasklet`을 stepName으로 분기.
- **stagingValidationStep** — 스테이징 건수 0·`worker_name` NULL 검사. 실패 시 Job FAILED → Step 5~7 미실행, 운영 테이블 무변경.
- **rosterCleanupStep** — `attendance_record`·`attendance_log`(당일), `worker_document`(픽스처 제목) 직렬 DELETE. `allowStartIfComplete(true)`로 재시작 시에도 반드시 재실행.
- **highRiskNotificationStep** — 피로도 80점 이상 근로자 현장별 집계·WARN 로그(FCM/SMS 연동 확장 지점).

**Chunk Step — syncChunk() 내부 흐름**

Partitioned `workerSyncSlaveStep`은 `StagingTableReader` → `WorkerSyncItemWriter`로 스테이징 JSON을 역직렬화한 뒤, 50명 단위로 `syncChunk()`를 호출합니다.

```
syncChunk() [@Transactional]
  ├── Worker upsert (externalCode 기준)
  ├── bulkNormalizeRosterDay()
  │     ├── AttendanceRecord(PENDING) INSERT
  │     └── AttendanceLog(CLOCK_IN, 06:00) INSERT  ← streak 당일 포함
  ├── WorkerDocument INSERT (픽스처 서류)
  ├── SafetyAccident INSERT
  └── bulkRecalculateFatigue(ref=syncDate)  ← sync와 동일 트랜잭션
```

피로도 재계산을 별도 Step으로 분리하면 sync 커밋 후 피로도만 실패하는 "출결 최신 + 피로도 구식" 구간이 생깁니다. chunk 트랜잭션 안에서 함께 커밋하도록 `syncChunk()` 마지막 단계로 통합했습니다.

**Multi-Thread Step → Partitioning 전환**

<div class="port-decision">
초기 멀티스레드 구현은 4개 스레드가 단일 <code>StepExecution</code>을 공유하는 구조로, <code>synchronized</code> Reader 경합과 null read 빈 chunk 커밋(commit=10, write=5)이 발생했습니다. Partitioning은 <code>SiteCodePartitioner</code>가 현장별 <code>ExecutionContext</code>를 생성하고, 각 SlaveStep이 siteCode 1개만 담당하므로 Reader 경합이 구조적으로 사라집니다. 한 현장의 SlaveStep이 실패해도 다른 현장은 계속 진행되며, 현장별 독립 StepExecution으로 실행 이력을 추적할 수 있습니다.
</div>

**최종 Job 구조 (7개 Step)**

```
① validationStep        — Tasklet · 직렬   활성 현장 존재 여부 검증
② tempTableCreateStep   — Tasklet · 직렬   스테이징 테이블 생성 · TRUNCATE
③ dataFetchMasterStep   — Partitioned · 4스레드  외부 HR 데이터 수집 → 스테이징 적재
④ stagingValidationStep — Tasklet · 직렬   데이터 품질 검증 (0건 · NULL 시 FAILED)
⑤ rosterCleanupStep    — Tasklet · 직렬   운영 테이블 당일 데이터 사전 삭제
⑥ workerSyncMasterStep — Partitioned · 4스레드  스테이징 → 운영 테이블 반영
⑦ tempTableDropStep    — Tasklet · 직렬   스테이징 테이블 정리
```

**스테이징 테이블 도입**: 기존 4 Step 구조에서는 `rosterCleanupStep`(Step 2)이 API 수집보다 먼저 실행되어, 수집 실패 시 운영 테이블이 빈 상태로 남는 위험이 있었습니다. `tmp_worker_sync_stage`에 API 응답 JSON(`raw_json`)을 적재하고, `stagingValidationStep` 통과 후에만 운영 테이블을 건드리도록 Step 순서를 재배치했습니다. `ApiWorkerReader`는 현재 픽스처를 JSON 직렬화하지만, `initRows()`만 HTTP 호출로 교체하면 실 API 전환이 가능합니다.

**InnoDB 데드락 해결**: 4개 SlaveStep이 동시에 `worker_document` DELETE+INSERT를 수행하자 gap lock 순환 대기 데드락이 간헐 발생했습니다. `rosterCleanupStep`을 병렬 Step 이전에 배치해 단일 직렬 트랜잭션으로 DELETE를 완료하고, SlaveStep은 INSERT만 수행하도록 책임을 분리했습니다.

**운영 장애 대응 — JobExecutionAlreadyRunningException**

K8s Pod 강제 종료 시 `BATCH_JOB_EXECUTION`에 `STATUS='STARTED'·END_TIME=NULL`이 남아 다음 실행이 "이미 실행 중"으로 판단되는 문제가 있었습니다. `RunIdIncrementer`는 `run.id`만 증가시키므로, 커스텀 `SyncDateJobParametersIncrementer`로 매 실행마다 `run.id`와 `syncDate`(당일)를 갱신해 항상 새 `JobInstance`를 생성하도록 해결했습니다.

---

### 4. 성능 개선 과정 — 401초 → 7.6초

5개 현장 · 1,194명 기준 실측 로그

<div class="port-perf-prog">
  <div class="port-perf-prog-item">
    <div class="pppi-label">Batch 전환 직후</div>
    <div class="pppi-mtt">401초</div>
    <div class="pppi-tps">직렬 Chunk · 단건 쿼리</div>
  </div>
  <div class="pppi-arrow">→</div>
  <div class="port-perf-prog-item">
    <div class="pppi-label">벌크 쿼리 전환</div>
    <div class="pppi-mtt">188초</div>
    <div class="pppi-tps">@Modifying 벌크 UPDATE/INSERT</div>
    <div class="pppi-delta">▼ 53%</div>
  </div>
  <div class="pppi-arrow">→</div>
  <div class="port-perf-prog-item">
    <div class="pppi-label">Partitioning 병렬화</div>
    <div class="pppi-mtt">25.6초</div>
    <div class="pppi-tps">현장별 4스레드 SlaveStep</div>
    <div class="pppi-delta">▼ 86%</div>
  </div>
  <div class="pppi-arrow">→</div>
  <div class="port-perf-prog-item is-final">
    <div class="pppi-label">스테이징 + chunk(50)</div>
    <div class="pppi-mtt">7.6초</div>
    <div class="pppi-tps">7 Steps · 스테이징 검증 구조</div>
    <div class="pppi-delta">▼ 70%</div>
  </div>
</div>

**핵심 병목 3가지 제거**

<div class="port-opt-steps">
  <div class="port-opt-step">
    <div class="port-opt-step-hd">
      <span class="port-opt-step-num">①</span>
      <span class="port-opt-step-title">flush() O(n²) 제거</span>
    </div>
    <div class="port-opt-step-body">
      200명 루프 내에서 근로자마다 JPA <code>flush()</code>가 3회 호출됐습니다. <code>flush()</code>는 영속성 컨텍스트에 올라온 모든 엔티티를 더티체킹하므로, 루프가 진행될수록 검사 대상이 증가해 비용이 O(n²)로 급증했습니다. <code>@Modifying + @Query</code> 벌크 쿼리로 영속성 컨텍스트를 우회해 SQL을 직접 실행하도록 전환했습니다. 출결 정규화 DB 호출이 <strong>1,000회 이상 → 4회</strong>로 줄었습니다.
    </div>
  </div>
  <div class="port-opt-step">
    <div class="port-opt-step-hd">
      <span class="port-opt-step-num">②</span>
      <span class="port-opt-step-title">N+1 피로도 DB 호출 제거</span>
    </div>
    <div class="port-opt-step-body">
      근로자 1명당 사고이력·출결로그·업데이트 3회 쿼리, 411명 기준 1,233회 DB 호출이 발생했습니다. <code>SELECT ... WHERE worker_idx IN (:ids)</code> IN 쿼리 2개로 일괄 조회한 뒤 메모리 내에서 그룹화해 피로도를 계산하고, <code>saveAll()</code>로 batch_size:50 배치 업데이트했습니다. DB 호출이 <strong>1,233회 → 3회 고정</strong>으로 줄었습니다.
    </div>
  </div>
  <div class="port-opt-step">
    <div class="port-opt-step-hd">
      <span class="port-opt-step-num">③</span>
      <span class="port-opt-step-title">Partitioning 병렬화</span>
    </div>
    <div class="port-opt-step-body">
      5개 현장이 4스레드에 분산되어 병렬 처리됩니다. 순차 처리 예상 시간 105초에서 실측 43초(약 2.5배 단축)가 됐고, 이후 스테이징 도입과 chunk(50) 최적화가 맞물려 최종 <strong>7.6초</strong>에 도달했습니다.
    </div>
  </div>
</div>

**Step별 실측 소요 시간**

<div class="port-perf-wrap">
  <table>
    <thead>
      <tr>
        <th>Step</th>
        <th>소요 시간</th>
      </tr>
    </thead>
    <tbody>
      <tr>
        <td>validationStep</td>
        <td>0.115초</td>
      </tr>
      <tr>
        <td>tempTableCreateStep</td>
        <td>0.097초</td>
      </tr>
      <tr>
        <td>dataFetchMasterStep (4스레드 병렬)</td>
        <td>1.136초</td>
      </tr>
      <tr>
        <td>stagingValidationStep</td>
        <td>0.027초</td>
      </tr>
      <tr>
        <td>rosterCleanupStep</td>
        <td>0.564초</td>
      </tr>
      <tr>
        <td><strong>workerSyncMasterStep (4스레드 병렬)</strong></td>
        <td><strong>5.407초</strong></td>
      </tr>
      <tr>
        <td>tempTableDropStep</td>
        <td>0.068초</td>
      </tr>
      <tr>
        <td><strong>Job 총 실행</strong></td>
        <td><strong>7.647초</strong></td>
      </tr>
    </tbody>
  </table>
</div>

