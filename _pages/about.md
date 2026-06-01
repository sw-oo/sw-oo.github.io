---
title: "About"
permalink: /about/
layout: single
comments: false
author_profile: true
---

문제의 원인을 끝까지 파고들어 근본적인 해결책을 만들어내는 개발자입니다.

`kokodd1234@gmail.com` &nbsp;·&nbsp; `010-8519-7066` &nbsp;·&nbsp; [GitHub](https://github.com/sw-oo)

---

## 자격증 & 어학

<div class="port-cert-grid">
  <div class="port-cert-card">
    <span class="cc-name">정보처리기사</span>
    <span class="cc-sub">한국산업인력공단</span>
    <span class="cc-date">2025.08</span>
  </div>
  <div class="port-cert-card">
    <span class="cc-name">한국사능력검정시험 1급</span>
    <span class="cc-sub">국사편찬위원회</span>
    <span class="cc-date">2025.02</span>
  </div>
  <div class="port-cert-card">
    <span class="cc-name">ADsP</span>
    <span class="cc-sub">데이터분석준전문가</span>
    <span class="cc-date">2025.06</span>
  </div>
  <div class="port-cert-card">
    <span class="cc-name">SQLD</span>
    <span class="cc-sub">SQL개발자</span>
    <span class="cc-date">2025.06</span>
  </div>
  <div class="port-cert-card">
    <span class="cc-name">PCCP LV2</span>
    <span class="cc-sub">프로그래머스</span>
    <span class="cc-date">2026.03</span>
  </div>
  <div class="port-cert-card">
    <span class="cc-name">TOEIC Speaking IH</span>
    <span class="cc-sub">150점 · ETS</span>
    <span class="cc-date">2025.10</span>
  </div>
</div>

---

## Skills

<div class="port-skill-section">

  <div class="port-skill-group">
    <div class="port-skill-label">Backend</div>
    <div class="port-skill-grid">
      <div class="port-skill-item">
        <div class="si-name">Spring Batch</div>
        <div class="si-desc">Tasklet + Chunk 혼합 설계, Partitioning 4스레드 병렬 처리로 401초 → 7.6초 달성</div>
      </div>
      <div class="port-skill-item">
        <div class="si-name">JPA · Bulk Query</div>
        <div class="si-desc">Fetch Join, @BatchSize로 N+1 제거 / @Modifying 벌크 UPDATE·INSERT로 flush() O(n²) 해소</div>
      </div>
      <div class="port-skill-item">
        <div class="si-name">JWT · Spring Security</div>
        <div class="si-desc">MSA stateless 인증 필터 구현, RBAC + ABAC 혼합 인가로 리소스 소유권 검증</div>
      </div>
      <div class="port-skill-item">
        <div class="si-name">Spring Boot · Java</div>
        <div class="si-desc">REST API 설계, MSA 멀티모듈 구조, 도메인 레이어 분리</div>
      </div>
    </div>
  </div>

  <div class="port-skill-group">
    <div class="port-skill-label">Real-time &amp; Notification</div>
    <div class="port-skill-grid">
      <div class="port-skill-item">
        <div class="si-name">WebSocket · STOMP</div>
        <div class="si-desc">채팅방 진입 시점 연결 제한, pub/sub 메시지 라우팅으로 채팅방 단위 구독 관리</div>
      </div>
      <div class="port-skill-item">
        <div class="si-name">SSE</div>
        <div class="si-desc">브라우저 포그라운드 실시간 알림 스트리밍, Web Push와 이원화 구성</div>
      </div>
      <div class="port-skill-item">
        <div class="si-name">Web Push API · Service Worker</div>
        <div class="si-desc">VAPID 구독 관리, 브라우저 종료 상태에서도 OS 레벨 알림 전달</div>
      </div>
    </div>
  </div>

  <div class="port-skill-group">
    <div class="port-skill-label">Infrastructure &amp; DevOps</div>
    <div class="port-skill-grid">
      <div class="port-skill-item">
        <div class="si-name">Kubernetes · K8s CronJob</div>
        <div class="si-desc">배치 독립 Pod 실행, ConcurrencyPolicy Forbid · backoffLimit으로 중복 방지 및 자동 재시도</div>
      </div>
      <div class="port-skill-item">
        <div class="si-name">Docker · Jenkins</div>
        <div class="si-desc">컨테이너 이미지 빌드·배포, Blue-Green / Canary 무중단 배포 파이프라인</div>
      </div>
    </div>
  </div>

  <div class="port-skill-group">
    <div class="port-skill-label">Monitoring &amp; Testing</div>
    <div class="port-skill-grid">
      <div class="port-skill-item">
        <div class="si-name">Jaeger</div>
        <div class="si-desc">분산 트레이싱으로 쿼리 스팬 병목 탐지, 개선 전후 응답 시간 정량 비교</div>
      </div>
      <div class="port-skill-item">
        <div class="si-name">Prometheus · Grafana</div>
        <div class="si-desc">시스템 메트릭 수집 및 대시보드 모니터링</div>
      </div>
      <div class="port-skill-item">
        <div class="si-name">nGrinder</div>
        <div class="si-desc">240 VU 동시 접속 부하 테스트, 에러율 0% 검증</div>
      </div>
    </div>
  </div>

</div>

---

## Projects

<div class="port-proj-cards">
  <a class="port-proj-card" href="/project/poticard/">
    <div class="ppc-type">B2C Platform · 2025.12 — 2026.04</div>
    <div class="ppc-title">Poticard</div>
    <div class="ppc-period">AI 기반 디지털 명함 & 채용 플랫폼</div>
    <div class="ppc-desc">경력 데이터 AI 분석 → 디지털 명함 생성 → 구직자·채용 담당자 간 실시간 채팅·지원 플랫폼. 실시간 채팅, 알림 시스템, 성능 최적화 담당.</div>
    <div class="ppc-tags">
      <span class="ppc-tag">WebSocket · STOMP</span>
      <span class="ppc-tag">Web Push API</span>
      <span class="ppc-tag">SSE</span>
      <span class="ppc-tag">JPA 최적화</span>
      <span class="ppc-tag">nGrinder</span>
    </div>
  </a>
  <a class="port-proj-card" href="/project/dndn/">
    <div class="ppc-type">ERP System · 2026.04 — 2026.06</div>
    <div class="ppc-title">DnDn</div>
    <div class="ppc-period">건설 현장관리 ERP 시스템</div>
    <div class="ppc-desc">인력 배치·출결 관리·피로도 기반 구역 배정 ERP. Multimodule + MSA 구조. 인증/인가, 인력 동기화 배치, 모바일 API 연동 담당.</div>
    <div class="ppc-tags">
      <span class="ppc-tag">JWT · RBAC+ABAC</span>
      <span class="ppc-tag">Spring Batch</span>
      <span class="ppc-tag">K8s CronJob</span>
      <span class="ppc-tag">Partitioning</span>
      <span class="ppc-tag">Ionic Vue</span>
    </div>
  </a>
</div>
