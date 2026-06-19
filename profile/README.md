# ClueRoom

<p align="center">
  <strong>AI suspect interrogation mystery game</strong><br>
  증거를 읽고, 용의자를 심문하고, 사건의 빈칸을 채우는 AI 추리게임 플랫폼
</p>

<p align="center">
  <a href="https://www.clueroom.xyz"><img alt="ClueRoom Web" src="https://img.shields.io/badge/Play-www.clueroom.xyz-111827?style=flat-square"></a>
  <a href="https://api.clueroom.xyz/actuator/health"><img alt="API Health" src="https://img.shields.io/badge/API-health_check-2563eb?style=flat-square"></a>
  <img alt="Java" src="https://img.shields.io/badge/Java-21-f97316?style=flat-square">
  <img alt="Spring Boot" src="https://img.shields.io/badge/Spring_Boot-4.x-16a34a?style=flat-square">
  <img alt="Flutter" src="https://img.shields.io/badge/Flutter-Android-0284c7?style=flat-square">
  <img alt="React" src="https://img.shields.io/badge/React_Web-Vite-7c3aed?style=flat-square">
</p>

---

## Case File

ClueRoom은 플레이어가 탐정이 되어 공식 사건을 조사하는 서비스입니다.

사용자는 사건 브리핑을 읽고, 현장/증거/타임라인을 확인한 뒤, AI 용의자에게 질문을 던집니다. AI는 자유롭게 추리해주는 해설자가 아니라 **백엔드가 허용한 공개 정보와 답변 정책 안에서만 말하는 NPC**입니다.

```text
목표: 챗봇과 대화하는 앱이 아니라, 증거를 조합해 추리하는 게임
핵심: AI 답변보다 사건 그래프, 증거 해금, 답변 정책, 최종 추리 채점
원칙: 범인과 정답 정보는 AI에게 직접 주지 않고 백엔드가 관리
```

---

## Investigation Loop

```mermaid
flowchart TD
    briefing[사건 브리핑] --> scene[현장 정보]
    scene --> evidence[증거 카드]
    evidence --> suspect[용의자 프로필]
    suspect --> chat[AI 심문]
    chat --> unlock[추가 증거 해금]
    unlock --> evidence
    evidence --> final[최종 추리 제출]
    final --> result[결과 / 해설]
```

플레이는 “질문을 많이 하는 것”보다 “어떤 증거를 누구에게 어떻게 제시할지 결정하는 것”에 가깝게 설계했습니다.

---

## What We Built

| Area | Built |
|---|---|
| Gameplay | 공식 시나리오, 사건 브리핑, 현장 정보, 증거/용의자/타임라인, 최종 추리 |
| AI | 용의자 심문, 최종 추리 채점, prompt policy, response policy resolver |
| Platform | 시나리오 목록/검색, 북마크, 리뷰, 플레이 기록, 커스텀 시나리오 확장 구조 |
| Web/App | Flutter Android 앱과 React/Vite 웹 프론트 공존 |
| Infra | Blue-Green 배포, external data server, S3 backup, Grafana/Loki/n8n 운영 알림 |
| LLMOps | AI_CALL/AI_CALL_CONTEXT 로그, token/latency/failure/fallback 집계, 비용/레이트리밋 보고서 |
| QA | Android E2E, Web E2E, public-safe QA report, 스포일러/정답성 정보 분리 |

---

## Repositories

| Repository | Purpose |
|---|---|
| [`start-up-project`](https://github.com/Final-Project-sixteam-company/start-up-project) | Spring Boot backend, AI/gameplay domain, infra and LLMOps docs |
| [`project-fe`](https://github.com/Final-Project-sixteam-company/project-fe) | Flutter Android app |
| [`clueroom-toss-miniapp`](https://github.com/Final-Project-sixteam-company/clueroom-toss-miniapp) | React/Vite web deployment surface |

---

## System Shape

```mermaid
flowchart LR
    subgraph Clients
        Web[Web]
        Android[Android App]
    end

    subgraph Runtime
        Nginx[Nginx]
        API[Spring Boot API]
        AI[AI Provider]
    end

    subgraph Data
        MySQL[(MySQL)]
        Redis[(Redis)]
        S3[S3 Backup]
    end

    subgraph Observe
        Alloy[Alloy]
        Loki[Loki]
        Grafana[Grafana]
        N8N[n8n Slack Alerts]
    end

    Web --> Nginx --> API
    Android --> Nginx
    API --> MySQL
    API --> Redis
    API --> AI
    MySQL --> S3
    API --> Alloy --> Loki --> Grafana
    Loki --> N8N
```

운영 기준은 `prod API + data server + ops monitoring` 구조입니다.`r`nScale-out은 Terraform app node와 수동 Nginx load balancing으로 PoC를 완료했고, 현재 운영 기본 구조로 과장하지 않습니다.

---

## Safety Rules for AI Suspects

AI 용의자는 정답을 알고 연기하는 모델이 아닙니다. 백엔드가 만든 안전한 문맥만 보고 짧게 답합니다.

```text
Do not pass culprit identity to NPC prompt.
Do not pass full solution text to NPC prompt.
Do not expose hidden suspect secrets before unlock.
Do not let the model decide response policy.
Do keep answers short, grounded, and policy-bound.
```

LLMOps 보고서도 같은 원칙을 따릅니다.

```text
raw prompt 저장 금지
AI 답변 원문 저장 금지
사용자 질문 전문 저장 금지
token/session id 공개 금지
정답/후보 상세 public report 노출 금지
```

---

## Stack

| Layer | Stack |
|---|---|
| Backend | Java 21, Spring Boot 4, Spring Security, JPA, QueryDSL |
| Data | MySQL 8.4, Redis |
| AI/LLMOps | promptVersion, templateHash, AI_CALL, AI_CALL_CONTEXT |
| Android | Flutter, Dart |
| Web | React, TypeScript, Vite |
| Infra | AWS Lightsail, Docker Compose, Nginx Blue-Green, S3 backup |
| Monitoring | Grafana, Loki, Alloy, n8n |
| QA | Playwright, Android emulator, Web E2E, public-safe reports |

---

## Current Focus

```text
1. 웹 배포 표면의 모바일 사용성 안정화
2. 증거 상세 guidance / 함께 볼 증거 / 추천 질문 UX 개선
3. AI 심문 prompt token 비용 절감
4. QA traffic과 organic traffic을 분리한 LLMOps 비용 측정
5. Android 앱과 웹이 같은 백엔드 계약을 공유하도록 인증/북마크/리뷰/플레이 기록 호환 유지
```

---

## Links

| Link | URL |
|---|---|
| Web | https://www.clueroom.xyz |
| API Health | https://api.clueroom.xyz/actuator/health |
| Backend | https://github.com/Final-Project-sixteam-company/start-up-project |
| Android | https://github.com/Final-Project-sixteam-company/project-fe |
| Web Frontend | https://github.com/Final-Project-sixteam-company/clueroom-toss-miniapp |
