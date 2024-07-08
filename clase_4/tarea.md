Eres un DevOps Engineer que recibio una alerta de que una aplicación crítica en una instancia EC2 (en AWS) está experimentando problemas de rendimiento y accesibilidad. Los usuarios han reportado que la aplicación Apache 2 está respondiendo lentamente y, en algunos casos, no está accesible.

### Para realizar este ejercicio, tenes que instalar Apache 2 en tu Multipass

sudo apt-get install apache2

Tarea:

#### Paso 1: Verificar el Estado del Sistema de Archivos y Dispositivos

- Lista los dispositivos de bloque y asegúrate de que todos los discos necesarios están disponibles y no presentan problemas.

  llsblk -o NAME,FSTYPE,SIZE,MOUNTPOINT,LABEL,UUID

  Resultado:
  NAME FSTYPE SIZE MOUNTPOINT LABEL UUID
  loop0 squashfs 74.2M /snap/core22/1380  
   loop3 squashfs 38.8M /snap/snapd/21759  
   sda 931.5G  
   └─sda1 vfat 931.3G /media/devmon/Plex-Drive Plex-Drive 58C9-A0A4
  sdb 223.6G  
   ├─sdb1 1M  
   ├─sdb2 ext4 2G /boot 085c379a-2aad-4a47-9b84-c5a042010c85
  └─sdb3 LVM2_member 221.6G lc9nhp-tFYX-LMXa-JIDT-PIeC-fueq-zmVuPP
  └─ubuntu--vg-ubuntu--lv ext4 100G / f6480343-e8f7-45b7-9922-6d953144cdf0

- Lista los dispositivos de carácter y confirma que no hay problemas con los dispositivos de entrada/salida.

  ll /dev/input/ | grep -i "input\|error\|fail\|tty\|usb"

  Resultado:
  crw-rw---- 1 root input 13, 64 Jul 7 00:40 event0
  crw-rw---- 1 root input 13, 65 Jul 7 00:40 event1
  crw-rw---- 1 root input 13, 66 Jul 7 00:40 event2
  crw-rw---- 1 root input 13, 67 Jul 7 00:40 event3
  crw-rw---- 1 root input 13, 68 Jul 7 00:40 event4
  crw-rw---- 1 root input 13, 69 Jul 7 00:40 event5
  crw-rw---- 1 root input 13, 70 Jul 7 00:40 event6
  crw-rw---- 1 root input 13, 71 Jul 7 00:40 event7
  crw-rw---- 1 root input 13, 63 Jul 7 00:40 mice

- Lista todas las particiones y asegúrate de que no hay particiones llenas o mal configuradas.

  sudo fdisk -l

  Resultado:
  Disk /dev/sda: 931.51 GiB, 1000204886016 bytes, 1953525168 sectors
  Disk model: External USB 3.0
  Units: sectors of 1 \* 512 = 512 bytes
  Sector size (logical/physical): 512 bytes / 512 bytes
  I/O size (minimum/optimal): 512 bytes / 512 bytes
  Disklabel type: dos
  Disk identifier: 0xaf839b49

  Device Boot Start End Sectors Size Id Type
  /dev/sda1 514080 1953520064 1953005985 931.3G c W95 FAT32 (LBA)

- Revisa la información de montajes para asegurarte de que todas las particiones necesarias están montadas correctamente.

  df -h

  Resultado:
  Filesystem Size Used Avail Use% Mounted on
  tmpfs 392M 3.5M 388M 1% /run
  /dev/mapper/ubuntu--vg-ubuntu--lv 98G 23G 71G 25% /
  tmpfs 2.0G 0 2.0G 0% /dev/shm
  tmpfs 5.0M 0 5.0M 0% /run/lock
  /dev/sdb2 2.0G 253M 1.6G 14% /boot
  tmpfs 392M 4.0K 392M 1% /run/user/1000

#### Paso 2: Verificar el Rendimiento del Sistema

- Revisa la información del CPU para entender mejor la capacidad del sistema.

lscpu

Resultado:
Architecture: x86_64
CPU op-mode(s): 32-bit, 64-bit
Address sizes: 36 bits physical, 48 bits virtual
Byte Order: Little Endian
CPU(s): 4
On-line CPU(s) list: 0-3
Vendor ID: GenuineIntel
Model name: Intel(R) Atom(TM) CPU D525 @ 1.80GHz
CPU family: 6
Model: 28
Thread(s) per core: 2
Core(s) per socket: 2
Socket(s): 1
Stepping: 10
BogoMIPS: 3590.99
Flags: fpu vme de pse tsc msr pae mce cx8 apic sep mtrr pge mca cmov pat pse36 clflush dts acpi mmx fxsr sse sse2 ss ht tm pbe sys
call nx lm constant_tsc arch_perfmon pebs bts nopl cpuid aperfmperf pni dtes64 monitor ds_cpl tm2 ssse3 cx16 xtpr pdcm movb
e lahf_lm dtherm
Caches (sum of all):  
 L1d: 48 KiB (2 instances)
L1i: 64 KiB (2 instances)
L2: 1 MiB (2 instances)
NUMA:  
 NUMA node(s): 1
NUMA node0 CPU(s): 0-3
Vulnerabilities:  
 Gather data sampling: Not affected
Itlb multihit: Not affected
L1tf: Not affected
Mds: Not affected
Meltdown: Not affected
Mmio stale data: Not affected
Retbleed: Not affected
Spec rstack overflow: Not affected
Spec store bypass: Not affected
Spectre v1: Not affected
Spectre v2: Not affected
Srbds: Not affected
Tsx async abort: Not affected

- Analiza la información de la memoria para identificar posibles cuellos de botella.

  lsmem

  Resultado:
  RANGE SIZE STATE REMOVABLE BLOCK
  0x0000000000000000-0x00000000bfffffff 3G online yes 0-23
  0x0000000100000000-0x000000013fffffff 1G online yes 32-39

  Memory block size: 128M
  Total online memory: 4G
  Total offline memory: 0B

- Analiza los últimos 20 registros del log del sistema para confirmar que no haya nada mal con el SO.

journalctl -n 20 | cat > lastLogSys.txt

less lastLogSys.txt

Resultado:
Jul 08 21:04:42 dvr-server casaos-message-bus[963]: {"time":"2024-07-08T21:04:42.078638181Z","id":"","remote_ip":"127.0.0.1","host":"127.0.0.1:459>
Jul 08 21:04:47 dvr-server casaos[988]: /bin/bash -c source /usr/share/casaos/shell/helper.sh ;CatNetCardState ens35
Jul 08 21:04:47 dvr-server casaos[988]: /bin/bash -c source /usr/share/casaos/shell/helper.sh ;CatNetCardState wls34
Jul 08 21:04:47 dvr-server casaos-message-bus[963]: {"time":"2024-07-08T21:04:47.078553245Z","id":"","remote_ip":"127.0.0.1","host":"127.0.0.1:459>
Jul 08 21:04:52 dvr-server casaos[988]: /bin/bash -c source /usr/share/casaos/shell/helper.sh ;CatNetCardState ens35
Jul 08 21:04:52 dvr-server casaos[988]: /bin/bash -c source /usr/share/casaos/shell/helper.sh ;CatNetCardState wls34
Jul 08 21:04:52 dvr-server casaos-message-bus[963]: {"time":"2024-07-08T21:04:52.09582078Z","id":"","remote_ip":"127.0.0.1","host":"127.0.0.1:4595>
Jul 08 21:04:52 dvr-server casaos-app-management[2869]: 2024-07-08T21:04:52.249Z error main app image not match for local app and st>
Jul 08 21:04:52 dvr-server casaos-app-management[2869]: 2024-07-08T21:04:52.299Z info main apps of local app and store app have diff>
Jul 08 21:04:52 dvr-server casaos-app-management[2869]: 2024-07-08T21:04:52.316Z info main apps of local app and store app have diff>
Jul 08 21:04:52 dvr-server casaos-app-management[2869]: {"time":"2024-07-08T21:04:52.447200525Z","id":"","remote_ip":"192.168.18.108","host":"192.>
Jul 08 21:04:52 dvr-server casaos-user-service[998]: [GIN] 2024/07/08 - 21:04:52 | 200 | 3.458246ms | 192.168.18.108 | GET "/v1/users/cur>
Jul 08 21:04:52 dvr-server casaos-user-service[998]: [GIN] 2024/07/08 - 21:04:52 | 200 | 3.919415ms | 192.168.18.108 | GET "/v1/users/cur>
Jul 08 21:04:57 dvr-server casaos[988]: /bin/bash -c source /usr/share/casaos/shell/helper.sh ;CatNetCardState ens35
Jul 08 21:04:57 dvr-server casaos[988]: /bin/bash -c source /usr/share/casaos/shell/helper.sh ;CatNetCardState wls34
Jul 08 21:04:57 dvr-server casaos-message-bus[963]: {"time":"2024-07-08T21:04:57.07845332Z","id":"","remote_ip":"127.0.0.1","host":"127.0.0.1:4595>
Jul 08 21:05:00 dvr-server casaos[988]: 消息来了，message：{"type":"ping"}
Jul 08 21:05:02 dvr-server casaos[988]: /bin/bash -c source /usr/share/casaos/shell/helper.sh ;CatNetCardState ens35
Jul 08 21:05:02 dvr-server casaos[988]: /bin/bash -c source /usr/share/casaos/shell/helper.sh ;CatNetCardState wls34
Jul 08 21:05:02 dvr-server casaos-message-bus[963]: {"time":"2024-07-08T21:05:02.078259744Z","id":"","remote_ip":"127.0.0.1","host":"127.0.0.1:459>

#### Paso 3: Verificación y Reinicio de Servicios

- Verificar el estado del servicio Apache

  systemctl status apache2 | cat > logApacheSys.txt

  Resultado:
  ● apache2.service - The Apache HTTP Server
  Loaded: loaded (/usr/lib/systemd/system/apache2.service; enabled; preset: enabled)
  Active: active (running) since Wed 2024-07-03 17:45:41 CST; 4 days ago
  Docs: https://httpd.apache.org/docs/2.4/
  Process: 13307 ExecReload=/usr/sbin/apachectl graceful (code=exited, status=0/SUCCESS)
  Main PID: 3864 (apache2)
  Tasks: 55 (limit: 1060)
  Memory: 6.3M (peak: 8.4M)
  CPU: 2.930s
  CGroup: /system.slice/apache2.service
  ├─ 3864 /usr/sbin/apache2 -k start
  ├─13311 /usr/sbin/apache2 -k start
  └─13312 /usr/sbin/apache2 -k start

  Jul 03 17:45:41 virtual1 systemd[1]: Started apache2.service - The Apache HTTP Server.
  Jul 06 13:30:51 virtual1 systemd[1]: Reloading apache2.service - The Apache HTTP Server...
  Jul 06 13:30:51 virtual1 apachectl[10498]: AH00558: apache2: Could not reliably determine the server's fully qualified domain name, using 127.0.1.1. Set the 'ServerName' directive globally to suppress this message
  Jul 06 13:30:51 virtual1 systemd[1]: Reloaded apache2.service - The Apache HTTP Server.
  Jul 07 00:10:42 virtual1 systemd[1]: Reloading apache2.service - The Apache HTTP Server...
  Jul 07 00:10:42 virtual1 apachectl[11704]: AH00558: apache2: Could not reliably determine the server's fully qualified domain name, using 127.0.1.1. Set the 'ServerName' directive globally to suppress this message
  Jul 07 00:10:42 virtual1 systemd[1]: Reloaded apache2.service - The Apache HTTP Server.
  Jul 08 00:04:21 virtual1 systemd[1]: Reloading apache2.service - The Apache HTTP Server...
  Jul 08 00:04:21 virtual1 apachectl[13310]: AH00558: apache2: Could not reliably determine the server's fully qualified domain name, using 127.0.1.1. Set the 'ServerName' directive globally to suppress this message
  Jul 08 00:04:21 virtual1 systemd[1]: Reloaded apache2.service - The Apache HTTP Server.

- Reiniciar el servicio Apache
  systemctl restart apache2

- Verificar los logs en /var/log/apache2/error.log para ver si todo funciona bien

  cat /var/log/apache2/error.log | cat > logApacheError.txt

  Resultado:
  [Mon Jul 08 00:04:21.432652 2024] [mpm_event:notice] [pid 3864:tid 270571720282144] AH00489: Apache/2.4.58 (Ubuntu) configured -- resuming normal operations
  [Mon Jul 08 00:04:21.432665 2024] [core:notice] [pid 3864:tid 270571720282144] AH00094: Command line: '/usr/sbin/apache2'
  logApacheError.txt (END)
