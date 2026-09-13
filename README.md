# aws-cloudwatch-sns-observability
# AWS Proactive Monitoring & Incident Alerting (CloudWatch + SNS)

Implementação prática de um pipeline de observabilidade, governança de custos e resposta a incidentes na AWS seguindo os pilares de **Excelência Operacional** e **Otimização de Custos** do AWS Well-Architected Framework.

---

## 🏛️ Arquitetura da Solução

* **Camada de Computação:** Instância Amazon EC2 rodando Amazon Linux 2023.
* **Camada de Telemetria:** Métricas operacionais e de integridade coletadas via Amazon CloudWatch.
* **Camada de Notificação:** Arquitetura desacoplada via Pub/Sub utilizando Amazon SNS com endpoint de e-mail verificado.
* **Governança Orçamentária (FinOps):** Alarme de faturamento preventivo (*Estimated Charges*).

---

## 🎯 Componentes Implementados

1. **Amazon CloudWatch Alarms:**
   * `EC2-Alta-Utilizacao-CPU`: Limiar estático acionado quando `CPUUtilization >= 50%` por 1 período de 60 segundos.
   * `EC2-StatusCheckFailed`: Monitoramento contínuo de falhas no hypervisor/hardware da instância.
   * `Alarme-Gasto-Maior-$5`: Alarme orçamentário para controle preventivo contra custos residuais.

2. **Amazon SNS (Simple Notification Service):**
   * Tópico `alerta-alta-cpu-servidores` gerenciando subscrição e entrega de mensagens em tempo real.

3. **Engenharia de Confiabilidade (Teste Sintético):**
   * Simulação de sobrecarga de processador utilizando o utilitário `stress` via terminal:
   ```bash
   sudo dnf install stress -y
   stress --cpu 1 --timeout 300


📊 Evidências de Execução
Simulação de Carga: Instalação e execução do stress na instância EC2.

Violação de Limiar: Gráfico do CloudWatch registrando pico de 88% de CPU rompendo a linha de base.

Entrega de Incidente: Notificação oficial recebida via e-mail com metadados detalhados de diagnóstico.

[Cassiano Alarmes e Notificacoes AWS CW.pdf](https://github.com/user-attachments/files/32162857/Cassiano.Alarmes.e.Notificacoes.AWS.CW.pdf)
## 📑 Apresentação do Projeto

![Slide 1](docs/slide1.png)
![Slide 2](docs/slide2.png)
![Slide 3](docs/slide3.png)
![Slide 4](docs/slide4.png)
![Slide 5](docs/slide5.png)
![Slide 6](docs/slide6.png)


