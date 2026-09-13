# AWS Proactive Monitoring & Incident Alerting (CloudWatch + SNS)

Implementação prática de uma esteira de observabilidade, resposta automatizada a incidentes e governança de custos na AWS, seguindo os pilares de **Excelência Operacional** e **Otimização de Custos** do AWS Well-Architected Framework.

---

## 📑 Apresentação Visual do Projeto

Confira abaixo o fluxo completo da arquitetura e as evidências técnicas de execução:

![Slide 1 - Capa](docs/slide1.png)
<img width="3000" height="1688" alt="Cassiano Alarmes e Notificacoes AWS CW_page-0001" src="https://github.com/user-attachments/assets/5dd8de4e-6ba5-4c78-a566-7081f7425a9d" />

![Slide 2 - Governança CloudWatch](docs/slide2.png)
<img width="3000" height="1688" alt="Cassiano Alarmes e Notificacoes AWS CW_page-0002" src="https://github.com/user-attachments/assets/781f8698-2f02-4115-aa67-a129e67bab3e" />

![Slide 3 - Desacoplamento SNS](docs/slide3.png)
<img width="3000" height="1688" alt="Cassiano Alarmes e Notificacoes AWS CW_page-0003" src="https://github.com/user-attachments/assets/9bdc21aa-7e84-4678-bf30-04623da4e54e" />

![Slide 4 - Teste de Carga Linux](docs/slide4.png)
<img width="3000" height="1688" alt="Cassiano Alarmes e Notificacoes AWS CW_page-0004" src="https://github.com/user-attachments/assets/1d3d4a2e-5315-49aa-a56d-21340bdb8d79" />

![Slide 5 - Comportamento da Métrica](docs/slide5.png)
<img width="3000" height="1688" alt="Cassiano Alarmes e Notificacoes AWS CW_page-0005" src="https://github.com/user-attachments/assets/4fb185e2-e871-4b29-8eaf-1c93b26d7f53" />

![Slide 6 - Notificação Recebida](docs/slide6.png)
<img width="3000" height="1688" alt="Cassiano Alarmes e Notificacoes AWS CW_page-0006" src="https://github.com/user-attachments/assets/00d04358-fccd-4e69-86e8-382e34214bf3" />


> 📄 **Documento Completo:** [Baixar apresentação em PDF](./Cassiano%20Alarmes%20e%20Notificacoes%20AWS%20CW%20ofc.pdf)

---

## 🏛️ Arquitetura da Solução

* **Camada de Computação:** Instância Amazon EC2 rodando Amazon Linux 2023.
* **Camada de Telemetria:** Métricas operacionais e de integridade coletadas via Amazon CloudWatch.
* **Camada de Notificação:** Arquitetura desacoplada via Pub/Sub utilizando Amazon SNS com endpoint de e-mail verificado.
* **Governança Orçamentária (FinOps):** Alarme de faturamento preventivo (*Estimated Charges*).

---

## 🎯 Componentes Implementados

### 1. Amazon CloudWatch Alarms
* `EC2-Alta-Utilizacao-CPU`: Limiar estático acionado quando `CPUUtilization >= 50%` por 1 período de 60 segundos.
* `EC2-StatusCheckFailed`: Monitoramento contínuo de falhas no hypervisor/hardware da instância.
* `Alarme-Gasto-Maior-$5`: Alarme orçamentário para controle preventivo contra custos residuais.

### 2. Amazon SNS (Simple Notification Service)
* Tópico `alerta-alta-cpu-servidores` gerenciando subscrição e entrega de mensagens em tempo real via protocolo EMAIL.

### 3. Engenharia de Confiabilidade (Teste Sintético)
* Simulação de sobrecarga de processador utilizando o utilitário `stress` via terminal Linux:
```bash
sudo dnf install stress -y
stress --cpu 1 --timeout 300
