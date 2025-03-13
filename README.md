## **Partie 1 : Learn**

### **1. Anatomy of a Program**

#### **A. Commande `file`**
La commande `file` permet de déterminer le type d'un fichier en analysant son contenu.

**Commandes utilisées :**
```bash
file /bin/ls
file /bin/ip
file chemin/vers/ton_fichier.mp3
```
![Capture d'écran 2025-03-13 110941](https://github.com/user-attachments/assets/5c000028-6d3a-42a8-9385-de5ccbb0efc9)

![Capture d'écran 2025-03-13 110914](https://github.com/user-attachments/assets/f02a587e-e834-4500-aa29-a5b0e3cfad28)


#### **B. Commande `readelf`**
La commande `readelf` permet d'analyser les fichiers ELF.

**Commandes utilisées :**
```bash
readelf -h /bin/ls
readelf -S /bin/ls
```
![Capture d'écran 2025-03-13 111308](https://github.com/user-attachments/assets/af8a9b6c-c11b-4e2b-b4de-9ebc9a02d5ce)

![Capture d'écran 2025-03-13 111438](https://github.com/user-attachments/assets/b268f5f2-9cfd-4bc3-abc0-7480b2140ab4)


#### **C. Commande `ldd`**
La commande `ldd` permet de lister les librairies dynamiques utilisées par un programme.

**Commandes utilisées :**
```bash
ldd /bin/ls
```

![Capture d'écran 2025-03-13 111632](https://github.com/user-attachments/assets/439b3831-f839-4279-bad2-d10d73b1edf2)


### **2. Syscalls Basics**

#### **A. Liste des syscalls**
Les syscalls sont des appels système utilisés par les programmes pour interagir avec le noyau.

**Commandes utilisées :**
- Aucune commande ici, mais recherche manuelle des syscalls dans la documentation ou sur [cette liste](https://filippo.io/linux-syscall-table/).

**Tableau de quelques syscalls avec leurs identifiant**
  
    | Syscall | Identifiant |
    |---------|-------------|
    | read    | 0           |
    | write   | 1           |
    | execve  | 59          |


#### **B. Commande `objdump`**
La commande `objdump` permet de désassembler un programme pour afficher son code en assembleur.

**Commandes utilisées :**
```bash
objdump -d /bin/ls | grep "call"
objdump -d /bin/ls | grep "syscall"
objdump -d /lib64/libc.so.6 | grep "syscall"
ldd /bin/ls | grep libc
objdump -d /lib64/libc.so.6 | grep -B 5 "mov.*\$0x3" | grep -B 5 -A 5 "syscall"
```
![Capture d'écran 2025-03-13 104806](https://github.com/user-attachments/assets/aa2476c5-c5b5-4776-a190-94673345df47)

Le programme `ls` ne contient pas directement d'instructions syscall. À la place, il utilise des fonctions de la Glibc, qui elles-mêmes appellent les syscalls.

![Capture d'écran 2025-03-13 110133](https://github.com/user-attachments/assets/493d06a2-ea9d-49f9-bf11-1512b63b32b6)

![Capture d'écran 2025-03-13 114142](https://github.com/user-attachments/assets/20a9899a-1009-46b3-a30e-70e4683f607f)

![Capture d'écran 2025-03-13 113806](https://github.com/user-attachments/assets/3d439127-fbb3-4e59-b280-f5b2d4668947)




## **Partie 2 : Observe**

## 1. strace  

### Commande exécutée :  
```sh
ls
strace ls 2>&1 | grep write
```
![Capture d'écran 2025-03-13 121821](https://github.com/user-attachments/assets/433f4648-5495-4e1e-92a8-11b6f6583370)


## Observation des syscalls de `cat`  

### Commande pour voir l’ouverture du fichier en lecture:  
```sh
strace cat mon_fichier.txt 2>&1 | grep openat
```
### Syscall identifié :  

![Capture d'écran 2025-03-13 122253](https://github.com/user-attachments/assets/438ed2b8-eee7-44b5-bf15-46430768013c)
 

### Commande pour voir l’écriture dans le terminal :  
```sh
strace cat passwords.txt 2>&1 | grep -A 20 "write"
```
### Syscall identifié :  

![Capture d'écran 2025-03-13 122513](https://github.com/user-attachments/assets/8cfd142c-2568-422c-a05d-5d57dc87e8c7)


## Observation des syscalls de `curl example.org`  

### Commande exécutée :  
```sh
strace -c curl example.org
```
### Résultat :  

![Capture d'écran 2025-03-13 122630](https://github.com/user-attachments/assets/4a8f7355-3739-4412-8c81-62102cc07c39)

- `-c` affiche un tableau récapitulatif du nombre de fois que chaque syscall est utilisé.  
- Permet d’avoir une vue d’ensemble sans voir chaque appel détaillé.  


Voici le compte rendu pour la partie "Observation des syscalls avec `sysdig`" :


## **2- sysdig**

`sysdig` est un outil qui permet de tracer les appels système (syscalls) reçus par le noyau Linux. Il fonctionne en temps réel, capturant et affichant ces appels, ce qui est utile pour observer ce qu'un processus fait en arrière-plan. On peut l'utiliser avec des filtres pour se concentrer sur des syscalls spécifiques.

### **Tracer les syscalls de `ls`**

#### **Commande utilisée :**  
```sh
sudo sysdig proc.name=ls 2>&1 | grep write
```

![Capture d'écran 2025-03-13 131853](https://github.com/user-attachments/assets/06a3457d-7035-48ca-b4b1-5c62a1980aa6)


### **Tracer les syscalls de `cat`**

#### **Commandes utilisées :**

##### **Ouvrir le fichier avec `openat` :**
```sh
sudo sysdig proc.name=cat 2>&1 | grep openat
```

![Capture d'écran 2025-03-13 132020](https://github.com/user-attachments/assets/10f7b355-25e8-4364-aa47-28d949e4a3dd)

##### **Écrire le contenu du fichier avec `write` :**
```sh
sudo sysdig proc.name=cat 2>&1 | grep write
```

![Capture d'écran 2025-03-13 132001](https://github.com/user-attachments/assets/f9c11250-f406-4d69-9181-ccec8a0a4cb7)


**Tracer les syscalls de l'utilisateur courant**

#### **Commande utilisée :**  
```sh
sudo sysdig user.name=$USER
```

### **Capturer les syscalls de `curl`**

#### **Commande utilisée :**  
```sh
sudo sysdig -w curl.scap proc.name=curl
```
