
Laboratorio 4, realizado por Miguel Montana, Jeferson Hernandez y Yossed Riaño

#  Laboratorio 4 – Explorando el mundo de las Comunicaciones

##  Descripción general
Este proyecto documenta la configuración de un **switch Cisco Catalyst 2960**, la **instalación y uso de QEMU/KVM** para máquinas virtuales Linux, y el **uso de herramientas de red** como Nmap y Minicom.  
El objetivo principal es comprender la configuración de dispositivos físicos y virtuales para establecer una red funcional y segura.

---

## Objetivos
1. Configurar un switch Cisco Catalyst 2960 para comunicación básica entre PC, Raspberry Pi y Router ETM.  
2. Instalar y gestionar máquinas virtuales en Ubuntu mediante QEMU/KVM y Virt-Manager.  
3. Aplicar herramientas de diagnóstico y auditoría de red (Minicom, Ethtool, Nmap).

---

##  Materiales utilizados
- PC con **Ubuntu 20.04 / 22.04**
- **Switch Cisco Catalyst 2960**
- **Raspberry Pi**
- **Router ETM**
- **Cables Ethernet** (Cat 5e o superior)
- **Cable consola USB-Serial**
- **Software:**
  - Minicom  
  - Ethtool  
  - Nmap  
  - Net-tools  
  - QEMU / KVM / Virt-Manager  

---

##  Parte 1 – Configuración del Switch Cisco 2960

### Instalación y configuración de Minicom
```bash
sudo apt install -y minicom
sudo minicom -s
```

**Parámetros:**
- Baud Rate: 9600  
- Data Bits: 8  
- Stop Bits: 1  
- Parity: None  
- Flow Control: None  

Una vez conectado, el switch inicia el IOS y se obtiene acceso al modo EXEC.

### Verificación de puertos
```bash
show interfaces status
```

| Puerto | Dispositivo | Estado | Velocidad |
|---------|-------------|--------|-----------|
| Fa0/1 | Router G0/0/0 | connected | 100 Mb/s |
| Fa0/2 | PC-Windows | connected | 100 Mb/s |
| Fa0/3 | Raspberry Pi | connected | 100 Mb/s |

###  Configuración básica
```bash
enable secret Cisco123
line console 0
password Cisco123
login
service password-encryption
banner motd # ACCESO RESTRINGIDO AL LAB #
copy running-config startup-config
```

###  Verificación de enlace Ethernet
```bash
sudo apt install -y ethtool
sudo ethtool enp0s31f6 | grep "Link detected"
```

---

##  Parte 2 – Máquinas Virtuales con QEMU/KVM

###  Requisitos previos
- CPU con virtualización (VT-x / AMD-V)
- Usuario con permisos sudo
- Directorios de trabajo:
```bash
mkdir -p ~/vms/isos ~/vms/images
```

###  Instalación de herramientas
```bash
sudo apt update
sudo apt install -y qemu-kvm libvirt-daemon-system libvirt-clients virtinst virt-manager qemu-utils bridge-utils
sudo systemctl enable --now libvirtd
sudo usermod -aG libvirt,kvm $(whoami)
```

###  Descarga de ISOs
```bash
wget -O ~/vms/isos/ubuntu-server.iso "https://releases.ubuntu.com/22.04/ubuntu-22.04-live-server-amd64.iso"
wget -O ~/vms/isos/centos.iso "https://mirror.stream.centos.org/9-stream/.../CentOS-Stream-9-latest-x86_64-dvd1.iso"
wget -O ~/vms/isos/alpine.iso "https://dl-cdn.alpinelinux.org/alpine/v3.18/releases/x86_64/alpine-virt-3.18.0-x86_64.iso"
```

###  Creación de discos qcow2
```bash
qemu-img create -f qcow2 ~/vms/images/ubuntu22.qcow2 20G
qemu-img create -f qcow2 ~/vms/images/centos.qcow2 15G
qemu-img create -f qcow2 ~/vms/images/alpine.qcow2 5G
```

###  Instalación de VMs (ejemplos)

**Ubuntu 22.04**
```bash
qemu-system-x86_64 -m 4096 -smp 2 -cpu host -enable-kvm \
  -drive file=~/vms/images/ubuntu22.qcow2,format=qcow2 \
  -cdrom ~/vms/isos/ubuntu-server.iso -boot d \
  -netdev user,id=net0,hostfwd=tcp::2222-:22 -device e1000,netdev=net0 -vga virtio
```

**CentOS Stream**
```bash
qemu-system-x86_64 -m 4096 -smp 2 -enable-kvm \
  -drive file=~/vms/images/centos.qcow2,format=qcow2 \
  -cdrom ~/vms/isos/centos.iso -boot d \
  -netdev user,id=net0,hostfwd=tcp::2223-:22 -device e1000,netdev=net0
```

**Alpine Linux**
```bash
qemu-system-x86_64 -m 1024 -smp 1 -enable-kvm \
  -drive file=~/vms/images/alpine.qcow2,format=qcow2 \
  -cdrom ~/vms/isos/alpine.iso -boot d \
  -netdev user,id=net0,hostfwd=tcp::2224-:22 -device e1000,netdev=net0
```

---

##  Parte 3 – Inspección de Red con Nmap

**Escaneo básico de hosts**
```bash
nmap -sn 192.168.1.0/24
```

**Detección de servicios y versiones**
```bash
nmap -sV 192.168.1.10
```

**Escaneo completo**
```bash
nmap -A -T4 192.168.1.10
```

---

##  Conclusiones
- Se logró establecer comunicación serial y Ethernet con el switch Cisco 2960.  
- Se implementaron entornos virtualizados funcionales mediante **QEMU/KVM**.  
- Las herramientas **Nmap** y **Ethtool** permitieron verificar la conectividad y mapear servicios.  
- Este laboratorio sienta las bases para futuras pruebas de red y seguridad en entornos mixtos (físicos + virtuales).

---

##  Referencias
- [Documentación oficial de QEMU](https://www.qemu.org/documentation/)  
- [Virt-Manager User Guide](https://virt-manager.org/)  
- [Guía oficial de Nmap](https://nmap.org/book/)  
- [Cisco Catalyst 2960 Series Documentation](https://www.cisco.com/)

