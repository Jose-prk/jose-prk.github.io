---
title: Como instalar Arch Linux
description: Esta guía ha sido creada con el propósito de facilitar la instalación de Arch Linux. Casi toda la información recopilada en esta guía a sido obtenida de la wiki oficial de Arch Linux.
date: 2024-07-21 19:28:00 +0200
categories: [Linux]
tags: [Linux, Arch Linux, Avanzado]
pin: false
mermaid: true
image:
  path: /assets/img/posts/archbtw.png
  alt: Imagen creada por el usuario xyproto (Alexander F. Rødseth) bajo la licencia CC0 "No Rights Reserved" (https://bbs.archlinux.org/viewtopic.php?id=259604)
---

## Índice
0. [Introducción](#Introducción)
1. [Preinstalación](#Preinstalación)
    1. [Definir la distribución del teclado](#Definir-la-distribución-del-teclado)
    2. [Verificar la modalidad de arranque (BIOS, UEFI)](#Verificar-la-modalidad-de-arranque-BIOS-UEFI)
    3. [Conectarse a Internet](#Conectarse-a-Internet)
    4. [Sincronizar el reloj del sistema](#Sincronizar-el-reloj-del-sistema)
    5. [Particionar el disco](#Particionar-el-disco)
    6. [Montar las particiones](#Montar-las-particiones)
2. [Instalación](#Instalación)
3. [Configuración básica del sistema](#Configuración-básica-del-sistema)
	1. [Generamos el archivo fstab](#Generamos-el-archivo-fstab)
	2. [Chroot](#Chroot)
	3. [Zona horaria](#Zona-horaria)
	4. [Idioma del sistema](#Idioma-del-sistema)
	5. [Configurar la red](#Configurar-la-red)
	6. [Contraseña de root](#Contraseña-de-root)
	7. [Instalar el gestor de arranque](#Instalar-el-gestor-de-arranque)
	8. [Crear un usuario](#Crear-un-usuario)
	9. [Conectarse a Internet](#Conectarse-a-Internet)
	10. [Reiniciar](#Reiniciar)
4. [AUR (Arch User Repository)](#AUR-Arch-User-Repository)
	1. [Yay](#Yay)
	2. [Paru](#Paru)
5. [Instalar o crear un entorno de escritorio](#Instalar-o-crear-un-entorno-de-escritorio)
6. [Fuentes](#Fuentes)

## Introducción {#Introducción}

Esta guía ha sido creada con el propósito de facilitar la instalación de Arch Linux. Casi toda la información recopilada en esta guía a sido obtenida de la wiki oficial de Arch Linux. Aunque no es habitual, es posible que a la fecha en que este leyendo esta guía, algunos pasos del método de instalación hayan cambiado, por ello también recomiendo revisar las fuentes oficiales para obtener información mas actualizada o detallada. Podéis encontrar las fuentes haciendo clic [aquí](#Fuentes) o al final de esta guía.

## 1. Preinstalación {#Preinstalación}
### 1.1. Definir la distribución del teclado {#Definir-la-distribución-del-teclado}

Arch Linux viene por defecto con la distribución del teclado en inglés, por lo que para cambiarlo a español deberemos utilizar el siguiente comando:

```bash
loadkeys es
```

### 1.2. Verificar la modalidad de arranque (BIOS, UEFI) {#Verificar-la-modalidad-de-arranque-BIOS-UEFI}

Para comprobar el modo de arranque, liste el directorio efivars:

```bash
ls /sys/firmware/efi/efivars
```

Si la orden muestra el directorio sin error, el sistema se iniciará en modo UEFI. Si no existe el directorio, el sistema se iniciará en modo BIOS o en modalidad CSM (Compatibility Support Module).

### 1. 3. Conectarse a Internet {#Conectarse-a-Internet}

Para poder instalar arch necesitamos conexión a Internet. Esto podemos hacerlo de dos formas, conectando nuestro pc por cable o por wifi. Conectarnos a Internet por cable es la manera más rápida y sencilla, pero si esto no es posible, veremos como podemos conectarnos por wifi. Para ello utilizaremos el programa `iwd` (iNet Wireless Daemon).

1. Para ejecutar el programa y empezar a utilizar sus comandos escribiremos:

    ```bash
iwctl
```

2. Para listar todos los dispositivos wifi:

    ```bash
device list
```

3. Si nuestros dispositivo o adaptador están apagados, podemos encenderlos con uno de los siguientes comandos:

    ```bash
device "nombre" set-property Powered on
adapter "adaptador" set-property Powered on
```

4. Para buscar las redes disponibles (este comando no mostrará nada):

    ```bash
station "nombre del dispositivo" scan
```

5. Para ver la lista de todas las redes encontradas:

    ```bash
station "nombre del dispositivo" get-networks
```

6. Por último, para conectarnos a una red ejecutaremos el siguiente comando e introduciremos la contraseña de la red:

    ```bash
station "nombre" connect "SSID o nombre de la red"
```

7. Para comprobar que tenemos Internet, salimos del prompt de iwd con el comando `exit` y hacemos un ping a una dirección ip o dominio. Esto nos indicará si tenemos Internet mostrándonos la latencia (tiempo) de ida y vuelta de los paquetes de datos entre nuestro equipo y el servidor. Por ejemplo:

    ```bash
ping archlinux.org
```
		
	Nos mostrará algo así:

    ```bash
PING archlinux.org (95.217.163.246) 56(84) bytes of data.
64 bytes from archlinux.org (95.217.163.246): icmp_seq=1 ttl=51 time=55.2 ms
64 bytes from archlinux.org (95.217.163.246): icmp_seq=2 ttl=51 time=65.7 ms
64 bytes from archlinux.org (95.217.163.246): icmp_seq=3 ttl=51 time=187 ms
```

### 1. 4. Sincronizar el reloj del sistema {#Sincronizar-el-reloj-del-sistema}

Ahora que tenemos Internet, podemos sincronizar el reloj del sistema con el siguiente comando:

```bash
timedatectl set-ntp true
```

### 1. 5. Particionar el disco {#Particionar-el-disco}

1. Para crear las particiones podemos usar las herramientas `fdisk` o `cfdisk`. Yo recomiendo `cfdisk` puesto que tiene una interfaz más fácil de usar. 

	Al ejecutar cfdisk para crear las particiones saldrá un menú preguntando el tipo de tabla de particiones. Si estáis en máquina virtual deberéis seleccionar DOS (MBR o Master Boot Record), si estáis en máquina real con UEFI deberéis seleccionar GPT (GUID Partition Table) y si vuestro pc utiliza BIOS, deberéis seleccionar DOS (MBR o Master Boot Record).

	Si os habéis equivocado al seleccionar la tabla de particiones, podéis cambiar la tabla de particiones utilizando la herramienta `parted`. 
	
	**Para cambiar de GPT a DOS (MBR):**

    Seleccionamos el disco en el que queremos trabajar:

    ```bash
sudo parted /dev/sdX
```

    Cambiamos la tabla de particiones:

    ```bash
(parted) mklabel msdos
```
    Salimos del programa:

    ```bash
(parted) quit
```
		
   **Para cambiar de DOS (MBR) a GPT:**
		
	Seleccionamos el disco en el que queremos trabajar:

    ```bash
sudo parted /dev/sda
```

    Cambiamos la tabla de particiones:

    ```bash
(parted) mklabel gpt
```
    Salimos del programa:

    ```bash
(parted) quit
```


	Deberéis crear las particiones que consideréis necesarias, sin embargo lo mas habitual es crear tres si tenéis BIOS, o cuatro si tenéis UEFI. 
	
	1. Una partición para el sistema (directorio raíz `/`).
	2. Otra partición para el directorio personal del usuario (`/home`).
	3. Otra partición para el `swap` o área de intercambio.
	
		La partición swap es un espacio reservado en el disco duro que utiliza el sistema como memoria virtual cuando se queda sin espacio suficiente en la memoria RAM. 
			
		No existe un tamaño predefinido para esta partición, pero lo mas habitual suele ser asignarle mínimo la misma cantidad o el doble que de memoria RAM en caso de que esta sea muy pequeña, como las de 2GB o 4GB. O asignarle la misma cantidad o la mitad que de memoria RAM en caso de que esta sea mas grande, como las de 8GB o 16GB. Sin embargo si  la memoria es muy grande, como las 64GB, no tiene sentido asignarle 32GB al swap, ya que es demasiado grande y todo ese espacio no se usará  y se estará desperdiciando, por lo que en este sentido el máximo sería de 16GB.

		Al crear esta partición en cfdisk, deberéis indicarle que es de tipo `Linux Swap`. Al resto de particiones no hay que asignarle nada.

	4. Partición `efi`. En caso de que tengáis un ordenador con UEFI, también deberéis crear esta partición. Esta debe tener mínimo 300MB. Aquí es donde se almacenan los cargadores de arranque EFI y las aplicaciones y controladores que utiliza el firmware UEFI durante el arranque.

2. Para listar las particiones que hemos creado:

    ```bash
lsblk
```

3. Formateamos las particiones con el sistema de archivos correspondiente. En este caso, para las particiones raiz `/` y `/home` utilizaremos el formato ext4, para la partición del swap utilizaremos el formato swap y para la partición efi, utilizaremos FAT32:

    ```bash
mkfs.ext4 /dev/"partición raiz, por ejemplo sda1"
mkfs.ext4 /dev/"partición para home, por ejemplo sda2"
mkswap /dev/"partición swap"
mkfs.fat -F 32 /dev/"partición efi, por ejemplo sda3"
```

4. Para activar la partición swap o área de intercambio:

    ```bash
swapon /dev/"nombre de la partición swap"
```

### 1. 6. Montar las particiones {#Montar-las-particiones}

1. La partición raíz la montamos en /mnt:

    ```bash
mount /dev/"partición_raiz" /mnt
```

2. La partición home la montamos dentro de /mnt/home. Puesto que no existe este directorio, tendremos que crearlo:

    ```bash
mount --mkdir /dev/"partición_home" /mnt/home
```

3. Por último, si disponemos de parción efi, la montamos en /mnt/boot:

    ```bash
mount --mkdir /dev/"partición_efi" /mnt/boot
```

### 2. Instalación {#Instalación}

Ahora ya estamos listos para instalar los paquetes esenciales de arch en el directorio donde hemos montado el sistema, en `/mnt`. Para ello utilizamos el script pacstrap para instalar el paquete base, un Kernel de Linux y un firmware para hardware común:

```bash
pacstrap -K /mnt base linux linux-firmware
```

## 3. Configuración básica del sistema {#Configuración-básica-del-sistema}
### 3.1. Generamos el archivo fstab {#Generamos-el-archivo-fstab}

Fstab, o "File System Table", es un archivo de configuración en sistemas operativos basados en Unix (como Linux) que especifica cómo y dónde se deben montar los sistemas de archivos en el sistema durante el proceso de arranque. Contiene entradas que describen las particiones y dispositivos de almacenamiento que deben montarse, así como la ubicación donde se montarán y las opciones de montaje asociadas.

Generamos el  archivo fstab:

```bash
genfstab -U /mnt >> /mnt/etc/fstab
```

Podemos comprobar que el archivo se ha generado correctamente:

```bash
cat /mnt/etc/fstab
```

### 3.2 Chroot {#Chroot}

Cambiamos la raíz al nuevo sistema:

```bash
arch-chroot /mnt
```

### 3.3. Zona horaria {#Zona-horaria}

Definimos nuestra zona horaria:

```bash
ln -sf /usr/share/zoneinfo/"región"/"ciudad" /etc/localtime
```

Generamos el archivo /etc/adjtime:

```bash
hwclock --systohc
```

### 3.4. Idioma del sistema {#Idioma-del-sistema}

1. Accedemos a `/etc/locale.gen`  y descomentamos el locale correspondiente, por ejemplo en el caso de España sería `es_ES.UTF-8 UTF-8`. Además, también debemos descomentar el de `en_US.UTF-8 UTF-8`. Para ello necesitaremos instalar un editor de textos de terminal como vim o nano. Luego generamos los locales con el siguiente comando:

    ```bash
locale-gen
```

2. Creamos el archivo `locale.conf` en `/etc` y definimos la variable LANG. En el caso de España:

    ```bash
LANG=es_ES.UTF-8
```

3. Definimos el idioma del teclado. Para ello creamos el archivo `vconsole.conf` en `/etc` y escribimos dentro de el `KEYMAP=es`.

### 3.5. Configurar la red {#Configurar-la-red}

1. Creamos el archivo `/etc/hostname`  y en el escribimos el nombre del equipo.

2. Editamos el archivo `/etc/hosts` y escribimos lo siguiente:

    ```bash
127.0.0.1    localhost
::1          localhost
127.0.1.1    "nombredehost".localhost "nombredehost"
```

### 3.6. Contraseña de root {#Contraseña-de-root}

Para establecer una contraseña al usuario `root` ejecutamos el siguiente comando:

```bash
passwd
```

### 3.7. Instalar el gestor de arranque {#Instalar-el-gestor-de-arranque}

En Linux el gestor de arranque es el grub. Este debe instalarse de dos forma dependiendo de si tu pc es BIOS o UEFI. 

1. Para instalarlo y configurarlo en BIOS:

    ```bash
    pacman -S grub
    grub-install /dev/"disco"
    ```

	Utilizamos la herramienta `grub-mkconfig` para generar `/boot/grub/grub.cfg`:
	
    ```bash
    pacman -S grub
    grub-install /dev/"disco"
	grub-mkconfig -o /boot/grub/grub.cfg
    ```

2. Para instalarlo y configurarlo en UEFI:

    ```bash
    pacman -S grub efibootmgr
    grub-install --target=x86_64-efi --efidirectory=/boot
    ```

	Utilizamos la herramienta `grub-mkconfig` para generar `/boot/grub/grub.cfg`:

    ```bash
    grub-mkconfig -o /boot/grub/grub.cfg
    ```

3. Comprobar si hay otros sistemas operativos:

	Para que `grub-mkconfig` busque otros sistemas instalados y los agregue automáticamente al menú de arranque, debemos instalar el paquete `os-prober` y ejecutarlo. Luego deberemos volver a generar el archivo `grub.cfg`:

    ```bash
    pacman -S os-prober
    os-prober
    grub-mkconfig -o /boot/grub/grub.cfg
    ```

### 3.8. Crear un usuario {#Crear-un-usuario}

1. Creamos nuestro usuario:

    ```bash
	useradd -m "usuario"
    ```

2. Ponemos contraseña a nuestro usuario:

    ```bash
    passwd "usuario"
    ```

3. Añadimos nuestro usuario a los grupos:

    ```bash
    usermod -aG wheel,video,audio,storage "usuario"
    ```

4. Instalamos `sudo` y descomentamos la línea `%wheel ALL=(ALL:ALL) ALL` en el fichero `/etc/sudoers`.

### 3.9. Conectarse a Internet {#Conectarse-a-Internet}

Por último, antes de reiniciar y arrancar por primera vez el sistema, debemos instalar `networkmanager` y habilitarlo para tener Internet:

```bash
pacman -S networkmanger
systemctl enable NetworkManager
```

Si ya has reiniciado el sistema y no has completado este paso, no tendrás Internet. Para solucionarlo, basta con iniciar nuevamente desde el usb booteable o desde la ISO en máquina virtual y volver a montar las particiones y acceder a ellas con `arch-chroot` como vimos en los pasos anteriores. Una vez hecho esto, ya podremos instalar y habitlitar NetworkManager.
### 3.10. Reiniciar {#Reiniciar}

Para comprobar que el grub se ha instalado correctamente e iniciar por primera vez el sistema:

1. Salimos del entorno chroot:

    ```bash
    exit
    ```

2. Desmontamos las particiones:

    ```bash
    umount -R /mnt
    ```

3. Reiniciamos el sistema y desconectamos el usb booteable si lo estamos instalando en máquina real, o la ISO si estamos lo instalando en máquina virtual:

    ```bash
    reboot
    ```

### 4. AUR (Arch User Repository) {#AUR-Arch-User-Repository}

AUR (Arch User Repository) es un repositorio mantenido por la comunidad de usuarios de Arch Linux, el cual es usado tanto en Arch como en distribuciones derivadas, como Manjaro, ArcoLinux,  BlackArch, EndeavourOS o Garuda Linux entre otros. Para instalar paquetes de AUR es necesario instalar algún gestor de paquetes compatible, también conocidos como helpers, puesto que pacman solo funciona con los repositorios oficiales. Existen muchos gestores de paquetes para AUR, como yay, paru, trizen, pakaur, etc. Como es lógico, no es necesario instalarlos todos, basta con tener uno. En este caso solo vamos a ver como instalar los dos principales, yay y paru.
#### 4.1. Yay {#Yay}

1. Instalamos los paquetes necesarios:

    ```bash
    pacman -S --needed git base-devel
    ```

2. Clonamos el repositorio:

    ```bash
    git clone https://aur.archlinux.org/yay.git
    ```

3.  Accedemos a la carpeta del repositorio clonado, compilamos e instalamos el paquete:

    ```bash
    cd yay
    makepkg -si
    ```
#### 4.2. Paru {#Paru}

1. Instalamos los paquetes necesarios:

    ```bash
    sudo pacman -S --needed base-devel
    ```

2. Clonamos el repositorio:

    ```bash
    git clone https://aur.archlinux.org/paru.git
    ```

3.  Accedemos a la carpeta del repositorio clonado, compilamos e instalamos el paquete:

    ```bash
    cd paru
    makepkg -si
    ```

### 5. Instalar o crear un entorno de escritorio {#Instalar-o-crear-un-entorno-de-escritorio}

Llegados a este punto, ya hemos instalado y hecho la configuración básica de arch. Sin embargo, aun no tenemos una interfaz gráfica como es debida, ni audio, ni nada. Para ello tenemos que instalar un entorno de escritorio, que no es mas que un conjunto de programas como un servidor gráfico, un compositor de imágenes, un servidor de audio, un gestor de ventanas, un gestor de sesiones, un explorador de archivos, un emulador de terminal entre otras herramientas básicas que hacen posible que podamos usar el sistema de forma habitual.

Para esto existen dos opciones, instalar un entorno de escritorio ya hecho y configurado, o crear el nuestro instalando y configurando todas las herramientas necesarias. Puesto que crear un entorno de escritorio es algo mas complejo y largo de explicar, en esta parte solo nos vamos a centrar en como instalar uno ya hecho (Si queréis crear vuestro propio entorno de escritorio, podéis hacer clic [aquí](#)). Existen muchos entornos de escritorio, como Gnome, KDE, Cinnamon, xfce, Budgie, etc., pero explicarlos todos nos llevaría demasiado tiempo, por lo que únicamente vamos a centrarnos en los dos mas populares, Gnome y KDE, así que vamos a ver como instalarlos.

1. Instalar Gnome. Existen tres paquetes: 
	- **gnome:** Contiene el escritorio base de GNOME y un subconjunto de aplicaciones.
	- **gnome-extra:** Contiene aplicaciones adicionales, como un cliente de correo, un conjunto de juegos, etc.
	- **gnome-shell:** Es la interfaz de usuario básica de Gnome.

	Para instalar Gnome tendremos que instalar los paquetes de `gnome` y `gnome-shell` y opcionalmente podemos instalar también `gnome-extra`.

    ```bash
    sudo pacman -S gnome gnome-shell
    ```

	Al ejecutar este comando, nos mostrará una lista de paquetes del grupo `gnome`, aquí podemos elegir los que queramos instalar o pulsar enter para instalarlos todos por omisión.

2. Instalar KDE. Antes de instalar KDE es necesario instalar el servidor gráfico `Xorg`, que es una implementación del sistema de ventanas `X11`:

    ```bash
    sudo pacman -S xorg
    ```

	Nos mostrará una lista de paquetes del grupo `xorg`, podemos elegir cuales queremos instalar o pulsar enter para instalarlos todos por omisión. Lo mas recomendado es instalarlos todos para no tener problemas.

	Para instalar KDE, tenemos disponibles los siguientes paquetes:
	- **plasma-meta:** Instala el entorno de escritorio KDE Plasma.
	- **plasma-desktop:** Una instalación minimalista de KDE Plasma.
	- **kde-applications-meta:** Instala el conjunto completo de aplicaciones de KDE.

	Para instalar KDE, tendremos que instalar uno de los siguientes paquetes `plasma-meta` o `plasma-desktop` y opcionalmente podemos instalar también `kde-applications-meta` para obtener un escritorio mas completo con aplicaciones extra.
	
    ```bash
    sudo pacman -S plasma-meta kde-applications-meta
    ```

	De nuevo, al ejecutar este comando nos mostrará una lista de paquetes del grupo `plasma`, aquí podemos elegir los que queramos instalar o pulsar enter para instalarlos todos por omisión.

	Una vez instalado nuestro entorno de escritorio, ya solo nos queda reiniciar el ordenador para empezar a disfrutar de Arch Linux.

### Fuentes {#Fuentes}

- Guía de instalación:

	[https://wiki.archlinux.org/title/Installation_guide_(Espa%C3%B1ol)](https://wiki.archlinux.org/title/Installation_guide_(Espa%C3%B1ol))

- Conectarse al Wifi:

	[https://wiki.archlinux.org/title/Iwd#iwctl](https://wiki.archlinux.org/title/Iwd#iwctl)

- Configuración de red:

	[https://wiki.archlinux.org/title/Network_configuration_(Espa%C3%B1ol)](https://wiki.archlinux.org/title/Network_configuration_(Espa%C3%B1ol))

- Instalar el grub:

	[https://wiki.archlinux.org/title/GRUB_(Espa%C3%B1ol)](https://wiki.archlinux.org/title/GRUB_(Espa%C3%B1ol))

	[https://wiki.archlinux.org/title/Arch_boot_process_(Espa%C3%B1ol)#Gestor_de_arranque](https://wiki.archlinux.org/title/Arch_boot_process_(Espa%C3%B1ol)#Gestor_de_arranque)

- yay:

	[https://github.com/Jguer/yay?tab=readme-ov-file](https://github.com/Jguer/yay?tab=readme-ov-file)

- Paru:

	[https://github.com/Morganamilo/paru](https://github.com/Morganamilo/paru)

- Instalar Gnome:

	[https://wiki.archlinux.org/title/GNOME_(Espa%C3%B1ol)](https://wiki.archlinux.org/title/GNOME_(Espa%C3%B1ol))

- Instalar KDE:

	[https://wiki.archlinux.org/title/KDE_(Espa%C3%B1ol)](https://wiki.archlinux.org/title/KDE_(Espa%C3%B1ol))