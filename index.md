---
layout: single
title: false
author_profile: true
comments: false
toc: true
toc_label: "목차"
toc_sticky: true
masthead_title: "swoo's Portfolio"
---

<h2 id="introduce">Introduce</h2>

문제의 원인을 끝까지 파고들어 근본적인 해결책을 만들어내는 개발자 최승우입니다.

---

<h2 id="cert">Certification / Language</h2>

<div class="port-tl-wrap port-cert-lang-wrap">

  <div class="port-cert-col">
    <div class="port-tl-section">Certification</div>
    <div class="port-timeline">
      <div class="port-tl-item">
        <div class="port-tl-date">2026.03</div>
        <div class="port-tl-title">PCCP LV2</div>
        <div class="port-tl-sub">프로그래머스</div>
      </div>
      <div class="port-tl-item">
        <div class="port-tl-date">2025.08</div>
        <div class="port-tl-title">정보처리기사</div>
        <div class="port-tl-sub">한국산업인력공단</div>
      </div>
      <div class="port-tl-item">
        <div class="port-tl-date">2025.06</div>
        <div class="port-tl-title">ADsP · SQLD</div>
        <div class="port-tl-sub">데이터분석준전문가 · SQL개발자</div>
      </div>
      <div class="port-tl-item">
        <div class="port-tl-date">2025.02</div>
        <div class="port-tl-title">한국사능력검정시험 1급</div>
        <div class="port-tl-sub">국사편찬위원회</div>
      </div>
    </div>
  </div>

  <div class="port-cert-col">
    <div class="port-tl-section">Language</div>
    <div class="port-timeline">
      <div class="port-tl-item">
        <div class="port-tl-date">2025.10</div>
        <div class="port-tl-title">TOEIC Speaking IH</div>
        <div class="port-tl-sub">150점 · ETS</div>
      </div>
    </div>
  </div>

</div>

---

<h2 id="edu">Graduate / Education</h2>

<div class="port-tl-wrap port-cert-lang-wrap">

  <div class="port-cert-col">
    <div class="port-tl-section">Graduate</div>
    <div class="port-timeline">
      <div class="port-tl-item">
        <div class="port-tl-date">2018.03 ~ 2025.08</div>
        <div class="port-tl-title">한신대학교 졸업</div>
        <div class="port-tl-sub">컴퓨터공학부</div>
      </div>
      <div class="port-tl-item">
        <div class="port-tl-date">2015.03 ~ 2017.02</div>
        <div class="port-tl-title">양천고등학교 졸업</div>
      </div>
    </div>
  </div>

  <div class="port-cert-col">
    <div class="port-tl-section">Education</div>
    <div class="port-timeline">
      <div class="port-tl-item">
        <div class="port-tl-date">2025.12 ~ 2026.06</div>
        <div class="port-tl-title">한화 Beyond SW Camp</div>
        <div class="port-tl-sub">24기 수료 · FE·BE·Devops</div>
      </div>
    </div>
  </div>

</div>

---

<h2 id="skills">Skills</h2>

<div class="port-tabs">

  <div class="port-tab-nav">
    <button class="port-tab-btn active" data-tab="tab-backend">Backend</button>
    <button class="port-tab-btn" data-tab="tab-realtime">Real-time</button>
    <button class="port-tab-btn" data-tab="tab-infra">DevOps</button>
    <button class="port-tab-btn" data-tab="tab-monitoring">Monitoring</button>
  </div>

  <div id="tab-backend" class="port-tab-panel active">
    <div class="port-skill-grid">
      <div class="port-skill-item">
        <div class="si-name">Spring Batch</div>
        <div class="si-desc">Tasklet + Chunk 혼합 설계, Partitioning 4스레드 병렬 처리로 401초 → 7.6초 달성</div>
      </div>
      <div class="port-skill-item">
        <div class="si-name">JPA</div>
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
      <div class="port-skill-item">
        <div class="si-name">Swagger</div>
        <div class="si-desc">REST API 명세 자동화, 팀 간 인터페이스 계약 문서화</div>
      </div>
    </div>
  </div>

  <div id="tab-realtime" class="port-tab-panel">
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
        <div class="si-name">Web Push API</div>
        <div class="si-desc">VAPID 구독 관리, 브라우저 종료 상태에서도 OS 레벨 알림 전달</div>
      </div>
    </div>
  </div>

  <div id="tab-infra" class="port-tab-panel">
    <div class="port-skill-grid">
      <div class="port-skill-item">
        <div class="si-name">Docker · Kubernetes</div>
        <div class="si-desc">컨테이너 이미지 빌드·배포, CronJob 독립 Pod 실행, ConcurrencyPolicy Forbid · backoffLimit으로 중복 방지 및 자동 재시도</div>
      </div>
      <div class="port-skill-item">
        <div class="si-name">Istio</div>
        <div class="si-desc">서비스 메시로 마이크로서비스 간 트래픽 관리 및 통신 제어</div>
      </div>
      <div class="port-skill-item">
        <div class="si-name">Jenkins</div>
        <div class="si-desc">CI/CD 파이프라인 구성, Blue-Green / Canary 무중단 배포</div>
      </div>
    </div>
  </div>

  <div id="tab-monitoring" class="port-tab-panel">
    <div class="port-skill-grid">
      <div class="port-skill-item">
        <div class="si-name">Jaeger</div>
        <div class="si-desc">분산 트레이싱으로 쿼리 스팬 병목 탐지, 개선 전후 응답 시간 정량 비교</div>
      </div>
      <div class="port-skill-item">
        <div class="si-name">Prometheus</div>
        <div class="si-desc">시스템 메트릭 수집 및 모니터링</div>
      </div>
      <div class="port-skill-item">
        <div class="si-name">Grafana</div>
        <div class="si-desc">대시보드 기반 시각화 및 알림 설정</div>
      </div>
      <div class="port-skill-item">
        <div class="si-name">nGrinder</div>
        <div class="si-desc">240 VU 동시 접속 부하 테스트, 에러율 0% 검증</div>
      </div>
    </div>
  </div>

</div>

---

<h2 id="projects">Projects</h2>

<div class="port-proj-cards">
  <a class="port-proj-card" href="/project/poticard/">
    <div class="ppc-type">B2C Platform · 2025.12 — 2026.04</div>
    <div class="ppc-title">Poticard</div>
    <div class="ppc-period">AI 기반 디지털 명함 &amp; 채용 플랫폼</div>
    <div class="ppc-desc">경력 데이터 AI 분석 → 디지털 명함 생성 → 구직자·채용 담당자 간 실시간 채팅·지원 플랫폼. 실시간 채팅, 알림 시스템, 성능 최적화 담당.</div>
    <div class="ppc-tags">
      <span class="ppc-tag">B2C</span>
      <span class="ppc-tag">Websocket·STOMP</span>
      <span class="ppc-tag">Web Push API</span>
      <span class="ppc-tag">SSE</span>
    </div>
  </a>
  <a class="port-proj-card" href="/project/dndn/">
    <div class="ppc-type">ERP System · 2026.04 — 2026.06</div>
    <div class="ppc-title">DnDn</div>
    <div class="ppc-period">건설 현장관리 ERP 시스템</div>
    <div class="ppc-desc">인력 배치·출결 관리·피로도 기반 구역 배정 ERP. Multimodule + MSA 구조. 인증/인가, 인력 동기화 배치, 모바일 API 연동 담당.</div>
    <div class="ppc-tags">
      <span class="ppc-tag">B2B</span>
      <span class="ppc-tag">MSA</span>
      <span class="ppc-tag">Spring Batch</span>
      <span class="ppc-tag">Application</span>
      <span class="ppc-tag">Kubernetes</span>
    </div>
  </a>
</div>

---

<h2 id="contact">Contact</h2>

<div class="port-contact-wrap">
  <a class="port-contact-item" href="mailto:kokodd1234@gmail.com">
    <span class="port-contact-icon">✉</span>
    <span>kokodd1234@gmail.com</span>
  </a>
  <a class="port-contact-item" href="tel:01085197066">
    <span class="port-contact-icon">📱</span>
    <span>010-8519-7066</span>
  </a>
</div>

<script>
(function () {
  function initTabs() {
    document.querySelectorAll('.port-tabs').forEach(function (wrap) {
      wrap.querySelectorAll('.port-tab-btn').forEach(function (btn) {
        btn.addEventListener('click', function () {
          wrap.querySelectorAll('.port-tab-btn').forEach(function (b) { b.classList.remove('active'); });
          wrap.querySelectorAll('.port-tab-panel').forEach(function (p) { p.classList.remove('active'); });
          btn.classList.add('active');
          var panel = wrap.querySelector('#' + btn.dataset.tab);
          if (panel) panel.classList.add('active');
        });
      });
    });
  }
  if (document.readyState === 'loading') {
    document.addEventListener('DOMContentLoaded', initTabs);
  } else {
    initTabs();
  }
})();
</script>
