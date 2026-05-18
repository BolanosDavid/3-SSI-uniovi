## Resumen Ejecutivo

**Objetivo**: Desplegar Wazuh single-node con Docker para simular ciberataques en lab SSI.  
**Agente**: 'ApacheServer-Prueba' (Ubuntu x64, localhost:127.0.0.1).  
**Estado**: Dashboard funcional, agente activo, FIM/CIS-CAT configurados, Apache reactivado, eventos auth/web detectados.

## Infraestructura Desplegada

text

`Single-Node Wazuh (Docker Compose oficial v4.11+) ├── wazuh-manager (SIEM core) ├── wazuh-indexer (Elasticsearch) ├── wazuh-dashboard (Kibana → https://localhost:443 SSL) └── Agente Ubuntu: ApacheServer-Prueba (DEB amd64 v4.14.1)`

**Acceso**: [https://localhost](https://localhost/) (admin/SecretPassword, self-signed SSL).

## Configuraciones Realizadas

## 1. Instalación Agente

bash

`wget https://packages.wazuh.com/4.x/apt/pool/main/w/wazuh-agent/wazuh-agent_4.14.1-1_amd64.deb sudo WAZUH_MANAGER='127.0.0.1' WAZUH_AGENT_NAME='ApacheServer-Prueba' dpkg -i wazuh-agent_*.deb sudo systemctl daemon-reload && sudo systemctl enable --now wazuh-agent`

## 2. FIM (File Integrity Monitoring)

**/var/ossec/etc/ossec.conf**:

xml

`<syscheck>   <directories realtime="yes" check_all="yes">/var/www/html,/etc/apache2</directories> </syscheck>`

## 3. CIS-CAT/SCA (Daily Audits)

Habilitado con `interval="24h"` para informes compliance diarios.

## 4. Apache Reactivado (Post-Remediación)

bash

`sudo apt install apache2 sudo systemctl unmask apache2 sudo systemctl enable apache2 sudo systemctl start apache2 curl -X DELETE http://localhost  # Test ataque web`

## Eventos Detectados (Threat Hunting)

|Evento|Trigger|Rule ID Esperada|Estado|
|---|---|---|---|
|User creation|`sudo useradd -m testuser`|~60109|✅ Detectado|
|Failed sudo|`su - testuser` (password débil)|5501/5502|✅ Detectado|
|Web attack|`curl -X DELETE http://localhost`|~31100|✅ Pendiente confirmación|
|FIM changes|Modifs /var/www|Realtime|Listo|


## Logs Útiles

bash

`# Agente tail -f /var/ossec/logs/alerts/alerts.log /var/ossec/bin/wazuh-logtest # Apache tail -f /var/log/apache2/{access,error}.log sudo systemctl status apache2 # Docker Wazuh docker-compose logs -f wazuh-manager`