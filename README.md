# Homelab SOC — Detección y Threat Hunting en Active Directory

He creado un Homelab de ciberseguridad orientado a Blue Team, construido en Proxmox VE con segmentación de red por VLANs (pfSense), un SIEM (Wazuh + Sysmon) y ejercicios de emulación de adversario (CALDERA) contra un Active Directory real on-premise.

***El proyecto está diseñado para ser escalable*** 

Actualmente, hoy cubre **detección y triage** básico orientado a un rol de SOC Analyst N1, con una arquitectura pensada para tener experiencia lo más cercana posible a un entorno real (case management, threat intelligence, más escenarios de emulación) para entender como funcionan los servicios utilizados, su despliegue, su accionar, a fines de poder observar y emular casos concretos.

A medida que avanzo en mi carrera, el objetivo es seguir con el desarrollo del proyecto ampliando su scope (alcance).



## Qué incluye

- Arquitectura segmentada en 4 VLANs (SOC Core, DMZ/Targets, Active Directory,
  Microservicios/Web) enrutadas mediante pfSense.
- SIEM Wazuh con agentes desplegados en Linux, Windows Server (Domain
  Controller) y Windows Client, con telemetría enriquecida por Sysmon.
- Targets vulnerables (Metasploitable2, DVWA, WebGoat, bWAPP) para práctica
  ofensiva/defensiva.
- Emulación de adversario con CALDERA (MITRE ATT&CK) contra el Domain
  Controller, con análisis de gap de detección documentado.



## Casos de uso documentados

1. **[Reconocimiento local y Discovery en host Windows 10 (TA0007)](https://github.com/Maaurogp/SOC-DB-Lab/blob/main/incident-reports/01-caso-uso-discovery-vm108.md)** — Enumeración de usuarios, grupos, procesos y recursos compartidos vía CALDERA sobre VM108. Gap encontrado: comandos LOLBin (`whoami`, `net user`, `tasklist`) sin cobertura por defecto — se creó la regla local `100012` (nivel 8) para cerrarlo.
2. **[Movimiento lateral vía creación de servicios (T1543.003 / T1021.002)](https://github.com/Maaurogp/SOC-DB-Lab/blob/main/incident-reports/02-lateral-movement-t1543-t1021.md)** — Pivote desde VM108 hacia el Domain Controller (VM109) mediante logon remoto tipo 3 con privilegios elevados. Gap encontrado: el evento quedó por debajo del umbral de alerta (nivel 3) pese a impactar un activo crítico — Triage: **P3**.
3. **[Actividad de red sospechosa — Movimiento lateral vía SMB/Admin Shares (T1021.002)](https://github.com/Maaurogp/SOC-DB-Lab/blob/main/incident-reports/03-incident-report-sysmon-eid3-network-activity.md)** — Conexión de VM108 hacia el DC (puerto 135/epmap) generada por un proceso con nombre de masquerading (`splunkd.exe` ejecutándose desde una ruta pública).

Ver la carpeta completa: [incident-reports/](https://github.com/Maaurogp/SOC-DB-Lab/tree/main/incident-reports)

> A medida que el proyecto continúe avanzando los nuevos casos serán adjuntados.



## Documentación técnica completa

Ver [docs/arquitectura-documentacion-tecnica.md](https://github.com/Maaurogp/SOC-DB-Lab/blob/main/docs/arquitectura-documentacion-tecnica.md) para el detalle completo de hardware, topología de red, inventario de VMs y estado de agentes.



## Stack técnico

- **Proxmox VE** — hypervisor que corre las 10 VMs del laboratorio sobre una sola máquina física (bare-metal).
- **pfSense** — router/firewall que segmenta y enruta el tráfico entre las 4 VLANs del lab.
- **Wazuh** — SIEM central: recibe la telemetría de los agentes y genera las alertas que se analizan en cada caso de uso.
- **Sysmon** — agente en los endpoints Windows que enriquece los logs nativos (creación de procesos, conexiones de red) antes de que lleguen a Wazuh.
- **CALDERA** — emula técnicas de MITRE ATT&CK contra el Domain Controller para medir qué detecta el SIEM y qué no.
- **MITRE ATT&CK** — framework usado para mapear cada técnica ejecutada y estructurar el análisis de gaps.
- **Active Directory (Windows Server)** — entorno de dominio real (DC + cliente Windows) que funciona como objetivo de los escenarios de ataque.



## Sobre mí

Mauro González — Soy una persona que tiene una pasión y una incansable curiosidad por la ciberseguridad, actualmente orientando mi desarrollo tanto personal como laboral, con el objetivo final de poder desarrollarme como SOC Analyst N1. 

Construí este laboratorio de forma autodidacta para aprender y desarrollar habilidades prácticas en detección, análisis de amenazas y triage, lo más cercano posible a un entorno real. 

📎 [LinkedIn](https://linkedin.com/in/maurogp/) · [GitHub](https://github.com/Maaurogp/) · maurogp@proton.me
