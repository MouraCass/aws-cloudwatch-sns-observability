# AWS Proactive Monitoring & Incident Alerting (CloudWatch + SNS)

Implementação prática de uma esteira de observabilidade, governança financeira e resposta automatizada a incidentes na AWS, estruturada sob os pilares de **Excelência Operacional** e **Otimização de Custos** do *AWS Well-Architected Framework*.

O projeto simula um cenário de monitoramento proativo de infraestrutura, utilizando **Amazon CloudWatch** para coleta e avaliação de métricas e **Amazon SNS** para desacoplar a detecção de eventos da camada de notificação.

A implementação contempla monitoramento de utilização de CPU, integridade da instância EC2 e controle preventivo de custos por meio da métrica de cobranças estimadas da AWS.

---

## 🏛️ Visão Geral da Arquitetura

A solução implementa um fluxo desacoplado de telemetria e mensageria para detecção precoce de anomalias computacionais e controle orçamentário:

```text
[ Amazon EC2 ]  ──(Métricas de CPU & Status)──>  [ Amazon CloudWatch ]  ──(Gatilho ALARM)──>  [ Amazon SNS ]  ──>  [ Notificação por E-mail ]

[ AWS Billing ] ──(Estimated Charges)────────>  [ Amazon CloudWatch ]  ──(Gatilho ALARM)──>  [ Amazon SNS ]  ──>  [ Notificação por E-mail ]
```

* **Telemetria Isolada:** As métricas são monitoradas diretamente pelo CloudWatch, sem necessidade de uma camada adicional de software para coleta.
* **Mensageria Assíncrona (Pub/Sub):** O Amazon SNS atua como intermediário desacoplado, permitindo conectar diferentes consumidores ao mesmo evento, como e-mail, AWS Lambda, Slack ou ferramentas de gerenciamento de incidentes.
* **Detecção Proativa:** CloudWatch Alarms transforma métricas de infraestrutura em eventos acionáveis a partir de thresholds previamente definidos.
* **FinOps Preventivo:** Monitoramento ativo da métrica `EstimatedCharges` em USD, permitindo identificar antecipadamente possíveis cobranças acima do limite definido para o ambiente de laboratório.
* **Resposta Operacional:** O fluxo simula o ciclo `Detect → Alert → Notify`, fornecendo informações necessárias para iniciar o processo de troubleshooting.

---

## 📑 Etapas da Implementação & Evidências Técnicas

### 1. Inicialização e Escopo do Projeto

Estruturação e planejamento das métricas críticas necessárias para sustentar a integridade de instâncias de missão crítica, monitoramento de performance e governança orçamentária na nuvem.

O escopo inicial foi dividido em três vetores principais:

* **Integridade da infraestrutura:** identificação de falhas através das verificações de status da instância.
* **Performance computacional:** acompanhamento da utilização de CPU para identificação de possíveis situações de sobrecarga.
* **Governança financeira:** acompanhamento das cobranças estimadas da conta AWS.

<img width="3000" height="1688" alt="Slide 1 - Capa" src="https://github.com/user-attachments/assets/5dd8de4e-6ba5-4c78-a566-7081f7425a9d" />

---

### 2. Painel de Alarmes e Camadas de Observabilidade

Definição dos limiares operacionais no Amazon CloudWatch divididos em três vetores estratégicos:

* **Integridade de Infraestrutura (`StatusCheckFailed`):** Detecção proativa de falhas relacionadas às verificações de status da instância EC2.
* **Performance Computacional (`CPUUtilization`):** Monitoramento contínuo com limiar estático para identificação de condições de sobrecarga.
* **Governança Financeira (`EstimatedCharges`):** Disparo preventivo caso os custos estimados ultrapassem **$5 USD** em uma janela de 6 horas.

A utilização de múltiplos alarmes permite observar o ambiente sob diferentes perspectivas, combinando disponibilidade, performance e custos em uma mesma estratégia operacional.

<img width="3000" height="1688" alt="Slide 2 - Governança CloudWatch" src="https://github.com/user-attachments/assets/781f8698-2f02-4115-aa67-a129e67bab3e" />

---

### 3. Desacoplamento de Alertas com Amazon SNS

Configuração do tópico centralizador `alerta-alta-cpu-servidores` utilizando o modelo Publish/Subscribe:

* **Protocolo:** Email.
* **Status:** `Confirmed` (assinatura validada via handshake de e-mail).
* **Propósito:** Isolar a origem do alerta da camada de destino, permitindo múltiplos consumidores para o mesmo evento de incidente.

<img width="3000" height="1688" alt="Slide 3 - Desacoplamento SNS" src="https://github.com/user-attachments/assets/9bdc21aa-7e84-4678-bf30-04623da4e54e" />

O desacoplamento permite que o mecanismo de detecção permaneça independente do canal utilizado para comunicação.

Em uma evolução do projeto, o mesmo tópico poderia distribuir eventos para diferentes consumidores:

```text
                    Amazon SNS
                         │
          ┌──────────────┼──────────────┐
          ▼              ▼              ▼
        E-mail         Lambda      Incident Management
```

Dessa forma, novos canais de resposta podem ser adicionados sem necessidade de alterar a lógica responsável pela detecção do incidente.

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

O teste de carga foi utilizado para criar uma condição controlada de alta utilização de CPU e validar o comportamento do mecanismo de monitoramento.

O fluxo esperado foi:

```text
Teste de carga
      ↓
Aumento da CPU
      ↓
CloudWatch coleta a métrica
      ↓
Threshold é ultrapassado
      ↓
CloudWatch Alarm → ALARM
      ↓
Amazon SNS
      ↓
Notificação por e-mail
```

Essa abordagem permitiu validar o comportamento da arquitetura em um cenário reproduzível, em vez de depender apenas de uma configuração teórica dos alarmes.

---

### 5. Violação do Limiar e Telemetria no CloudWatch

Acompanhamento da métrica `CPUUtilization` no console em tempo real:

* **Threshold Configurado:** $\ge 50\%$ por 1 período de 60 segundos.
* **Comportamento Observado:** A carga sintética forçou o rompimento imediato do limiar estabelecido, disparando a transição de estado da regra no painel.

<img width="3000" height="1688" alt="Slide 5 - Comportamento da Métrica" src="https://github.com/user-attachments/assets/4fb185e2-e871-4b29-8eaf-1c93b26d7f53" />

O teste permitiu observar a correlação entre o aumento real da utilização da CPU e a mudança de estado do CloudWatch Alarm.

Essa validação é importante porque demonstra que o mecanismo configurado não apenas existe, mas responde efetivamente à condição operacional criada durante o teste.

---

### 6. Disparo do Incidente e Entrega da Mensagem

Validação final do ciclo completo de resposta operacional:

* **Pico Atingido:** **88.07%** de CPU registrado no payload oficial da telemetria.
* **Transição de Estado:** `OK` → `ALARM`.
* **Payload Recebido:** Notificação entregue na caixa de entrada contendo metadados de diagnóstico, incluindo identificação da instância e timestamp UTC para aceleração do troubleshooting.

<img width="3000" height="1688" alt="Slide 6 - Notificação Recebida" src="https://github.com/user-attachments/assets/00d04358-fccd-4e29-8eaf-1c93b26d7f53" />

> 🔐 **Nota de segurança:** identificadores sensíveis da conta AWS foram mascarados nesta documentação pública. O objetivo é demonstrar o funcionamento do mecanismo de alerta sem expor informações desnecessárias do ambiente.

O resultado final confirmou o funcionamento do fluxo completo:

```text
CPU elevada
   ↓
CloudWatch Metric
   ↓
Threshold Violated
   ↓
Alarm: OK → ALARM
   ↓
Amazon SNS
   ↓
E-mail
   ↓
Incidente identificado para troubleshooting
```

Esse fluxo representa um padrão básico de resposta operacional baseado em observabilidade:

> **Detect → Alert → Notify → Investigate**

---

## 🧹 Higiene de Recursos & Práticas FinOps

Procedimentos executados ao término da validação para mitigar custos residuais na conta:

1. **Terminação da EC2:** Encerramento definitivo da instância de testes.
2. **Desalocação de Volumes EBS:** Exclusão de discos órfãos para evitar cobranças de armazenamento persistente.
3. **Limpeza de Métricas:** Exclusão dos alarmes com status `Insufficient data` provenientes da máquina terminada, mantendo ativo permanentemente o alarme preventivo de **Billing ($5)**.

A etapa de cleanup foi tratada como parte do próprio ciclo do laboratório:

```text
Provision
   ↓
Configure
   ↓
Test
   ↓
Validate
   ↓
Document
   ↓
Cleanup
```

Essa prática reduz custos residuais e reforça a importância de governança financeira mesmo em ambientes destinados exclusivamente a aprendizado e experimentação.

---

## 🧠 Principais Aprendizados

Durante a implementação, foram consolidados conceitos relacionados a:

* Monitoramento de infraestrutura utilizando Amazon CloudWatch.
* Configuração e avaliação de CloudWatch Alarms.
* Monitoramento de métricas de performance.
* Utilização de Amazon SNS para mensageria desacoplada.
* Modelo Publish/Subscribe.
* Geração controlada de carga em ambiente Linux.
* Validação prática de thresholds e transições de estado.
* Construção de fluxos básicos de detecção e resposta a incidentes.
* Monitoramento preventivo de custos.
* Higiene e encerramento de recursos AWS após utilização.

Além da implementação dos serviços, o projeto reforçou uma abordagem operacional baseada em:

> **Observabilidade → Detecção → Alerta → Investigação → Remediação**

---

## 🏗️ Aplicação dos Princípios AWS Well-Architected

O projeto foi estruturado principalmente considerando dois pilares do **AWS Well-Architected Framework**:

### Excelência Operacional

Aplicação de conceitos relacionados a:

* Monitoramento contínuo.
* Detecção de condições anormais.
* Alertas automatizados.
* Resposta a incidentes.
* Observabilidade.
* Melhoria contínua dos processos operacionais.

### Otimização de Custos

Aplicação de conceitos relacionados a:

* Monitoramento de cobranças estimadas.
* Alertas preventivos.
* Consciência sobre custos durante laboratórios.
* Limpeza de recursos após os testes.
* Redução de cobranças residuais.

---

## 🚀 Possíveis Evoluções

Este projeto representa a primeira versão da solução e pode ser evoluído para aproximá-lo de um cenário operacional mais completo.

### Infrastructure as Code

Provisionar a arquitetura utilizando **Terraform**, permitindo:

* Infraestrutura versionada.
* Provisionamento reproduzível.
* Redução de configurações manuais.
* Facilidade para recriar e destruir o ambiente.

### Automated Remediation

Adicionar **AWS Lambda** ao fluxo para executar ações automatizadas após determinados eventos:

```text
CloudWatch
     ↓
SNS
     ↓
Lambda
     ↓
Automated Remediation
```

### Observability Dashboard

Expandir o projeto com dashboards contendo métricas de:

* CPU.
* Status Checks.
* Network.
* Alarm States.
* Billing.

### Incident Management

Integrar o fluxo de alertas com ferramentas externas de gerenciamento de incidentes, criando um processo mais próximo de um ambiente corporativo:

```text
CloudWatch
     ↓
SNS
     ↓
Incident Management
     ↓
Engineer
     ↓
Troubleshooting
     ↓
Remediation
```

### Multi-Resource Monitoring

Expandir o monitoramento para outros serviços AWS, como:

* Amazon RDS
* AWS Lambda
* Amazon ECS
* Amazon S3

permitindo construir uma estratégia de observabilidade mais abrangente.

---

## 📄 Documentação Complementar

> 📄 **Apresentação em Formato Original:** [Acessar apresentação em PDF](./Cassiano%20Alarmes%20e%20Notificacoes%20AWS%20CW%20ofc.pdf)

---

## 🛠️ Tecnologias e Serviços

`AWS` `Amazon EC2` `Amazon CloudWatch` `CloudWatch Alarms` `Amazon SNS` `AWS Billing` `Amazon Linux 2023` `Linux` `FinOps` `Observability` `Incident Management`

---

## 👨‍💻 Autor

**Cassiano Moura**

Cloud Support Engineer | Cloud Infrastructure | AWS

Profissional com experiência em suporte técnico enterprise N2/N3, troubleshooting, análise de causa raiz (RCA), observabilidade, APIs REST e ambientes corporativos globais.

Atualmente direcionando a carreira para **Cloud Computing**, com foco em AWS, infraestrutura, operações e arquitetura de soluções.
