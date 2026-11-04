# 🚀 Proyecto SecNetDevOps - Infraestructura como Código (IaC)

Este repositorio contiene la Prueba de Concepto (PoC) del proyecto de Integración Curricular enfocado en la implementación de **SecNetDevOps, Integración/Despliegue Continuo (CI/CD) y Control de Versiones (Git)** aplicados a redes corporativas programáticas.

La infraestructura ha sido emulada en **EVE-NG** utilizando imágenes de Cisco IOS, automatizada de manera centralizada y *agentless* mediante **Ansible**, y versionada a través de **GitHub** utilizando flujos de autorización y revisión.

---

## 🏗️ Arquitectura de la Topología

La red emulada se divide en los siguientes segmentos lógicos y físicos:

*   **Sede Matriz:** Topología jerárquica de 3 capas (Core, Distribución y Acceso).
*   **Sede Sucursal:** Red remota conectada a través de un enlace WAN Point-to-Point (OSPF).
*   **Segmentación Lógica (VLANs):** Administración (10), Finanzas (20), RRHH (30), TI (40), Servidores (50), Sucursal (60), Desarrollo (100), Preproducción (200), Producción (300).
*   **Gestión Out-Of-Band (OOB):** Red aislada (VLAN 99 - `192.168.99.0/24`) dedicada exclusivamente para el tráfico de administración de Ansible (SSH).

## 🛠️ Tecnologías y Herramientas Utilizadas

*   **Emulación:** EVE-NG (Cisco IOSv L3, Cisco IOSvL2 L2, Alpine Linux QEMU).
*   **Control Node:** Ubuntu Server 22.04 LTS.
*   **Automatización:** Ansible Core (Colección `cisco.ios`), Python 3.
*   **Seguridad:** Ansible Vault (Cifrado de inventario y credenciales) y SSH RSA de 2048 bits.
*   **Versionamiento y CI/CD:** Git, GitHub, GitHub Actions.

---

## 📂 Estructura del Repositorio

El proyecto utiliza un enfoque modular basado en *Playbooks* declarativos:

| Archivo / Directorio | Descripción |
| :--- | :--- |
| `hosts` | Inventario centralizado agrupado por roles (Cifrado con Ansible Vault para proteger credenciales). |
| `ansible.cfg` | Configuración base del motor de Ansible. |
| `deploy_network.yml` | Playbook principal. Configura accesos, SVIs, troncales, OSPF, DHCP y seguridad de Capa 2 (Port-Security, STP BPDUGuard). |
| `rollback_network.yml` | Playbook quirúrgico para devolver la infraestructura a estado "Día 0" (limpieza idempotente de OSPF, SVIs y VLANs). |
| `backup_red.yml` | Extrae y respalda las configuraciones actuales de todos los equipos (`show run`). |
| `update_vlan11.yml` | Artefacto de prueba CI/CD: migra controladamente la VLAN 10 a la VLAN 11. |
| `deploy_telemetry.yml` | Configura el envío de eventos Syslog hacia el servidor central. |
| `.github/workflows/` | Contiene el pipeline de CI/CD (`ansible-ci.yml`) ejecutado por GitHub Actions. |

---

## 🔒 Seguridad por Diseño (SecNetDevOps)

Este proyecto implementa prácticas de seguridad desde el código base:
1. **Ansible Vault:** Las contraseñas de administración (`ansible_password`) no existen en texto plano en este repositorio. Se requiere la llave maestra (`vault_pass.txt`) local en el servidor de control para desencriptar el inventario en tiempo de ejecución.
2. **Exclusión de Secretos:** El archivo `.gitignore` bloquea preventivamente la subida de llaves `.pem`, respaldos locales y archivos temporales sensibles.
3. **Validación Estática:** El pipeline analiza la sintaxis y estructura estricta del YAML y Ansible antes de cualquier despliegue.

---

## 🔄 Flujo CI/CD y Gestión de Ramas

La promoción de código a la red de producción sigue un estricto flujo de gobernanza:

1. **Desarrollo:** Creación de una rama aislada (ej. `feature/actualizacion-vlan`).
2. **Integración Continua (CI):** Al subir los cambios, **GitHub Actions** ejecuta pruebas de *Linting* (`yamllint` y `ansible-lint`) y *Syntax Check*.
3. **Revisión:** Creación de un *Pull Request (PR)*. La rama `main` está protegida y exige que los *Status Checks* de Actions pasen en verde.
4. **Despliegue Continuo (CD):** Tras la aprobación y el *Merge*, el código final se extrae desde el servidor Ansible (`git pull`) y se inyecta en la red (`ansible-playbook`).

---

## 🚀 Instrucciones de Uso

> **Nota:** Se asume que la red OOB (Día 0) y el enrutamiento base del servidor Ubuntu ya están operativos.

**1. Clonar el repositorio:**
```bash
git clone https://github.com/TU_USUARIO/secnetdevops.git
cd secnetdevops
```

**2. Configurar la llave del Vault (Solo entorno local):**
Crea el archivo con la contraseña maestra para desencriptar el inventario (este archivo está excluido en el `.gitignore`).
```bash
echo "TU_CONTRASEÑA_VAULT" > vault_pass.txt
```

**3. Ejecutar el despliegue principal:**
```bash
ansible-playbook deploy_network.yml
```

**4. Ejecutar el rollback (en caso de fallas):**
```bash
ansible-playbook rollback_network.yml
```

---
*Proyecto desarrollado para demostrar la viabilidad técnica y operativa de flujos continuos en infraestructuras de red críticas.*