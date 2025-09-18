# 베스핀 AWS 첫 프로젝트 — Ansible (ELK 8.15.5 기준)

## Topology (고정)
- ES 192.168.1.50
- LS 192.168.1.51
- KB 192.168.1.52
- Web/Filebeat 192.168.1.53
- Prometheus 192.168.1.54
- Grafana 192.168.1.55
- OS: Ubuntu 22.04, Network: Bridge 192.168.1.0/24

## 목적
- Logstash 파서를 Ansible로 **일관 배포**하고, 매 배포마다 **문법검사→재시작→헬스체크**까지 자동화.
- `/etc/logstash/conf.d`는 **항상 초기화**, 아래 3개만 배포:
  - `01-input-beats.conf`, `20-parse-weblogs.conf`, `30-output-es.conf`
- `pipelines.yml`은 `conf.d/*.conf`만 바라봄.
- `logstash.yml`은 `api.http.host: 0.0.0.0`만 설정(※ `http.host` 금지).

## 사용법
```bash
ansible-playbook -i inventory.ini playbooks/ls.yml -K

# elkf-IaC
