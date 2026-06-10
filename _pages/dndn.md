---
title: "DnDn — 건설 현장관리 ERP 시스템"
permalink: /project/dndn/
layout: single
author_profile: true
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
    <div class="port-meta-label">링크</div>
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

- **인증/인가** — JWT 인증 필터, RBAC+ABAC 혼합 인가 구현 (dndn-core / auth 패키지)
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

### 1. JWT 인증 방식 채택

MSA 구조에서 세션 기반 인증을 적용하려면 두 가지 중 하나를 선택해야 합니다. 공유 세션 저장소(Redis 등)에 모든 마이크로서비스가 접근하거나, Sticky Session으로 특정 인스턴스에 요청을 고정하는 방식입니다. 공유 저장소 방식은 세션 관리 의존성이 서비스 간에 생기므로, 인증 저장소 장애가 전체 서비스 장애로 전파될 위험이 있습니다. Sticky Session은 수평 확장 시 특정 인스턴스에 부하가 집중되어 MSA의 독립 확장 이점을 희석시킵니다.

<div class="port-decision">
JWT는 클레임(사용자 ID, 역할 등)을 토큰 자체에 포함하므로, 각 서비스가 공개 키로 서명만 검증하면 인증 상태를 독립적으로 확인할 수 있습니다. 별도 인증 서버 호출 없이 stateless하게 동작하므로 서비스 간 결합도가 낮고, 수평 확장 시 세션 동기화 문제가 발생하지 않습니다. dndn-core 내 인증 필터가 토큰을 검증하고 SecurityContext에 사용자 정보를 주입하면, 이후 모든 요청 처리가 외부 의존 없이 진행됩니다.
</div>

---

### 2. RBAC + ABAC 혼합 인가 설계

역할 기반 접근 제어(RBAC)만으로 설계를 시작했을 때, "근로자는 자신이 배치된 현장의 출결·서류 데이터만 열람 가능하다"는 요구사항을 표현하기 어려웠습니다. 모든 근로자에게 동일한 `ROLE_WORKER`를 부여하면 타 현장 데이터에도 접근 가능해지는 허점이 생기고, 이를 RBAC 안에서 해결하려면 현장마다 별도 역할을 생성해야 해 역할이 폭발적으로 증가합니다.

순수 ABAC(속성 기반)로만 구성하면, 기본적인 역할 구분(관리자/고용주/근로자)마저 모든 요청마다 속성 평가 연산이 필요해 불필요한 오버헤드가 발생합니다.

<div class="port-decision">
<strong>혼합 전략</strong>: RBAC로 역할 수준의 1차 접근 통제를 처리합니다(현장 생성은 관리자·고용주만 가능, 근로자는 조회만 가능). 역할 체크를 통과한 요청에 한해서만 ABAC로 리소스 소유권을 검증합니다(요청 파라미터의 현장 ID가 자신이 배치된 현장 ID와 일치하는지 확인). 역할 체크가 실패하면 속성 평가 없이 즉시 거부되므로, 연산 비용이 높은 ABAC 평가를 최소한의 요청에만 적용합니다.
</div>

---

### 3. K8s CronJob 트리거 전환

초기 구현은 `@Scheduled(cron = "0 30 0 * * *")`로 매일 00:30에 외부 HR 시스템의 인력 정보를 DB에 동기화했습니다. 이 방식은 세 가지 문제를 만들었습니다.

**HTTP 요청 라이프사이클에 처리가 묶임**: 프론트에서 수동 트리거를 시도할 경우 처리 시간이 길어져 HTTP timeout이 발생했습니다. 클라이언트 입장에서는 실패처럼 보이지만 백엔드는 계속 실행 중인 불투명한 상태가 연출됐습니다.

**API 서버와 JVM 공유**: 배치 처리가 API 서버와 같은 JVM에서 동작하므로, 야간 대량 처리 중 API 응답 지연이 유발됐습니다. 배치 부하가 증가할수록 서비스 전체의 응답성에 영향을 미쳤습니다.

**실행 이력 부재**: 성공·실패 여부를 로그로만 파악해야 했고, 특정 현장 처리 중 예외가 발생하면 이후 모든 현장이 중단되는 격리 부재 문제도 있었습니다.

<div class="port-decision">
K8s CronJob으로 전환하면서 배치를 <strong>독립 Pod</strong>에서 실행하도록 분리했습니다. 배치 실패가 API 서버에 영향을 주지 않고, <code>concurrencyPolicy: Forbid</code>로 이전 실행이 남아있으면 다음 실행을 건너뛰며, <code>backoffLimit: 2</code>로 실패 시 자동 재시도합니다.<br><br>
K8s CronJob에서 curl로 백엔드 API를 호출하는 방식(Architecture C)도 검토했습니다. 이는 별도 배치 프로젝트 없이 단순하지만, 백엔드 JVM을 공유하므로 부하 분리가 되지 않고 Spring Batch 메타 테이블 기반 실행 이력도 남지 않습니다. <strong>부하 분리와 실행 이력이 운영 핵심 요건</strong>이었기 때문에 독립 배치 프로젝트(Architecture A)를 채택했습니다.
</div>

---

### 4. Spring Batch Tasklet + Chunk 혼합 방식 적용

Quartz Scheduler와 비교했을 때, Quartz는 Job 스케줄링에 특화되어 있지만 대량 데이터를 chunk 단위 트랜잭션으로 처리하는 기능, step 수준 실행 이력, 파티셔닝 병렬 처리가 없습니다. Spring Batch는 `BATCH_JOB_EXECUTION` 메타 테이블에 Job·Step 실행 이력이 자동 기록되어 운영 가시성이 확보됩니다.

**Tasklet과 Chunk의 혼합 근거**

1회성으로 실행되는 작업(검증·임시 테이블 DDL·정리 등)에는 Tasklet을 사용했습니다. 반복 처리 단위가 없는 작업에 Chunk 구조를 적용하면 불필요한 추상화가 됩니다. 대량 데이터를 반복 처리하는 구간에는 Chunk를 사용했습니다. chunk(50) 단위로 트랜잭션이 커밋되므로, 중간에 예외가 발생해도 이미 커밋된 chunk의 데이터는 보존됩니다.

**Multi-Thread Step → Partitioning 전환**

<div class="port-decision">
초기 멀티스레드 구현은 4개 스레드가 단일 <code>StepExecution</code>을 공유하는 구조로, synchronized Reader 경합이 발생했습니다. Partitioning은 현장별로 독립 SlaveStep을 생성하므로 Reader 경합이 구조적으로 사라집니다. 한 현장의 SlaveStep이 실패해도 다른 현장의 처리는 계속 진행되며, 현장별 독립 StepExecution이 생성되어 실행 이력도 현장 단위로 추적됩니다.
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

**스테이징 테이블 도입**: 기존 구조에서 운영 테이블 삭제(rosterCleanupStep)가 API 수집보다 먼저 실행되면, 수집 실패 시 운영 테이블이 빈 상태로 남아 당일 출근 데이터가 손실되는 위험이 있었습니다. `tmp_worker_sync_stage` 스테이징 테이블을 버퍼로 두고 "수집 → 검증 → 운영 반영"을 3단계로 분리했습니다. `stagingValidationStep`이 실패하면 Step 5~7이 실행되지 않으므로 운영 테이블이 항상 보존됩니다.

**InnoDB 데드락 해결**: 초기 구조에서 4개 SlaveStep이 동시에 동일 테이블에 DELETE+INSERT를 실행하자 gap lock 순환 대기 데드락이 발생했습니다. `rosterCleanupStep`을 병렬 Step 이전에 배치해 단일 직렬 트랜잭션으로 삭제를 완료하고, 이후 SlaveStep은 INSERT만 수행하도록 분리했습니다. gap lock은 DELETE 구간에서 발생하므로, INSERT만 남은 병렬 구간에서 충돌이 구조적으로 사라졌습니다.

---

### 5. 성능 개선 과정 — 401초 → 7.6초

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

<div class="port-decision">
  <strong>추가 장애 해결 — JobExecutionAlreadyRunningException</strong><br>
  K8s Pod 강제 종료 시 <code>BATCH_JOB_EXECUTION</code> 테이블에 <code>STATUS='STARTED'·END_TIME=NULL</code>이 남아 다음 실행에서 "이미 실행 중"으로 판단하는 문제가 있었습니다. <code>RunIdIncrementer</code>에서 커스텀 <code>SyncDateJobParametersIncrementer</code>로 교체해 매 실행마다 <code>run.id</code>를 증가시키고 <code>syncDate</code>를 당일 날짜로 갱신하여 항상 새로운 <code>JobInstance</code>를 생성하도록 해결했습니다.
</div>
