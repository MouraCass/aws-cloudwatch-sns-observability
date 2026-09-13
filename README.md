# AWS Proactive Monitoring & Incident Alerting (CloudWatch + SNS)

Implementação prática de uma esteira de observabilidade, governança financeira e resposta automatizada a incidentes na AWS, estruturada sob os pilares de **Excelência Operacional** e **Otimização de Custos** do *AWS Well-Architected Framework*.

---

## 🏛️ Visão Geral da Arquitetura

A solução implementa um fluxo desacoplado de telemetria e mensageria para detecção precoce de anomalias computacionais e controle orçamentário:

```text
[ Amazon EC2 ]  ──(Métricas de CPU & Status)──>  [ Amazon CloudWatch ]  ──(Gatilho ALARM)──>  [ Amazon SNS ]  ──>  [ Notificação por E-mail ]
[ AWS Billing ] ──(Estimated Charges)────────>  [ Amazon CloudWatch ]  ──(Gatilho ALARM)──>  [ Amazon SNS ]  ──>  [ Notificação por E-mail ]
```

* **Telemetria de Baixo Acoplamento:** As regras de monitoramento operam no CloudWatch de forma isolada das instâncias, sem a necessidade de agentes pesados para métricas padrão do hypervisor.
* **Mensageria Assíncrona (Pub/Sub):** O Amazon SNS atua como intermediário, permitindo plugar futuros canais (Slack, PagerDuty, AWS Lambda para auto-remediação) sem reconfigurar os alarmes.
* **FinOps Preventivo:** Monitoramento ativo da métrica `EstimatedCharges` em moeda local (USD), evitando cobranças surpresa por recursos não desprovisionados.

---

## 📑 Etapas da Implementação & Evidências Técnicas

### 1. Governança e Inicialização do Ambiente
Criação do escopo do projeto, garantindo o alinhamento das configurações com as metas de monitoramento contínuo.

<img width="3000" height="1688" alt="Slide 1 - Capa" src="https://github.com/user-attachments/assets/5dd8de4e-6ba5-4c78-a566-7081f7425a9d" />

---

### 2. Painel de Alarmes e Camadas de Observabilidade
Definição dos limiares operacionais no Amazon CloudWatch divididos em três vetores estratégicos:
* **Integridade de Infraestrutura (`StatusCheckFailed`):** Monitora falhas no hardware do host da AWS e no sistema operacional da EC2.
* **Performance Computacional (`CPUUtilization`):** Monitoramento contínuo com limiar estático para contenção de sobrecarga.
* **Governança Financeira (`EstimatedCharges`):** Alarme acionado caso os custos estimados ultrapassem **$5 USD** em uma janela de 6 horas.

<img width="3000" height="1688" alt="Slide 2 - Governança CloudWatch" src="https://github.com/user-attachments/assets/781f8698-2f02-4115-aa67-a129e67bab3e" />

---

### 3. Desacoplamento de Alertas com Amazon SNS
Configuração do tópico centralizador `alerta-alta-cpu-servidores` utilizando o modelo Publish/Subscribe:
* **Protocolo:** Email.
* **Confirmação de Inscrição:** Assinatura autenticada com status `Confirmed`, garantindo que o canal de entrega seja válido e responsivo antes da ativação dos disparos.

<img width="3000" height="1688" alt="Slide 3 - Desacoplamento SNS" src="https://github.com/user-attachments/assets/9bdc21aa-7e84-4678-bf30-04623da4e54e" />

---

### 4. Engenharia de Confiabilidade: Teste de Carga Sintética
Validação prática do alarme forçando uma anomalia em ambiente controlado. Acessou-se a instância EC2 via terminal SSH (Amazon Linux 2023) executando o utilitário `stress`:

```bash
# Atualização de pacotes e instalação do utilitário de benchmark
sudo dnf install stress -y

# Alocação de 100% de 1 núcleo de CPU por 300 segundos (5 minutos)
stress --cpu 1 --timeout 300
```


