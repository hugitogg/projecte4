# Gestió d'emmagatzematge redundant 

### Paso 1\. Preparar la máquina virtual
Abans d'iniciar Zorin OS, apaga la màquina virtual.

A VirtualBox:
1. Selecciona la màquina virtual Zorin OS.
2. Entra a Configuració.
3. Ves a Emmagatzematge.
4. Afegeix dos discs durs virtuals.
5. Selecciona VDI.
6. Selecciona mida dinàmica.
7. Assigna 10 GB a cada disc.

![img1](/Tasca%204/img/lv1.png)

### Paso 2\. Comprobar els discos
Un cop iniciat Zorin OS, hem obert el terminal i hem utilitzat l'ordre lsblk per comprovar que els dos discos nous estaven connectats correctament.

Aquesta ordre permet visualitzar els dispositius d'emmagatzematge del sistema.

![img2](/Tasca%204/img/lv2.png)

### Paso 3\. Instalar LVM  
 Hem instal·lat el paquet LVM2 mitjançant el terminal. Aquesta eina permet agrupar diversos discos físics i gestionar l'espai d'emmagatzematge de manera flexible.

Ordres utilitzades:

sudo apt update

sudo apt install lvm2 -y

![img3](/Tasca%204/img/lv3.png)

![img4](/Tasca%204/img/lv4.png)

### Paso 4\. Creació del grup de volums (VG)
Hem preparat els dos discos físics per utilitzar-los amb LVM i posteriorment els hem agrupat en un únic grup de volums anomenat vg_principal.

Això permet gestionar l'espai dels dos discos com un conjunt.

Ordres utilitzades:

sudo pvcreate /dev/sdb /dev/sdc

sudo vgcreate vg_principal /dev/sdb /dev/sdc

sudo vgs

![img5](/Tasca%204/img/lv5.png)

![img6](/Tasca%204/img/lv6.png)

![img7](/Tasca%204/img/lv7.png)

### Paso 5\. Creació del volum lògic (LV) 
Dins del grup de volums hem creat un volum lògic de 10 GB anomenat lv_principal.

Després l'hem formatat amb el sistema de fitxers ext4 i l'hem muntat a la carpeta /mnt/principal.

Ordres utilitzades:

sudo lvcreate -L 10G -n lv_principal vg_principal

sudo mkfs.ext4 /dev/vg_principal/lv_principal

sudo mkdir -p /mnt/principal

sudo mount /dev/vg_principal/lv_principal /mnt/principal

![img8](/Tasca%204/img/lv8.png)

![img9](/Tasca%204/img/lv9.png)

![img10](/Tasca%204/img/lv10.png)

![img11](/Tasca%204/img/lv11.png)

![img12](/Tasca%204/img/lv12.png)

### Paso 6\. Configuració del muntatge automàtic (/etc/fstab)
Hem modificat el fitxer /etc/fstab per configurar el muntatge automàtic del volum lògic cada vegada que s'inicia el sistema.

Hem consultat l'UUID del volum i l'hem afegit al fitxer de configuració.

Ordres utilitzades:

sudo blkid /dev/vg_principal/lv_principal

sudo nano /etc/fstab

sudo mount -a

![img13](/Tasca%204/img/lv13.png)

![img14](/Tasca%204/img/lv14.png)

![img15](/Tasca%204/img/lv15.png)

### Paso 7\. Configuració del mirall (Mirror) 
Hem creat un volum amb configuració RAID 1 per mantenir dues còpies de la informació en discos diferents.

Aquesta configuració proporciona redundància i permet continuar disposant de les dades si un dels discos falla.

Ordres utilitzades:

sudo lvcreate --type raid1 -m 1 -L 4G -n lv_espejo vg_principal

sudo lvs -a -o +devices
![img16](/Tasca%204/img/lv16.png)

![img17](/Tasca%204/img/lv17.png)

![img18](/Tasca%204/img/lv18.png)

### Paso 8\. Ampliació del grup de volums
Hem afegit dos discos virtuals nous de 10 GB a la màquina virtual.

Després els hem preparat i incorporat al grup de volums existent mitjançant l'ordre vgextend.

Això permet disposar de més espai per crear nous volums lògics.

Ordres utilitzades:

sudo pvcreate /dev/sdd /dev/sde

sudo vgextend vg_principal /dev/sdd

sudo vgextend vg_principal /dev/sde

![img19](/Tasca%204/img/lv19.png)

![img20](/Tasca%204/img/lv20.png)

![img21](/Tasca%204/img/lv21.png) 
![img22](/Tasca%204/img/lv22.png)

### Paso 9\. Creació del volum de dades
Hem creat un nou volum lògic anomenat lv_dades de 10 GB utilitzant el primer dels discos afegits.

Després l'hem formatat amb ext4 i l'hem muntat a /mnt/dades.

Finalment, hem creat un fitxer de prova per comprovar que el volum permet emmagatzemar informació.

Ordres utilitzades:

sudo lvcreate -L 10G -n lv_dades vg_principal /dev/sdd

sudo mkfs.ext4 /dev/vg_principal/lv_dades

sudo mkdir -p /mnt/dades

sudo mount /dev/vg_principal/lv_dades /mnt/dades

![img23](/Tasca%204/img/lv23.png)  
![img24](/Tasca%204/img/lv24.png)

![img25](/Tasca%204/img/lv25.png)

![img26](/Tasca%204/img/lv26.png)

### Paso 10\. Creació d'una instantània (Snapshot)
Hem creat una instantània del volum lv_dades amb el nom lv_snapshot.

Aquesta instantània permet conservar l'estat del volum en un moment concret i facilita la recuperació de dades davant possibles modificacions o errors.

Ordres utilitzades:

sudo lvcreate -L 5G -s -n lv_snapshot vg_principal/lv_dades

sudo lvs

![img27](/Tasca%204/img/lv27.png)

![img28](/Tasca%204/img/lv28.png)

![img29](/Tasca%204/img/lv29.png)
![img30](/Tasca%204/img/lv30.png)
