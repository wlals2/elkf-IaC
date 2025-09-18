# ELKF on Ubuntu 22.04 — Ansible IaC (Elastic 8.15.5)

베스핀 AWS 첫 프로젝트의 **재현 가능한** ELKF 스택(Elasticsearch / Logstash / Kibana / Filebeat / Web)을 Ansible로 자동화합니다.  
목표는 다음과 같습니다.

- **Logstash 파서**를 idempotent하게 배포: `/etc/logstash/conf.d` 초기화 → 3개 파일(01/20/30)만 반영
- 배포 시 **문법검사(-t) → 재시작 → 헬스체크(5044/9600/hash)** 자동 수행
- **Filebeat → Logstash** 고정(ES 직행 금지), ES 인덱스는 `webapp2-%{+YYYY.MM.dd}`
- Kibana는 ES에 붙고 대시보드 확인
- **Web 앱**은 `/var/log/webapp2/*.log`를 출력 → Filebeat 수집 → Logstash 파싱 → ES 색인 → Kibana 시각화

---

## 🗺️ 아키텍처 & 고정값

- 네트워크: Bridge `192.168.1.0/24`
- OS: Ubuntu 22.04 (VM Workstation)
- 버전: **Elastic 8.15.5** (ES, Kibana, Logstash, Filebeat)
- 노드:
  - **ES**: `192.168.1.50`
  - **Logstash (LS)**: `192.168.1.51`
  - **Kibana (KB)**: `192.168.1.52`
  - **Web/Filebeat**: `192.168.1.53`

```mermaid
flowchart LR
  subgraph Web[Web / Filebeat (192.168.1.53)]
    L[Web App\n/var/log/webapp2/*.log]
    FB[Filebeat 8.15.5]
    L --> FB
  end

  subgraph LS[Logstash (192.168.1.51)]
    LS01[01-input-beats]
    LS20[20-parse-weblogs (GROK)]
    LS30[30-output-es]
    FB -->|beats:5044| LS01 --> LS20 --> LS30
  end

  subgraph ES[Elasticsearch (192.168.1.50)]
    IDX[(webapp2-YYYY.MM.dd)]
  end

  subgraph KB[Kibana (192.168.1.52)]
    UI[Kibana UI:5601]
  end

  LS30 -->|HTTP| IDX
  KB -->|HTTP| ES
```
## 📁 Repo 구조

```bash
ansible/
├── .gitignore
├── README.md
├── inventory.ini
├── group_vars/
│   └── all.yml
└── playbooks/
    ├── bootstrap.yml   # Elastic APT repo 등록
    ├── es.yml          # Elasticsearch 8.15.5 단일 노드 (보안 OFF)
    ├── kb.yml          # Kibana 8.15.5 (보안 OFF)
    ├── ls.yml          # Logstash 파이프라인 (conf.d 초기화 + 3파일 + 검증/재시작/헬스체크)
    ├── filebeat.yml    # Filebeat 8.15.5 → Logstash:5044
    └── web_app.yml     # (옵션) Web 앱 배포/로그 디렉터리/유닛 관리

```

## 🚀 배포 순서

```
ansible-playbook -i inventory.ini playbooks/bootstrap.yml -K
```

- **컴포 넌트 설치/배포**
```
ansible-playbook -i inventory.ini playbooks/es.yml -K       # Elasticsearch
ansible-playbook -i inventory.ini playbooks/kb.yml -K       # Kibana
ansible-playbook -i inventory.ini playbooks/ls.yml -K       # Logstash (파서+파이프라인)
ansible-playbook -i inventory.ini playbooks/filebeat.yml -K # Filebeat (→ Logstash)
ansible-playbook -i inventory.ini playbooks/web_app.yml -K  # (옵션) Web 앱
```
