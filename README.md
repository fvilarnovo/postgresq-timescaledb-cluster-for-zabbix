# Clúster de PostgreSQL con TimescaleDB y Alta Disponibilidad para Zabbix

Este proyecto de Ansible despliega un clúster de alta disponibilidad de 3 nodos con PostgreSQL 15, la extensión TimescaleDB y Patroni. Está diseñado para proveer un único punto de entrada para aplicaciones como Zabbix, garantizando resiliencia y failover automático.

## Arquitectura

El clúster se compone de los siguientes elementos en cada uno de los 3 nodos:
* **PostgreSQL 15:** El motor de base de datos.
* **TimescaleDB:** Extensión para potenciar PostgreSQL con funcionalidades de series de tiempo.
* **Patroni:** Se encarga de la gestión del ciclo de vida de PostgreSQL y del failover automático.
* **Etcd:** Almacén de clave-valor distribuido que Patroni usa como DCS (Distributed Configuration Store) para mantener el estado del clúster.
* **Keepalived:** Gestiona una IP Virtual (VIP) que siempre apunta al nodo líder del clúster.
* **PgBouncer:** Pool de conexiones para gestionar eficientemente las conexiones a la base de datos.

## Requisitos Previos

1.  **Nodo de Control:** Una máquina (ej. tu Mac o un servidor Linux) con **Ansible** instalado.
2.  **Nodos Servidores:** 3 servidores con una distribución basada en RedHat como **Rocky Linux 9**.
3.  **Conectividad:** Acceso por SSH desde el nodo de control a los 3 servidores mediante clave pública.

## Instrucciones de Despliegue

### Paso 1: Clonar el Repositorio
```bash
git clone [https://github.com/fvilarnovo/postgresq-timescaledb-cluster-for-zabbix.git](https://github.com/fvilarnovo/postgresq-timescaledb-cluster-for-zabbix.git)
cd postgresq-timescaledb-cluster-for-zabbix
```

### Paso 2: Preparar los Nodos Servidores
En **cada uno de los 3 servidores**, ejecuta los siguientes comandos para crear un usuario para Ansible y darle permisos de `sudo` sin contraseña.
```bash
# Crear el usuario
sudo useradd ansible

# Asignarle una contraseña (la necesitarás una vez para copiar la clave SSH)
sudo passwd ansible

# Darle permisos de sudo sin contraseña
echo "ansible ALL=(ALL) NOPASSWD:ALL" | sudo tee /etc/sudoers.d/ansible
```
Desde tu **nodo de control**, copia tu clave SSH a cada uno de los servidores:
```bash
ssh-copy-id ansible@<IP_SERVIDOR_1>
ssh-copy-id ansible@<IP_SERVIDOR_2>
ssh-copy-id ansible@<IP_SERVIDOR_3>
```

### Paso 3: Configurar los Secretos
Este proyecto separa la configuración pública de los secretos.
1.  Copia el archivo de ejemplo de secretos:
    ```bash
    cp vars/secret.yml.example vars/secret.yml
    ```
2.  Edita el archivo `vars/secret.yml` y reemplaza los valores de ejemplo con tus contraseñas e IP virtual reales. **Este archivo es ignorado por Git y nunca debe ser público.**

### Paso 4: Configurar el Inventario
Edita el archivo `inventory` y reemplaza las IPs de ejemplo con las IPs reales de tus 3 servidores.

### Paso 5: Ejecutar el Playbook
Una vez configurado, ejecuta el playbook principal desde tu nodo de control:
```bash
ansible-playbook -i inventory deploy_pgcluster.yml
```

## Validación Post-Instalación

Una vez que el playbook termine, puedes validar el clúster de las siguientes maneras:

1.  **Conexión al Clúster:** Desde cualquier máquina que tenga acceso a la red, conéctate usando la IP virtual y el puerto de PgBouncer (`6432`).
    ```bash
    psql "host=<TU_IP_VIRTUAL> port=6432 dbname=postgres user=zabbix password=<TU_PASSWORD_ZABBIX>"
    ```

2.  **Verificar el Estado del Clúster:** Conéctate por SSH a cualquiera de los nodos y usa `patronictl` para ver el estado. (Puede que necesites crear primero el archivo `~/.config/patroni/patronictl.yml`).
    ```bash
    sudo patronictl list
    ```
    El resultado debería mostrar un nodo como `Leader` y los otros dos como `Replica`.

## Limpieza del Entorno
Para eliminar todos los componentes instalados por este playbook, puedes ejecutar el playbook de limpieza:
```bash
ansible-playbook -i inventory remove_cluster.yml
```
## Créditos
Este playbook es una versión modificada y adaptada del excelente trabajo original encontrado en el repositorio [vitabaks/postgresql-cluster](https://github.com/vitabaks/postgresql-cluster).

---

## License
Licensed under the MIT License. See the [LICENSE](./LICENSE) file for details.

## Author
Vitaliy Kukharik (PostgreSQL DBA) \
vitabaks@gmail.com

## Sponsor this project

Support our work through [Patreon](https://www.patreon.com/vitabaks):

[![Support me on Patreon](https://img.shields.io/endpoint.svg?url=https%3A%2F%2Fshieldsio-patreon.vercel.app%2Fapi%3Fusername%3Dvitabaks%26type%3Dpatrons&style=for-the-badge)](https://patreon.com/vitabaks)

Support our work through crypto wallet:

USDT (TRC20): `TSTSXZzqDCUDHDjZwCpuBkdukjuDZspwjj`

---

## Feedback, bug-reports, requests, ...
Are [welcome](https://github.com/vitabaks/postgresql_cluster/issues)!
