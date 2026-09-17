# Esquema de Ataque y Modelado de Amenazas



## 1. Narrativa del Escenario

Un atacante compromete inicialmente una estación de trabajo miembro del dominio (VM 108), realiza reconocimiento local para entender el entorno (usuarios, grupos, procesos, recursos compartidos, AV/firewall activos), y con los privilegios ya obtenidos en ese endpoint pivota directamente hacia el Controlador de Dominio (VM 109) — primero creando un servicio remoto, y luego generando una segunda conexión de red hacia el mismo destino disfrazada bajo un proceso con nombre de software legítimo.

**Alcance definido:** la cadena se cierra en Discovery + Movimiento Lateral. La extracción de credenciales (Credential Access) y la persistencia sobre el DC (Scheduled Task) quedaron fuera de alcance a propósito — el agente ya contaba con privilegios suficientes en VM108 para pivotar sin necesitar dumpear credenciales primero, así que ese paso no aporta valor de detección adicional a este escenario y no se ejecutó.



## 2. Matriz de Mapeo: Técnica, Táctica, VM y Caso de Uso

|Fase|VM Objetivo|Táctica ATT&CK|Técnica ATT&CK|Método de Ejecución|Caso de Uso|
|---|---|---|---|---|---|
|**A**|VM 108|Discovery (`TA0007`)|T1033, T1087.001, T1057, T1135, T1018, T1518, T1087.002, T1082|Comandos CLI (`whoami`, `net user`, `tasklist`, `net share`, `nltest`, `wmic`, `net group`, `netsh`) vía agente Sandcat/CALDERA|[C1](https://github.com/Maaurogp/SOC-DB-Lab/incident-reports/01-caso-uso-discovery-vm108.md)|
|**B**|VM 108 → VM 109|Lateral Movement (`TA0008`) / Persistence (`TA0003`)|T1021.002 (SMB/Windows Admin Shares) + T1543.003 (Create/Modify System Process: Windows Service)|Autenticación remota con privilegios ya elevados del agente + creación de servicio en el DC|[C2](https://github.com/Maaurogp/SOC-DB-Lab/02-lateral-movement-t1543-t1021.md)|
|**C**|VM 108 → VM 109|Lateral Movement (`TA0008`) / Defense Evasion (`TA0005`)|T1021.002 (SMB/Windows Admin Shares) + T1036 (Masquerading)|Conexión manual hacia el DC (puerto 135/epmap) desde un binario con nombre de software legítimo (`splunkd.exe`) ejecutándose desde una ruta pública|[C3](https://github.com/Maaurogp/SOC-DB-Lab/incident-reports/03-incident-report-sysmon-eid3-network-activity.md)|




## 3. Metodología de Detección y Validación

Cada una de las fases anteriores se audita mediante:

1. **Telemetría de Endpoint:** Sysmon (Event ID 1 para ejecución de procesos, Event ID 3 para conexiones de red, Event ID 4624/4648 para autenticación).
2. **Ingesta SIEM:** Wazuh Manager (VM 104) procesando los logs mediante reglas predeterminadas y reglas locales custom (`local_rules.xml`).
3. **Análisis de Brechas (Gap Analysis):** en los 3 casos se identificó que la regla que disparaba clasificaba el evento por debajo del umbral de alerta activa (nivel bajo pese a involucrar un activo crítico), lo que llevó a crear una regla local nueva (Caso 1) o a documentar el gap como hallazgo para trabajo futuro (Casos 2 y 3).
4. **Triage:** cada caso tiene su propia sección de Triage (Severidad × Criticidad × Confianza) dentro de su incident report, con la prioridad final justificada — no es un puntaje aislado, depende de si la detección está corroborada por una operación de CALDERA o fue ejecución manual sin esa segunda fuente (ver Caso 3).





