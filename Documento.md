# UNIVERSIDAD CENTROAMERICANA  
## “JOSÉ SIMEÓN CAÑAS”

### Departamento de Electrónica e Informática

#### Sistemas Operativos  
**Tarea Ex-aula 03**

**Docente:**  
Jaime Antonio García Santos  

**Integrantes:**  
- Aguilar Chavez, Gerardo Mauricio 00195721  
- Guidos Rodriguez, Jose Angel 00137321  
- Martínez Villtoro, Mario Antonio 00072520  
- Martínez Ventura, Gabriel Enrique 00389819  

San Salvador, Antiguo Cuscatlán, 27 de Noviembre de 2024

# OpenSUSE con Oracle Database

## Instalación del sistema operativo:

Para la presente tarea se optó por el uso de GCP (Google Cloud Platform), en la cual para instalarlo se siguieron los siguientes pasos:

1. Seleccionamos el menú en la parte superior izquierda.
2. Después seleccionamos el apartado de **Compute Engine** (nos pedirá agregar una extensión si no la tenemos instalada).
3. Seleccionamos la opción de crear una instancia, y se usaron los siguientes parámetros:
   - **Llaves SSH:** Se agregaron para trabajar de forma remota.
   - **IP estática:** Configurada desde el apartado “Red de VPC” y la opción de direcciones IP.

---

## Instalación de Oracle Database

### Paso 1: Instalación de dependencias
Al ingresar a la máquina virtual, instalamos las dependencias requeridas por Oracle Database:

```bash
sudo zypper install -y gcc gcc-c++ glibc glibc-devel libaio libaio-devel libstdc++ libstdc++-devel make binutils glibc-static
```

### Paso 2: Creación de grupos y usuario

1. Creamos los grupos para controlar la instalación y el acceso a la base de datos:

```bash
sudo groupadd oinstall  
sudo groupadd dba  
```

2. Creamos un usuario llamado `oracle` como propietario de la base de datos:

```bash
sudo useradd -m -g oinstall -G dba oracle  
```

3. Asignamos una contraseña al usuario:

```bash
sudo passwd oracle  
```

---

### Paso 3: Creación de directorios

Creamos los directorios necesarios para la instalación y les asignamos permisos:

```bash
sudo mkdir -p /u01/app/oracle/product/19c/dbhome_1  
sudo chown -R oracle:oinstall /u01  
sudo chmod -R 775 /u01  
```

---

### Paso 4: Configuración del kernel

Editamos el archivo `/etc/sysctl.conf` para configurar las variables del kernel con el siguiente contenido:

```bash
fs.file-max = 6815744  
kernel.sem = 250 32000 100 128  
kernel.shmmni = 4096  
kernel.shmall = 2097152  
kernel.shmmax = 4294967295  
net.ipv4.ip_local_port_range = 9000 65500  
net.core.rmem_default = 262144  
net.core.rmem_max = 4194304  
net.core.wmem_default = 262144  
net.core.wmem_max = 1048576  
```

Aplicamos la configuración con el comando:

```bash
sudo sysctl -p  
```

---

### Paso 5: Configuración de variables de entorno

Accedemos como usuario `oracle` y editamos el archivo `~/.bashrc` para agregar las siguientes variables de entorno:

```bash
export ORACLE_BASE=/u01/app/oracle  
export ORACLE_HOME=$ORACLE_BASE/product/19c/dbhome_1  
export ORACLE_SID=orcl  
export PATH=$ORACLE_HOME/bin:$PATH  
export LD_LIBRARY_PATH=$ORACLE_HOME/lib:/lib:/usr/lib  
export CLASSPATH=$ORACLE_HOME/jlib:$ORACLE_HOME/rdbms/jlib  
```

---

### Paso 6: Transferencia del archivo de instalación

Descargamos el archivo `.ZIP` proporcionado por Oracle desde su sitio web y lo transferimos a la máquina virtual usando el servicio SFTP de Termius.

---

### Paso 7: Descompresión del archivo

Descomprimimos el archivo en el directorio preparado con el comando:

```bash
unzip oracle-database-19c.zip -d /u01/app/oracle/product/19c/dbhome_1  
```
---

### Paso 8: Ejecución del instalador

Accedemos al directorio de instalación:

```bash
cd /u01/app/oracle/product/19c/dbhome_1  
```

Creamos un archivo de respuesta (`.rsp`) para especificar los parámetros de la instalación:

```bash
vim /u01/app/oracle/product/19c/dbhome_1/install/response  
```

Ejemplo de contenido del archivo:

```bash
ORACLE_BASE=/u01/app/oracle  
ORACLE_HOME=/u01/app/oracle/product/19c/dbhome_1  
oracle.install.db.Edition=SE  
oracle.install.db.OSDBA_GROUP=dba  
oracle.install.db.OSOPER_GROUP=oinstall  
oracle.install.db.OSBACKUPDBA_GROUP=dba  
oracle.install.db.OSDGDBA_GROUP=dba  
oracle.install.db.OSKMDBA_GROUP=dba  
oracle.install.db.OSRACDBA_GROUP=dba  
...
```

Ejecutamos el instalador en modo silencioso con el siguiente comando:

```bash
./runInstaller -silent -responseFile install/response/db_install.rsp  
```

---

### Paso 9: Comandos post-instalación

Tras finalizar la instalación, ejecutamos los comandos necesarios:

```bash
sudo /u01/app/oraInventory/orainstRoot.sh  
sudo /u01/app/oracle/product/19c/dbhome_1/root.sh  
```

---

### Paso 10: Verificación de la instalación

Desde el usuario `oracle`, verificamos la instalación con el comando:

```bash
$ORACLE_HOME/bin/sqlplus -v
```