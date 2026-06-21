# 🛡️ 기업형 고가용성 인프라 구축 및 통합 보안 관제(IDS/SIEM) 프로젝트

<p align="center">
  <img src="https://img.shields.io/badge/GNS3-2F6B8A?style=for-the-badge&logo=gns3&logoColor=white" alt="GNS3">
  <img src="https://img.shields.io/badge/Rocky%20Linux%209-CD2C2E?style=for-the-badge&logo=redhat&logoColor=white" alt="Rocky Linux">
  <img src="https://img.shields.io/badge/Kali%20Linux-557C94?style=for-the-badge&logo=kali-linux&logoColor=white" alt="Kali Linux">
  <img src="https://img.shields.io/badge/Suricata%20(IDS)-EF3B2C?style=for-the-badge&logo=suricata&logoColor=white" alt="Suricata">
  <img src="https://img.shields.io/badge/Elastic%20Stack-005571?style=for-the-badge&logo=elastic-stack&logoColor=white" alt="Elastic Stack">
</p>

## 📌 프로젝트 개요
본 프로젝트는 **GNS3 시뮬레이터**를 활용하여 인프라 장애 조치(Failover) 및 게이트웨이 이중화가 완벽히 구현된 **3-Tier 기업형 네트워크 환경**을 설계하고, 외부 공격 구역 및 격리 구역(DMZ)을 체계적으로 분리했습니다. 

또한, 실제 웹 취약점 및 무차별 대입 공격을 수행한 후, **Suricata(IDS)**로 유입되는 악성 트래픽을 정밀 탐지하고 **Elasticsearch & Kibana (SIEM)** 환경을 연동하여 전 세계 위협 인텔리전스 및 프로토콜별 통계를 **실시간 통합 관제 대시보드로 시각화**한 종합 보안 인프라 구축 프로젝트입니다.

---

## 🛠️ 개발 환경 및 기술 스택 (Tech Stacks)

| 분류 | 기술 스택 / 활용 도구 |
| :--- | :--- |
| **인프라 및 가상화** | GNS3 Architecture, Rocky Linux 9 |
| **보안 및 공격 테스트** | Kali Linux, Suricata (IDS) |
| **로그 수집 및 시각화** | Elastic Stack (Filebeat, Elasticsearch, Kibana) |
| **구축 서버** | WordPress Web Server, DVWA (취약점 테스트용), DNS (Master/Slave) |

---

## 📐 네트워크 아키텍처 (Network Topology)

> **3-Tier 아키텍처 및 핵심 장비 이중화 구조**
> *기에 GNS3 망구성도 이미지 주소를 입력하거나 레포지토리에 올린 이미지 경로를 적어주세요.*
> 예시: `![Topology](./image_27ac17.jpg)`

### 1️⃣ OUTSIDE (외부 인터넷망)
* **라우터 이중화 (R1, R2):** 외부 공인 IP 대역(`192.168.0.37`)으로부터 인바운드 트래픽 진입 시 부하 분산 및 백업 경로 제공.

### 2️⃣ FIREWALL (방화벽 이중화 영역)
* **FW1, FW2 고가용성(HA) 구성:** 서브 인터페이스(`Fa0/1.100`, `Fa0/1.200`) 맵핑 및 **VRRP 프로토콜(`vrrp 10.10.100.3`)** 기반의 게이트웨이 이중화 달성. 단일 방화벽 장애 시에도 무중단 서비스 유지.

### 3️⃣ INSIDE (사내 내부망)
* 내부 스위치(`SW1`, `SW2`, `SW3`) 간 백업 링크 및 트래픽 분리를 위해 VLAN 구조 설계. 사내 직원 PC 배치.

### 4️⃣ DMZ (비무장 지대 서버 구역)
* **SPAN (포트 미러링):** 분배 백본 스위치(`SW10`)에서 웹 서버 트래픽(`Fa 1/3`, `Fa 1/4`)을 복사하여 침입탐지시스템인 **Suricata(`Fa 1/10`)**로 상시 전송하도록 구성.
* **이중화 네임서버:** `DNS_MASTER` 및 `DNS_SLAVE` 구조 구축.
* **취약 웹 타겟:** 모의 침투를 위해 `WordPress` 및 `DVWA` 웹 애플리리케이션 가동.

---

## 📊 SIEM 실시간 통합 보안 관제 (Kibana Dashboard)

> **Suricata 탐지 기반의 실시간 위협 모니터링**
> *여기에 키바나 대시보드 이미지 주소를 입력하거나 레포지토리에 올린 이미지 경로를 적어주세요.*
> 예시: `![Kibana Dashboard](./image_273ec2.png)`

Kali Linux의 침투 테스트 도구(`sqlmap`, `Hydra` 등)를 이용해 DMZ 내 웹 서버에 공격을 수행한 결과, Suricata가 이를 탐지하여 Kibana 대시보드 상에 실시간으로 매칭시킨 모니터링 화면입니다.

### 🔍 관제 위젯별 주요 모니터링 기능
* **프로토콜 통계 (Protocols Count):** 전체 네트워크 패킷 중 TCP(`2,003건`), UDP(`1,352건`), ICMP(`13건`)의 트래픽 점유율을 명확히 분류.
* **주요 위협 룰 탐지 (Alert Summary):** * `BruteForce Attack`: 내부 시스템 패스워드 크래킹 시도 **373건** 탐지 및 방어 유효성 검증.
  * `SQL Injection & URL`: 웹 애플리케이션 파라미터 변조 공격 시도 **150건 / 149건** 정밀 수집.
  * `Command Injection & XSS`: 시스템 권한 탈취 시도 소량 유입 탐지 확인.
* **Geo-IP 기반 공격 진원지 분석:**
  * **Detect_Map & Word Cloud**: 로그 내 IP 데이터 기반 지리 정보 맵핑 결과, **China(CN, 80.74%)** 및 Asia 대륙(84.2%) 중심의 공격 유입을 직관적으로 확인.
* **시계열 트래픽 추이 (@timestamp per minute):**
  * 분당 트래픽 유입량을 실시간 꺾은선 그래프로 분석하여 **14:55 및 15:00 전후로 발생한 대규모 스캔 공격(Traffic Spike)** 흐름을 정확히 역추적 및 인지 가능.

---

## 💡 프로젝트 핵심 성과 및 역량
1. **인프라 이중화 및 가용성 확보:** VRRP 및 라우터 홉 설정을 통해 단일 장애점(SPOF)이 없는 견고한 인프라 인식 및 트러블슈팅 경험.
2. **네트워크 데이터 미러링(SPAN):** 스위치 단에서 발생하는 인바운드/오바운드 데이터를 유실 없이 보안 장비로 포워딩하는 메커니즘 이해.
3. **오픈소스 IDS & SIEM 데이터 파이프라인 정립:** Suricata의 이벤트 로그(`eve.json`)를 Filebeat로 실시간 파싱하고, Elasticsearch 색인 과정을 거쳐 Kibana로 파이프라인을 완전 자동화하는 엔지니어링 역량 획득.
