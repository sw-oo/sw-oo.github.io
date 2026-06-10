---
title: "Poticard — AI 기반 디지털 명함 & 채용 플랫폼"
permalink: /project/poticard/
layout: single
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
    <div class="port-meta-value">2025.12 — 2026.04</div>
  </div>
  <div class="port-meta-item">
    <div class="port-meta-label">팀 구성</div>
    <div class="port-meta-value">Fullstack Developer 4인</div>
  </div>
  <div class="port-meta-item">
    <div class="port-meta-label">유형</div>
    <div class="port-meta-value">B2C 웹 서비스</div>
  </div>
  <div class="port-meta-item">
    <div class="port-meta-label">링크</div>
    <div class="port-meta-value">
      <a href="https://github.com/beyond-sw-camp/be24-4th-DevOops-Poticard" target="_blank">GitHub</a> &nbsp;·&nbsp;
      <a href="https://www.poticard.kro.kr" target="_blank">Service</a>
    </div>
  </div>
</div>

## 프로젝트 개요

- 사용자의 경력·프로젝트 데이터를 AI가 분석해 핵심 키워드를 추출하고, 디지털 명함과 포트폴리오를 자동 생성하는 B2C 채용 플랫폼
- QR 코드·고유 URL로 명함을 공유하고, 구직자와 채용 담당자가 1:1·그룹 채팅 및 화상 면접으로 실시간 연결
- Kubernetes 환경에서 Canary·Blue-Green 무중단 배포

## 담당 기능

- **Chatting** — WebSocket STOMP 기반 1:1·그룹 텍스트 채팅 및 VideoChat 구현
- **Notification** — SSE(포그라운드) + Web Push API(백그라운드·오프라인) 이원화 알림 구성
- **채팅·알림 관련 성능 최적화** — 채팅방 목록 N+1 제거(TPS +403%), 메시지 이력 Slice 페이징 전환(TPS +142%)

## 기술 스택

<div class="port-tech-groups">
  <div class="port-tech-group">
    <div class="ptg-label">Backend</div>
    <div class="ptg-badges">
      <span class="port-badge pb-b">Java</span>
      <span class="port-badge pb-b">Spring Boot</span>
      <span class="port-badge pb-b">Spring Security</span>
      <span class="port-badge pb-b">Swagger</span>
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
    <div class="ptg-label">Real-time</div>
    <div class="ptg-badges">
      <span class="port-badge pb-p">WebSocket · STOMP</span>
      <span class="port-badge pb-p">SSE</span>
      <span class="port-badge pb-p">Web Push API</span>
    </div>
  </div>
  <div class="port-tech-group">
    <div class="ptg-label">Infra</div>
    <div class="ptg-badges">
      <span class="port-badge pb-o">Kubernetes</span>
      <span class="port-badge pb-o">Jenkins</span>
      <span class="port-badge pb-o">Docker</span>
      <span class="port-badge pb-o">AWS S3</span>
    </div>
  </div>
  <div class="port-tech-group">
    <div class="ptg-label">Monitoring</div>
    <div class="ptg-badges">
      <span class="port-badge pb-s">Prometheus</span>
      <span class="port-badge pb-s">Grafana</span>
      <span class="port-badge pb-s">Jaeger</span>
      <span class="port-badge pb-s">nGrinder</span>
    </div>
  </div>
</div>

---

## 기술 의사결정

### 1. WebSocket STOMP 채택

실시간 채팅 구현 방식을 검토했을 때 가장 먼저 고려한 것은 HTTP 폴링이었습니다. 클라이언트가 주기적으로 서버를 조회하는 방식은 인프라 변경 없이 적용할 수 있지만, 구직자·채용 담당자가 동시에 수백 명 접속하는 상황에서 요청 수가 사용자 수와 비례해 선형으로 증가합니다. 실시간성을 높이려면 폴링 간격을 줄여야 하고, 그럴수록 대부분의 요청이 빈 응답을 반환하는 낭비가 커집니다.

Long-polling은 서버가 데이터 준비 시까지 응답을 지연하므로 폴링보다 불필요한 요청 수는 줄지만, 메시지가 빈번한 채팅 시나리오에서는 TCP 연결 수립·해제가 반복되어 비슷한 수준의 오버헤드가 남습니다.

<div class="port-decision">
WebSocket은 초기 HTTP Upgrade 핸드셰이크 이후 단일 TCP 연결을 유지하므로, 연결당 비용이 최초 1회에 고정됩니다. 여기에 STOMP 서브프로토콜을 선택한 이유는 <strong>메시지 라우팅 책임을 브로커에 위임</strong>하기 위해서입니다. 순수 WebSocket으로 특정 채팅방에만 메시지를 전달하려면 서버가 연결 목록을 직접 순회하며 필터링해야 합니다. STOMP는 <code>/topic/chat.{roomId}</code> 같은 destination 기반 pub/sub 구조를 제공하므로, 채팅방 단위 구독 관리와 메시지 라우팅이 브로커 수준에서 처리됩니다.
</div>

**핵심 결정 — 채팅방 진입 시점 연결 제한**

WebSocket은 상태를 유지하는 지속 연결입니다. 로그인과 동시에 연결하면 채팅방에 들어오지 않은 사용자까지 서버에 커넥션을 점유하게 됩니다. 동시 접속자가 늘수록 실제 채팅에 사용되지 않는 연결이 쌓이므로, **채팅방 진입 시 연결하고 퇴장 시 해제**하는 방식으로 서버 자원의 효율성을 확보했습니다. 연결 시점에 STOMP CONNECT 헤더의 JWT를 검증하여 인증되지 않은 채널 구독도 차단합니다.

---

### 2. Web Push API + SSE 이원화 구성

WebSocket을 알림에도 재사용하는 방안을 검토했습니다. 그러나 알림은 서버에서 클라이언트로 향하는 단방향 데이터 흐름이므로, 양방향 지속 연결을 강제하는 WebSocket은 알림 목적에서 불필요한 오버헤드를 만들었습니다.

SSE는 HTTP 위에서 동작하는 서버→클라이언트 단방향 스트림으로, 알림처럼 서버가 이벤트를 푸시하는 패턴에 구조적으로 부합합니다. 그러나 **브라우저 탭이 닫히거나 백그라운드로 전환되면 SSE 연결이 끊기며 알림이 유실**됩니다. 채팅 서비스에서 오프라인 상태의 메시지 알림을 놓치는 것은 사용자 경험에 직결되는 문제였습니다.

<div class="port-decision">
Web Push API는 VAPID(Voluntary Application Server Identification) 방식으로 구독을 관리하며, Service Worker가 백그라운드에서 Push 이벤트를 수신합니다. 브라우저가 완전히 닫힌 상태에서도 OS 레벨 알림이 전달되므로, SSE가 커버하지 못하는 오프라인·백그라운드 구간을 채웁니다.<br><br>
<strong>이원화 전략</strong>: 탭이 활성화된 상태에서는 SSE로 즉각 반영하고, 백그라운드·탭 미접속 상태에서는 Web Push로 전달합니다. 두 채널을 중복 운영하는 대신, 서버가 SSE 연결 여부를 추적하여 미연결 상태일 때만 Web Push를 전송하도록 분기했습니다. 사용자가 푸시 권한을 차단한 경우에는 SSE만으로 실시간성을 제공합니다.
</div>

---

### 3. 채팅방 목록 / 메시지 이력 성능 최적화

#### 채팅방 목록 조회

초기 구현에서는 채팅방 목록을 불러올 때 각 채팅방의 최신 메시지와 참여자 정보를 건별로 조회하는 N+1 구조였습니다. Jaeger 분산 트레이싱으로 쿼리 스팬을 분석한 결과, 전체 응답 시간의 대부분이 반복 쿼리에서 비롯됨을 확인했습니다.

<div class="port-opt-steps">
  <div class="port-opt-step">
    <div class="port-opt-step-hd">
      <span class="port-opt-step-num">1차</span>
      <span class="port-opt-step-title">Slice 페이징 전환</span>
    </div>
    <div class="port-opt-step-body">
      JPA의 <code>Page</code>는 전체 데이터 건수를 위한 <code>COUNT(*)</code> 쿼리를 항상 수행합니다. 채팅방 목록에서 필요한 정보는 "다음 페이지가 존재하는가"뿐이므로, <code>COUNT(*)</code> 없이 n+1건만 조회해 다음 페이지 여부를 판단하는 <code>Slice</code>로 전환했습니다.
    </div>
  </div>
  <div class="port-opt-step">
    <div class="port-opt-step-hd">
      <span class="port-opt-step-num">2차</span>
      <span class="port-opt-step-title">Fetch Join + @BatchSize 적용</span>
    </div>
    <div class="port-opt-step-body">
      N+1의 근본 원인인 연관 엔티티 개별 조회를 Fetch Join과 <code>@EntityGraph</code>로 단일 쿼리에 조인해 해결했습니다. 컬렉션 연관관계는 Fetch Join으로 페이징이 불가하므로 <code>@BatchSize</code>를 적용해 IN 쿼리로 일괄 로딩했습니다. N+1이 발생하는 지점을 구조적으로 제거했습니다.
    </div>
  </div>
</div>

<div class="port-perf-prog">
  <div class="port-perf-prog-item">
    <div class="pppi-label">초기 (N+1)</div>
    <div class="pppi-mtt">8.3s</div>
    <div class="pppi-tps">TPS 14.4</div>
  </div>
  <div class="pppi-arrow">→</div>
  <div class="port-perf-prog-item">
    <div class="pppi-label">1차 Slice</div>
    <div class="pppi-mtt">3.5s</div>
    <div class="pppi-tps">TPS 38.2</div>
    <div class="pppi-delta">▲ TPS 165%</div>
  </div>
  <div class="pppi-arrow">→</div>
  <div class="port-perf-prog-item is-final">
    <div class="pppi-label">2차 Fetch Join</div>
    <div class="pppi-mtt">2.0s</div>
    <div class="pppi-tps">TPS 58.1</div>
    <div class="pppi-delta">▲ TPS 403%</div>
  </div>
</div>

#### 메시지 이력 조회

<div class="port-opt-steps">
  <div class="port-opt-step">
    <div class="port-opt-step-hd">
      <span class="port-opt-step-num">개선</span>
      <span class="port-opt-step-title">Slice 페이징 전환</span>
    </div>
    <div class="port-opt-step-body">
      75만 건 데이터를 전량 로드하는 구조로 인해 메모리 과부하가 원인이었습니다. <code>Page</code> 기반 전체 페이지네이션에서 <code>Slice(size=20)</code>으로 전환하여 한 번에 20건씩만 조회하고 <code>COUNT(*)</code> 쿼리를 제거했습니다.
    </div>
  </div>
</div>

<div class="port-perf-prog">
  <div class="port-perf-prog-item">
    <div class="pppi-label">초기 (전량 로드)</div>
    <div class="pppi-mtt">8.1s</div>
    <div class="pppi-tps">TPS 29.2</div>
  </div>
  <div class="pppi-arrow">→</div>
  <div class="port-perf-prog-item is-final">
    <div class="pppi-label">Slice(size=20)</div>
    <div class="pppi-mtt">3.3s</div>
    <div class="pppi-tps">TPS 70.7</div>
    <div class="pppi-delta">▲ TPS 142%</div>
  </div>
</div>

<div class="port-decision">
  <strong>부하 테스트 결과</strong>: nGrinder 240 가상 사용자 동시 접속 시 에러율 0% 유지. Jaeger 트레이싱으로 특정 쿼리 스팬이 전체 응답 시간의 80% 이상을 점유하는 병목을 파악하고, 개선 전후를 트레이싱 데이터로 정량 비교했습니다.
</div>
