# AWS Proactive Monitoring & Incident Alerting (CloudWatch + SNS)

Implementação prática de uma esteira de observabilidade, governança financeira e resposta automatizada a incidentes na AWS, estruturada sob os pilares de **Excelência Operacional** e **Otimização de Custos** do *AWS Well-Architected Framework*.

---

## 🏛️ Visão Geral da Arquitetura

A solução implementa um fluxo desacoplado de telemetria e mensageria para detecção precoce de anomalias computacionais e controle orçamentário:

```text
[ Amazon EC2 ]  ──(Métricas de CPU & Status)──>  [ Amazon CloudWatch ]  ──(Gatilho ALARM)──>  [ Amazon SNS ]  ──>  [ Notificação por E-mail ]
[ AWS Billing ] ──(Estimated Charges)────────>  [ Amazon CloudWatch ]  ──(Gatilho ALARM)──>  [ Amazon SNS ]  ──>  [ Notificação por E-mail ]
```

* **Telemetria Isolada:** Regras operam diretamente no CloudWatch sem sobrecarregar a camada de software interna da instância.
* **Mensageria Assíncrona (Pub/Sub):** O Amazon SNS atua como intermediário desacoplado, permitindo plugar novos canais de alerta (Slack, PagerDuty ou funções AWS Lambda para remediação automatizada) sem precisar reconfigurar os alarmes.
* **FinOps Preventivo:** Monitoramento ativo da métrica `EstimatedCharges` em USD, evitando cobranças surpresa por recursos ativos na conta.

---

## 📑 Etapas da Implementação & Evidências Técnicas

### 1. Inicialização e Escopo do Projeto
Estruturação e planejamento das métricas críticas necessárias para sustentar a integridade de instâncias de missão crítica e governança orçamentária na nuvem.

<img width="3000" height="1688" alt="Slide 1 - Capa" src="https://github.com/user-attachments/assets/5dd8de4e-6ba5-4c78-a566-7081f7425a9d" />

---

### 2. Painel de Alarmes e Camadas de Observabilidade
Definição dos limiares operacionais no Amazon CloudWatch divididos em três vetores estratégicos:
* **Integridade de Infraestrutura (`StatusCheckFailed`):** Detecção proativa de falhas no hypervisor físico e no sistema operacional da instância `i-0577e84d23d286fc7`.
* **Performance Computacional (`CPUUtilization`):** Monitoramento contínuo com limiar estático para contenção de sobrecarga.
* **Governança Financeira (`EstimatedCharges`):** Disparo preventivo caso os custos estimados ultrapassem **$5 USD** em uma janela de 6 horas.

<img width="3000" height="1688" alt="Slide 2 - Governança CloudWatch" src="https://github.com/user-attachments/assets/781f8698-2f02-4115-aa67-a129e67bab3e" />

---

### 3. Desacoplamento de Alertas com Amazon SNS
Configuração do tópico centralizador `alerta-alta-cpu-servidores` utilizando o modelo Publish/Subscribe:
* **Protocolo:** Email.
* **Status:** `Confirmed` (assinatura validada via handshake de e-mail).
* **Propósito:** Isola a origem do alerta da camada de destino, permitindo múltiplos consumidores para o mesmo evento de incidente.

<img width="3000" height="1688" alt="Slide 3 - Desacoplamento SNS" src="https://github.com/user-attachments/assets/9bdc21aa-7e84-4678-bf30-04623da4e54e" />

---

### 4. Engenharia de Confiabilidade: Teste de Carga Sintética
Validação prática executada diretamente no terminal SSH da instância EC2 (Amazon Linux 2023) utilizando o utilitário `stress` para induzir 100% de uso de CPU:

```bash
# Atualização de pacotes e instalação do utilitário de benchmark
sudo dnf install stress -y

# Alocação de 100% de 1 núcleo de CPU por 300 segundos (5 minutos)
stress --cpu 1 --timeout 300
```

<img width="3000" height="1688" alt="Slide 4 - Teste de Carga Linux" src="https://github.com/user-attachments/assets/1d3d4a2e-5315-49aa-a56d-21340bdb8d79" />

---

### 5. Violação do Limiar e Telemetria no CloudWatch
Acompanhamento da métrica `CPUUtilization` no console em tempo real:
* **Threshold Configurado:** $\ge 50\%$ por 1 período de 60 segundos.
* **Comportamento Observado:** A carga sintética forçou o rompimento imediato do limiar estabelecido, disparando a transição de estado da regra no painel.

<img width="3000" height="1688" alt="Slide 5 - Comportamento da Métrica" src="https://github.com/user-attachments/assets/4fb185e2-e871-4b29-8eaf-1c93b26d7f53" />

---

### 6. Disparo do Incidente e Entrega da Mensagem
Validação final do ciclo completo de resposta operacional:
* **Pico Atingido:** **88.07%** de CPU registrado no payload oficial da telemetria.
* **Transição de Estado:** `OK` $\rightarrow$ `ALARM`.
* **Payload Recebido:** Notificação entregue na caixa de entrada contendo metadados completos de diagnóstico (Account ID `231348293040`, Instance ID `i-0577e84d23d286fc7` e timestamp UTC) para aceleração de troubleshooting.

<img width="3000" height="1688" alt="Slide 6 - Notificação Recebida" src="https://github.com/user-attachments/assets/00d04358-fccd-4e69-86e8-382e34214bf3" />

> 📄 **Apresentação em Formato Original:** [Acessar apresentação em PDF](./Cassiano%20Alarmes%20e%20Notificacoes%20AWS%20CW%20ofc.pdf)

---

## 🧹 Higiene de Recursos & Práticas FinOps

Procedimentos executados ao término da validação para mitigar custos residuais na conta:
1. **Terminação da EC2:** Encerramento definitivo da instância de testes (`i-0577e84d23d286fc7`).
2. **Desalocação de Volumes EBS:** Exclusão de discos órfãos para evitar cobranças de armazenamento persistente.
3. **Limpeza de Métricas:** Exclusão dos alarmes com status `Insufficient data` provenientes da máquina terminada, mantendo ativo permanentemente o alarme preventivo de **Billing ($5)**.
