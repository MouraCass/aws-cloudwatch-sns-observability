# AWS Proactive Monitoring & Incident Alerting (CloudWatch + SNS)

Implementação prática de uma esteira de observabilidade, governança financeira e resposta automatizada a incidentes na AWS, estruturada sob os pilares de **Excelência Operacional** e **Otimização de Custos** do *AWS Well-Architected Framework*.

---

## 🏛️ Visão Geral da Arquitetura

A solução implementa um fluxo desacoplado de telemetria e mensageria para detecção precoce de anomalias computacionais e controle orçamentário:

```text
[ Amazon EC2 ]  ──(Métricas de CPU & Status)──>  [ Amazon CloudWatch ]  ──(Gatilho ALARM)──>  [ Amazon SNS ]  ──>  [ Notificação por E-mail ]
[ AWS Billing ] ──(Estimated Charges)────────>  [ Amazon CloudWatch ]  ──(Gatilho ALARM)──>  [ Amazon SNS ]  ──>  [ Notificação por E-mail ]

# Atualização de pacotes e instalação do utilitário de benchmark
sudo dnf install stress -y

# Alocação de 100% de 1 núcleo de CPU por 300 segundos (5 minutos)
stress --cpu 1 --timeout 300
