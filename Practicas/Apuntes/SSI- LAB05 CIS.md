# Laboratorio SSI 05: Auditoría y Hardening con Lynis + OpenSCAP (CIS Ubuntu)

## 1. Objetivos

- Realizar auditoría inicial con **Lynis** y obtener hardening index baseline.
    
- Implementar control **CIS 5.3.2.2** (pam_faillock) manualmente.
    
- Automatizar auditoría CIS Level 1 Server con **OpenSCAP**.
    
- Generar script de remediación automática y analizar resultados.
    

## 2. Auditoría Inicial - Lynis

## Instalación Lynis

bash

`sudo apt update sudo apt install lynis`

## Análisis inicial

bash

`sudo lynis audit system`

**Resultados iniciales** (pre-hardening):

text

`======================================== Lynis Security Audit for Linux 2.x ... [+] 1285 tests executed, 42 warnings, 85 suggestions Hardening index : 62 [##############----] ========================================`

**Ficheros generados**: `/var/log/lynis.log`, `/var/log/lynis-report.dat`

## 3. Implementación CIS 5.3.2.2 - pam_faillock

## Script de creación de perfil PAM

bash

`# /opt/5.3.2.2_CIS_ubuntu.sh #!/usr/bin/env bash arr=(     "Name: Enable pam_faillock to deny access"    "Default: yes"    "Priority: 0"    "Auth-Type: Primary"    "Auth:"    "        [default=die]                   pam_faillock.so authfail" ) printf '%s\n' "${arr[@]}" > /usr/share/pam-configs/faillock`

**Ejecución**:

bash

`chmod 740 /opt/5.3.2.2_CIS_ubuntu.sh /opt/5.3.2.2_CIS_ubuntu.sh pam-auth-update --enable faillock  # Warnings Perl normales (funciona)`

## Configuración faillock.conf

bash

`cat > /etc/security/faillock.conf << 'EOF' deny = 3 fail_interval = 900 unlock_time = 600 even_deny_root EOF`

**Verificación**:

bash

`grep faillock /etc/pam.d/common-auth # auth    required                                pam_faillock.so preauth # auth    [default=die]                          pam_faillock.so authfail`

## 4. Auditoría Automatizada - OpenSCAP

## Exploración perfiles disponibles

bash

`cd /opt oscap info scap-security-guide-0.1.79/ssg-ubuntu2404-ds.xml`

**Perfil seleccionado**: `xccdf_org.ssgproject.content_profile_cis_level1_server`

## Ejecución auditoría CIS Level 1

bash

`mkdir -p results reports remediacion 
oscap xccdf eval  --profile xccdf_org.ssgproject.content_profile_cis_level1_server   --results results/cis_level1_server_results.xml   --report reports/cis_level1_server_report.html 
scap-security-guide-0.1.79/ssg-ubuntu2404-ds.xml`

## Visualización resultados

bash

`# Opción 1: w3m (terminal) sudo apt install w3m w3m reports/cis_level1_server_report.html # Opción 2: Copia a Desktop usuario cp reports/cis_level1_server_report.html /home/ssiuser/Desktop/ chown ssiuser:ssiuser /home/ssiuser/Desktop/cis_level1_server_report.html`

## 5. Generación Script Remediación

bash

`oscap xccdf generate fix    --profile xccdf_org.ssgproject.content_profile_cis_level1_server   --fix-type bash   scap-security-guide-0.1.79/ssg-ubuntu2404-ds.xml >   /opt/remediacion/remediacion_cis_level1_server.sh`

**Características script**:

- ~1500-2000 líneas de Bash
    
- Un bloque `remediation` por cada regla CIS
    
- Formato comentado: `# remediation = { rule: '1.1.1.1' }`
    
Una vez generado eliminamos los que tenga que ver con AIDE ( se encarga de comprobar los hashes de todos los archivos de mi máquina por lo que tarda mucho)
Y volvemos a generar el lynis

## 6. Comandos Clave (Examen)

bash

`# Lynis sudo lynis audit system # PAM faillock pam-auth-update --enable faillock faillock --user usuario # OpenSCAP CIS L1 Server oscap xccdf eval --profile xccdf_org.ssgproject.content_profile_cis_level1_server \   --results results.xml --report report.html archivo.xml # Remediation Bash oscap xccdf generate fix --profile PERFIL --fix-type bash archivo.xml`

---

**Estado**: Auditoría baseline completa. 1er control CIS implementado. Script remediación generado. **Pendiente: análisis informe HTML y hardening iterativo.**