# -Arquitetura-de-Rede-Segmentada-DMZ-com-pfSense-e-Suricata-IDS-IPS
Uma empresa precisa hospedar um servidor web acessível pela internet, mas deve garantir que, caso esse servidor seja invadido, o atacante não consiga atingir os computadores internos dos funcionários (LAN).

# 🛡️ Arquitetura de Rede Segmentada (DMZ) e Resposta a Incidente com pfSense e Suricata IPS

## 📋 Resumo Executivo
Este projeto simula um ambiente corporativo segmentado para demonstrar, na prática, o ciclo completo de defesa cibernética: **Arquitetura Segura (Hardening)**, **Detecção de Intrusão (IDS)**, **Contenção de Movimentação Lateral** e **Prevenção Ativa (IPS)**. 

O objetivo principal é provar como a segmentação adequada de rede e o monitoramento em tempo real impedem que o comprometimento de um servidor exposto na internet (DMZ) resulte na invasão da rede interna corporativa (LAN).

---

## 🏗️ 1. Arquitetura do Laboratório

O ambiente foi virtualizado no **Oracle VirtualBox** com isolamento total da rede hospedeira, utilizando o **pfSense** como firewall de borda e roteador central dividido em três zonas de segurança:

* **WAN (Zona Externa / Hostil - `10.10.10.0/24`):** Simula a internet, onde opera a máquina atacante (**Kali Linux** - `10.10.10.4`).
* **DMZ (Zona Desmilitarizada - `192.168.20.0/24`):** Hospeda serviços acessíveis externamente (**Metasploitable 2 / DVWA** - `192.168.20.10`), isolada da rede interna.
* **LAN (Rede Interna Corporativa - `192.168.10.0/24`):** Rede protegida onde ficam os ativos críticos e estações de trabalho (**Windows 10** - `192.168.10.50`) e Ubuntu Desktop 192.168.10.51.

---

## 🔒 2. Políticas de Firewall e Hardening (pfSense)

Para aplicar o **Princípio do Menor Privilégio** e conter possíveis invasões, foram implementadas as seguintes regras:

. **NAT / Port Forwarding (WAN → DMZ):** Redirecionamento controlado das requisições externas direcionadas ao IP da WAN do pfSense (`10.10.10.3`) para o servidor web na DMZ (`192.168.20.10`), ocultando a topologia interna.
. **Bloqueio de Movimentação Lateral (`Block DMZ net → LAN net`):** Regra posicionada no topo da interface DMZ proibindo que qualquer servidor comprometido na zona desmilitarizada inicie conexões com a rede interna (`LAN`), com registro de logs ativado.
. **Acesso Administrativo Restrito:** O painel web do pfSense (`HTTPS/443`) permanece acessível apenas a partir da rede interna (`LAN`), bloqueado para a `WAN` e `DMZ`.

<img width="1024" height="768" alt="image" src="https://github.com/user-attachments/assets/f920790a-445d-4e9e-8328-c769c092585e" />

---

## 🚨 3. Simulação de Ataque e Triagem SOC

### Etapa 3.1: Reconhecimento e Exploração (Red Team)
A partir do **Kali Linux (`10.10.10.4`)**, foram realizadas varreduras de serviços (`Nmap`) e a exploração de uma vulnerabilidade de **Command Execution** na aplicação web DVWA hospedada na DMZ, resultando na exfiltração de informações sensíveis do sistema (`/etc/passwd`).

> <img width="1280" height="780" alt="image" src="https://github.com/user-attachments/assets/efae6df0-f5a6-41f8-aaa8-8daef47ccfb7" />
<img width="1280" height="780" alt="image" src="https://github.com/user-attachments/assets/815c7d79-7cee-4945-bf77-0037b51a71ab" />



### Etapa 3.2: Detecção de Intrusão via Suricata IDS (Blue Team)
O motor **Suricata**, operando na interface DMZ com assinaturas *Emerging Threats*, inspecionou o tráfego em tempo real e disparou alertas críticos identificando a atividade maliciosa:

* **Indicadores de Comprometimento (IoCs) Identificados:**
  * **IP Atacante (Source):** `10.10.10.4`
  * **Ativo Alvo (Destination):** `192.168.20.10` (Porta 80/HTTP)
  * **Alertas Disparados:** `GPL ATTACK_RESPONSE id check returned root` / `ET SCAN`

<img width="1024" height="768" alt="image" src="https://github.com/user-attachments/assets/b6c0248e-f042-4faa-95b7-0f040d9b1da4" />

### Etapa 3.3: Contenção de Movimentação Lateral (Pivoting Bloqueado)
Simulando um cenário pós-exploração onde o atacante obteve acesso ao servidor da DMZ (`192.168.20.10`), tentou-se alcançar a estação interna na LAN (`192.168.10.50`). O firewall bloqueou 100% dos pacotes imediatamente, registrando a tentativa nos logs do sistema.

<img width="1024" height="768" alt="image" src="https://github.com/user-attachments/assets/1f4e1fcf-adc7-4c3d-9687-35a347a88253" />

---

## 🛡️ 4. Resposta a Incidentes e Prevenção Ativa (IPS)

Como ação de resposta imediata para neutralizar o ataque em andamento:
1. O Suricata foi alternado de modo **IDS** (apenas monitoramento) para **IPS** (*Block Offenders* + *Kill States* ativados).
2. Ao tentar executar novamente a exploração a partir do Kali Linux (`10.10.10.4`), o tráfego foi derrubado automaticamente em tempo real, impedindo novas requisições ao servidor web.

<img width="1280" height="780" alt="image" src="https://github.com/user-attachments/assets/d44f016d-afff-4ecd-a299-64c47a9d8d92" />

---

## 💡 5. Recomendações de Remediação e Lições Aprendidas

Em um ambiente corporativo de produção, as seguintes medidas complementares seriam recomendadas após este incidente:
* **Correção na Aplicação (Patching):** Implementar sanitização rigorosa de entradas no código da aplicação web para eliminar a falha de *Command Injection*.
* **Web Application Firewall (WAF):** Posicionar um WAF (como ModSecurity ou Cloudflare) na frente do servidor web para filtrar payloads maliciosos na camada de aplicação (Camada 7).
* **Centralização de Logs (SIEM):** Encaminhar os logs via *Syslog* do pfSense e do Suricata para um SIEM (como Wazuh ou Splunk) para correlação de eventos e automação de resposta (SOAR).

---

## 👨‍💻Estudante de Cibersegurança evoluindo 1% todos os dias. Aberto a dicas, feedbacks e conexões!
*Analista de Cibersegurança | SOC | Segurança de Redes e Firewall*  


