# Compte rendu TP 3 — Premiers conteneurs avec Docker

**Cours :** Introduction au Cloud · L3 SI RIST & BDGL  
**Étudiant / trinôme :** TRAORE IBRAHIM — KOFFI BONI — DJANGORAN CHRISTIAN  
**Professeur :** Dr. Bakayoko

**Livrable :** commandes lancées, réponse de chaque commande, tableau comparatif, réponses aux questions, lien vers l'image publiée.

> Invite hôte : `PS C:\Users\HP>`. Invite VM : `etudiant@ubuntu-srv:~$`.  
> Nom VirtualBox du modèle : `ubuntu-s` (nom d'hôte Linux : `ubuntu-srv`). Les commandes `VBoxManage` utilisent donc `ubuntu-s` à la place de `ubuntu-srv` de l'énoncé.

---

## Environnement

| Élément       | Valeur                                                                  |
| ------------- | ----------------------------------------------------------------------- |
| Hôte          | Windows 11, PowerShell, utilisateur `HP`                                |
| Hyperviseur   | Oracle VirtualBox (type 2)                                              |
| VM            | `ubuntu-s` (Ubuntu Server 26.04.1 LTS, noyau 7.0.0-38-generic, 2,6 Gi de RAM, utilisateur `etudiant`) |
| Docker        | Docker Engine 29.1.3 (paquet Ubuntu `docker.io`), containerd 2.2.2, runc 1.4.0 |
| Registre      | Docker Hub, compte `ibrahimbdgl`                                        |
| Redirections VirtualBox | `2222 → 22` (SSH), `8080 → 8080` (règle `web`), `5000 → 5000` (règle `api`), déjà présentes sur la VM |

### Architecture réalisée

| Conteneur | Image                          | Port dans le conteneur | Port dans la VM | Adresse depuis l'hôte    |
| --------- | ------------------------------ | ---------------------- | --------------- | ------------------------ |
| `web`     | `nginx`                        | 80                     | 8080            | `http://127.0.0.1:8080`  |
| `api`     | `api:1.0` puis image publiée   | 5000                   | 5000            | `http://127.0.0.1:5000`  |

---

## Partie 0 — Préparer la VM et installer Docker

### 0.1 Démarrer la VM et s'y connecter

**Commandes** (hôte)

```powershell
$env:Path += ";C:\Program Files\Oracle\VirtualBox"
VBoxManage startvm ubuntu-s --type headless
ssh -p 2222 etudiant@127.0.0.1
```

**Réponse**

```text
Waiting for VM "ubuntu-s" to power on...
VM "ubuntu-s" has been successfully started.
etudiant@127.0.0.1's password:
Welcome to Ubuntu 26.04.1 LTS (GNU/Linux 7.0.0-38-generic x86_64)

 * Documentation:  https://docs.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/pro

 System information as of mar. 27 oct 2026 19:36:15 UTC
```

**Lecture.** La première commande n'affiche rien *(sortie attendue)* : le dossier VirtualBox est ajouté au `PATH` de la session PowerShell. La VM démarre en **headless** (sans fenêtre) et on s'y connecte en SSH par la redirection `127.0.0.1:2222`.

### 0.2 Installer Docker

**Commandes** (VM)

```bash
sudo apt update
sudo apt install -y docker.io docker-buildx docker-compose-v2
sudo usermod -aG docker $USER
exit
```

**Réponse** (`apt update`)

```text
[sudo: authenticate] Password:
Atteint : 1 http://ci.archive.ubuntu.com/ubuntu resolute InRelease
Atteint : 2 http://ci.archive.ubuntu.com/ubuntu resolute-updates InRelease
Atteint : 3 http://ci.archive.ubuntu.com/ubuntu resolute-backports InRelease
Atteint : 4 http://security.ubuntu.com/ubuntu resolute-security InRelease
29 paquets peuvent être mis à jour. Exécutez « apt list --upgradable » pour les voir.
```

**Réponse** (fin de `apt install`)

```text
Dépaquetage de docker.io (29.1.3-0ubuntu4.1) ...
Dépaquetage de docker-buildx (0.30.1-0ubuntu1) ...
Dépaquetage de docker-compose-v2 (2.40.3+ds1-0ubuntu1) ...
Paramétrage de containerd (2.2.2-0ubuntu1.2) ...
Created symlink '/etc/systemd/system/multi-user.target.wants/containerd.service' → '/usr/lib/systemd/system/containerd.service'.
Paramétrage de docker.io (29.1.3-0ubuntu4.1) ...
Created symlink '/etc/systemd/system/multi-user.target.wants/docker.service' → '/usr/lib/systemd/system/docker.service'.
Created symlink '/etc/systemd/system/sockets.target.wants/docker.socket' → '/usr/lib/systemd/system/docker.socket'.
Paramétrage de docker-buildx (0.30.1-0ubuntu1) ...
Scanning processes...
Scanning linux images...

Running kernel seems to be up-to-date.
No services need to be restarted.
No containers need to be restarted.
No user sessions are running outdated binaries.
No VM guests are running outdated hypervisor (qemu) binaries on this host.
```

**Réponse** (`usermod`, `exit`) *(sortie attendue)* : aucune sortie pour `usermod` ; `exit` ferme la session SSH (`logout`). Il faut ensuite se **reconnecter** pour que le groupe `docker` soit pris en compte.

**Lecture.** `apt update` rafraîchit l'index des paquets (29 mises à jour disponibles, non installées). `apt install` installe le moteur Docker (`docker.io`), `buildx` (construction d'images), `compose v2`, ainsi que `containerd` et `runc` qui exécutent réellement les conteneurs. Les services `docker` et `containerd` sont **activés au démarrage** (liens symboliques créés). `usermod -aG docker` **ajoute** (`-a`) l'utilisateur `etudiant` au groupe `docker` (`-G`) sans le retirer des autres groupes.

### 0.3 Vérifier l'installation

**Commandes** (VM, après reconnexion)

```bash
docker version
docker info | grep -E "Server Version|Storage Driver|Cgroup|Kernel Version"
```

**Réponse** (`docker version`)

```text
Client:
 Version:           29.1.3
 API version:       1.52
 Go version:        go1.24.13
 Git commit:        29.1.3-0ubuntu4.1
 Built:             Wed Apr 29 16:40:20 2026
 OS/Arch:           linux/amd64
 Context:           default

Server:
 Engine:
  Version:          29.1.3
  API version:      1.52 (minimum version 1.44)
  Go version:       go1.24.13
  Git commit:       29.1.3-0ubuntu4.1
  Built:            Wed Apr 29 16:40:20 2026
  OS/Arch:          linux/amd64
  Experimental:     false
 containerd:
  Version:          2.2.2
 runc:
  Version:          1.4.0-0ubuntu1
 docker-init:
  Version:          0.19.0
```

**Réponse** (`docker info | grep ...`)

```text
Server Version: 29.1.3
Storage Driver: overlayfs
Cgroup Driver: systemd
Cgroup Version: 2
Kernel Version: 7.0.0-38-generic
```

**Lecture.** La présence de la section **Server** prouve que le client joint le démon sans `sudo`, donc que l'appartenance au groupe `docker` est effective. Configuration : stockage `overlayfs`, cgroups **v2** pilotés par `systemd`.

### Réponses aux questions

**Q1. Quelle version de Docker est installée ? Quel noyau Docker utilise-t-il ?**

Docker **29.1.3** (client et serveur, API 1.52). La ligne `Kernel Version` indique **7.0.0-38-generic**, c'est-à-dire **le noyau de la VM Ubuntu** : Docker n'a pas de noyau propre, il utilise celui de l'hôte (ici la VM).

**Q2. Pourquoi être membre du groupe** `docker` **revient presque à être administrateur ? Quelle précaution sur un serveur partagé ?**

Le démon Docker tourne en **root** et la socket `/var/run/docker.sock` est accessible aux membres du groupe `docker`. Un membre peut donc demander au démon de créer un conteneur qui **monte n'importe quel dossier de la machine** (par exemple `/`) avec les droits root, et ainsi lire ou modifier tous les fichiers, y compris `/etc/shadow`. Sur un serveur partagé, il faut donc n'ajouter au groupe que des **utilisateurs de confiance**, privilégier un accès via `sudo` journalisé, ou utiliser Docker en mode **rootless** (le démon ne tourne plus en root).

---

## Partie 1 — Lancer et inspecter des conteneurs

### 1.1 Premier conteneur

**Commandes** (VM)

```bash
docker run hello-world
docker images
docker ps -a
```

**Réponse** (`docker run hello-world`)

```text
Unable to find image 'hello-world:latest' locally
latest: Pulling from library/hello-world
4f55086f7dd0: Pull complete
d5e71e642bf5: Download complete
Digest: sha256:c567b231f65614688711a75f31518a064258a51d442117a3e771a3a8fdba78ef
Status: Downloaded newer image for hello-world:latest

Hello from Docker!
This message shows that your installation appears to be working correctly.

To generate this message, Docker took the following steps:
 1. The Docker client contacted the Docker daemon.
 2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
    (amd64)
 3. The Docker daemon created a new container from that image which runs the
    executable that produces the output you are currently reading.
 4. The Docker daemon streamed that output to the Docker client, which sent it
    to your terminal.

To try something more ambitious, you can run an Ubuntu container with:
 $ docker run -it ubuntu bash

Share images, automate workflows, and more with a free Docker ID:
 https://hub.docker.com/

For more examples and ideas, visit:
 https://docs.docker.com/get-started/
```

**Réponse** (`docker images`)

```text
IMAGE                ID             DISK USAGE   CONTENT SIZE   EXTRA
hello-world:latest   c567b231f656       25.9kB         9.49kB    U
```

**Réponse** (`docker ps -a`)

```text
CONTAINER ID   IMAGE         COMMAND    CREATED              STATUS                     PORTS   NAMES
f5f0d0289262   hello-world   "/hello"   About a minute ago   Exited (0) 58 seconds ago          laughing_stonebraker
```

**Lecture.** L'image n'existait pas : Docker la télécharge depuis le Docker Hub (`Pull complete`), puis crée et lance un conteneur. Le conteneur **s'arrête dès que son programme** `/hello` **a fini** : `Exited (0)` (code 0 = succès). Il n'apparaît que dans `docker ps -a`. Le nom `laughing_stonebraker` est généré aléatoirement.

### 1.2 Un serveur web en une commande

**Commandes** (VM)

```bash
docker run -d --name web -p 8080:80 nginx
docker ps
curl -s http://localhost:8080 | head -n 5
```

**Réponse** (`docker run -d --name web -p 8080:80 nginx`)

```text
Unable to find image 'nginx:latest' locally
latest: Pulling from library/nginx
ee284e8d8c11: Download complete
8334177aa35b: Pull complete
f2be67225471: Pull complete
dd00b11a22aa: Pull complete
d5f3a4724daf: Pull complete
9a2c3eb3312e: Pull complete
24bf03b6a474: Pull complete
a856bfc2a772: Pull complete
6542b5f86ffd: Download complete
Digest: sha256:4c41e2d0a3a55b325ed572cd0200bdb72af1433c3daa2bca6ef8eea68ce504a2
Status: Downloaded newer image for nginx:latest
d7052bde2f08e69797d9dbae6d8454cb40eadc337e870a407d5c3f6876b80318
```

**Réponse** (`docker ps`)

```text
CONTAINER ID   IMAGE   COMMAND                  CREATED          STATUS          PORTS                                     NAMES
d7052bde2f08   nginx   "/docker-entrypoint.…"   41 seconds ago   Up 37 seconds   0.0.0.0:8080->80/tcp, [::]:8080->80/tcp   web
```

**Réponse** (`curl -s http://localhost:8080| head -n 5`)

```text
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
```

**Étape 3 — navigateur de l'hôte.** La page « Welcome to nginx! » s'affiche sur `http://127.0.0.1:8080`. Les journaux de Nginx le confirment : requêtes `GET /` en **200** venant de `10.0.2.2` (l'hôte Windows vu depuis la VM) avec l'agent `Edg/156` .

**Lecture.** `-d` lance le conteneur en arrière-plan, `--name web` le nomme, `-p 8080:80` publie le port 80 du conteneur sur le port 8080 de la VM. Dans `PORTS`, `0.0.0.0:8080->80/tcp` signifie « toutes les interfaces IPv4 », `[::]` l'équivalent IPv6. La longue ligne affichée par `docker run -d` est l'**ID complet** du conteneur.

### 1.3 Regarder à l'intérieur

**Commandes** (VM)

```bash
docker exec web cat /etc/os-release | head -n 2
cat /etc/os-release | head -n 2
docker exec web uname -r
uname -r
```

**Réponse**

```text
PRETTY_NAME="Debian GNU/Linux 13 (trixie)"
NAME="Debian GNU/Linux"
PRETTY_NAME="Ubuntu 26.04.1 LTS"
NAME="Ubuntu"
7.0.0-38-generic
7.0.0-38-generic
```

**Lecture.** Le conteneur est un **Debian 13**, la VM un **Ubuntu 26.04.1**, mais les deux affichent le **même noyau** `7.0.0-38-generic`.

**Commandes** (VM)

```bash
docker logs --tail 5 web
docker top web
ps aux | grep "nginx: master" | grep -v grep
docker stats --no-stream web
```

**Réponse** (`docker logs --tail 5 web`)

```text
2026/10/27 20:22:56 [notice] 1#1: start worker process 31
172.17.0.1 - - [27/Oct/2026:20:24:40 +0000] "GET / HTTP/1.1" 200 896 "-" "curl/8.18.0" "-"
10.0.2.2 - - [27/Oct/2026:20:26:14 +0000] "GET / HTTP/1.1" 200 896 "-" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/156.0.0.0 Safari/537.36 Edg/156.0.0.0" "-"
10.0.2.2 - - [27/Oct/2026:20:26:14 +0000] "GET /favicon.ico HTTP/1.1" 404 555 "http://127.0.0.1:8080/" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/156.0.0.0 Safari/537.36 Edg/156.0.0.0" "-"
2026/10/27 20:26:14 [error] 30#30: *2 open() "/usr/share/nginx/html/favicon.ico" failed (2: No such file or directory), client: 10.0.2.2, server: localhost, request: "GET /favicon.ico HTTP/1.1", host: "127.0.0.1:8080", referrer: "http://127.0.0.1:8080/"
```

**Réponse** (`docker top web`)

```text
UID      PID    PPID   C   STIME   TTY   TIME       CMD
root     2488   2464   0   20:22   ?     00:00:00   nginx: master process nginx -g daemon off;
syslog   2569   2488   0   20:22   ?     00:00:00   nginx: worker process
syslog   2570   2488   0   20:22   ?     00:00:00   nginx: worker process
syslog   2571   2488   0   20:22   ?     00:00:00   nginx: worker process
```

**Réponse** (`ps aux | grep "nginx: master" | grep -v grep`)

```text
root  2488  0.0  0.3  14976  9268 ?  Ss  20:22  0:00 nginx: master process nginx -g daemon off;
```

**Réponse** (`docker stats --no-stream web`)

```text
CONTAINER ID   NAME   CPU %   MEM USAGE / LIMIT     MEM %   NET I/O          BLOCK I/O     PIDS
d7052bde2f08   web    0.00%   4.145MiB / 2.585GiB   0.16%   4.04kB / 4.01kB  0B / 24.6kB   4
```

**Lecture.**

- `logs` : la requête `curl` vient de `172.17.0.1` (la VM vue depuis le réseau Docker), celle du navigateur de `10.0.2.2` (l'hôte Windows vu depuis la VM). Le `404` sur `/favicon.ico` est normal : le navigateur réclame l'icône de l'onglet, qui n'existe pas.
- `top` : un processus **master** (root) et **3 workers**, soit **4 processus**, ce que confirme la colonne `PIDS` de `docker stats`.
- `stats` : Nginx au repos consomme **4,1 Mio** de RAM et 0 % de CPU, sans limite fixée (la limite affichée est la RAM de la VM, 2,585 Gio).

### 1.4 Un conteneur est éphémère

**Commandes** (VM)

```bash
docker exec web sh -c 'echo "<h1>trinome TRAORE IBRAHIM - KOFFI BONI - DJANGORAN CHRISTIAN</h1>" > /usr/share/nginx/html/index.html'
curl -s http://localhost:8080
docker rm -f web
docker run -d --name web -p 8080:80 nginx
curl -s http://localhost:8080 | grep -o "<title>.*</title>"
```

**Réponse**

```text
(docker exec ...)   aucune sortie (sortie attendue : le fichier est écrit dans le conteneur)
<h1>trinome TRAORE IBRAHIM - KOFFI BONI - DJANGORAN CHRISTIAN</h1>
web
98fef72969163fa2a94c272d099f3b8383426bfbc11df68e24d4d0bc6e7965a1
<title>Welcome to nginx!</title>
```

**Lecture.** La page modifiée est servie immédiatement, puis `docker rm -f web` détruit le conteneur **et sa couche d'écriture**. Le nouveau conteneur (ID différent) est recréé depuis l'image d'origine : la page par défaut « Welcome to nginx! » est revenue, la modification est **perdue**.

**Commandes** (VM, avec un volume)

```bash
mkdir -p ~/site
echo "<h1>trinome TRAORE IBRAHIM - KOFFI BONI - DJANGORAN CHRISTIAN : servi depuis un volume</h1>" > ~/site/index.html
docker rm -f web
docker run -d --name web -p 8080:80 -v ~/site:/usr/share/nginx/html:ro nginx
curl -s http://localhost:8080
```

**Réponse**

```text
(mkdir -p ~/site)   aucune sortie (sortie attendue)
(echo > ...)        aucune sortie (sortie attendue)
web
386811689f4df72c9b6da7bc1b129a94140d793588a5491bb3b40fc3aee19dbf
<h1>trinome TRAORE IBRAHIM - KOFFI BONI - DJANGORAN CHRISTIAN : servi depuis un volume</h1>
```

**Lecture.** Le fichier vit **sur la VM** (`~/site/index.html`) et est monté dans le conteneur à l'emplacement où Nginx cherche ses pages. `:ro` (read-only) empêche le conteneur de le modifier. On peut supprimer et recréer le conteneur, la page reste.

### Réponses aux questions

**Q1. D'après le message de hello-world, quelles sont les quatre étapes réalisées par Docker ?**

1. Le **client** Docker contacte le **démon** Docker.
2. Le démon **télécharge** l'image `hello-world` depuis le **Docker Hub** (version amd64).
3. Le démon **crée un nouveau conteneur** à partir de cette image, qui exécute le programme produisant le message.
4. Le démon **renvoie cette sortie au client**, qui l'affiche dans le terminal.

**Q2. Le conteneur** `web` **et la VM ont-ils la même distribution ? Le même noyau ? Explication.**

**Distribution : non.** Le conteneur est un **Debian GNU/Linux 13 (trixie)**, la VM un **Ubuntu 26.04.1 LTS**. **Noyau : oui**, tous deux affichent `7.0.0-38-generic`. D'après le cours, un conteneur **ne virtualise pas le matériel mais le système d'exploitation** : il embarque ses propres bibliothèques et son propre système de fichiers (c'est ce que fournit l'image), mais il **partage le noyau de l'hôte**. Il n'y a pas d'OS invité, contrairement à une VM.

**Q3. Les processus Nginx du conteneur apparaissent-ils dans** `ps aux` **sur la VM ? Qu'est-ce que cela montre ?**

**Oui** : `ps aux` sur la VM montre `nginx: master process nginx -g daemon off;` avec le **PID 2488**, le même que celui affiché par `docker top web`. Un conteneur n'est donc **qu'un processus (ou un groupe de processus) ordinaire de l'hôte**, isolé par les **namespaces** (vue propre des processus, du réseau, des fichiers) et limité par les **cgroups** (CPU, RAM). Ce n'est pas une machine virtuelle avec son propre noyau, ce qui explique sa légèreté.

**Q4. Chemin d'une requête du navigateur de l'hôte jusqu'à Nginx.**

```text
Navigateur Windows  →  127.0.0.1:8080 (hôte)
                    →  port 8080 de la VM     (1re redirection : règle NAT VirtualBox « web »)
                    →  port 80 du conteneur   (2e redirection : option Docker -p 8080:80)
                    →  Nginx
```

Elle traverse **deux redirections de port** : VirtualBox (hôte → VM), puis Docker (VM → conteneur). La preuve est dans les journaux : la requête du navigateur apparaît avec l'adresse source `10.0.2.2`.

**Q5. Pourquoi la page modifiée a-t-elle disparu après** `docker rm` **? Que change l'option** `-v` **?**

Une image est **en lecture seule** ; un conteneur y ajoute une fine **couche d'écriture temporaire**. La modification faite avec `docker exec` a été écrite dans cette couche, **détruite avec le conteneur** par `docker rm`. Avec `-v ~/site:/usr/share/nginx/html:ro`, le dossier de la **VM** est **monté** dans le conteneur : le fichier est stocké en dehors de la couche du conteneur, donc **conservé** après suppression, et modifiable depuis la VM sans redémarrer le conteneur.

---

## Partie 2 — Comparer conteneur et VM

**Commandes** (VM)

```bash
docker pull alpine
docker images
time docker run --rm alpine echo "Bonjour depuis un conteneur"
time docker run --rm alpine echo "Bonjour depuis un conteneur"
systemd-analyze
free -h
```

**Réponse** (`docker pull alpine`)

```text
Using default tag: latest
latest: Pulling from library/alpine
7b229517ed22: Pull complete
3ec36f3670df: Download complete
3da1513d2124: Download complete
Digest: sha256:be2656061b536c1883c4034790e21d0a09c5aaaf51968b89368a3c0371ea2c9a
Status: Downloaded newer image for alpine:latest
docker.io/library/alpine:latest
```

**Réponse** (`docker images`)

```text
IMAGE                ID             DISK USAGE   CONTENT SIZE   EXTRA
alpine:latest        be2656061b53        13MB         3.94MB
hello-world:latest   c567b231f656      25.9kB         9.49kB    U
nginx:latest         4c41e2d0a3a5       243MB         66.8MB    U
```

**Réponse** (`time docker run --rm alpine echo ...`, deux fois)

```text
Bonjour depuis un conteneur

real    0m5,129s
user    0m0,118s
sys     0m0,309s

Bonjour depuis un conteneur

real    0m3,958s
user    0m0,054s
sys     0m0,246s
```

**Réponse** (`systemd-analyze`)

```text
Startup finished in 10.251s (kernel) + 26.490s (initrd) + 1min 4.995s (userspace) = 1min 41.737s
graphical.target reached after 1min 4.990s in userspace.
```

**Réponse** (`free -h`)

```text
               total        used        free      shared  buff/cache   available
Mem:           2,6Gi       509Mi       1,0Gi       1,2Mi       1,3Gi       2,1Gi
Swap:          2,0Gi          0B       2,0Gi
```

**Lecture.** `--rm` supprime le conteneur à sa fin. `time` affiche la durée réelle (`real`), le temps CPU en mode utilisateur (`user`) et noyau (`sys`). `docker pull` télécharge une image sans créer de conteneur. L'image `alpine` pèse **13 Mo** sur disque contre **243 Mo** pour `nginx`.

Le conteneur `alpine` est une distribution Alpine Linux, mais son noyau est celui de la VM (`7.0.0-38-generic`).

### Tableau comparatif

| Mesure             | VM `ubuntu-srv`                             | Conteneur `nginx`                             | Conteneur `alpine`                                                                                                                                                      |
| ------------------ | ------------------------------------------- | --------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Taille sur disque  | **8,65 Go** (dossier de la VM, TP 1)        | **243 Mo** (contenu : 66,8 Mo)                | **13 Mo** (contenu : 3,94 Mo)                                                                                                                                           |
| Temps de démarrage | **1 min 41,7 s** (`systemd-analyze`)        | **6,15 s**                                    | **3,96 s** (2e `time docker run`) ; 5,13 s au 1er essai                                                                                                                 |
| Mémoire utilisée   | **509 Mi** (`free -h`, colonne `used`)      | **4,145 MiB** (`docker stats`)                | **356 KiB**                                                                                                                                                             |
| Système            | Ubuntu 26.04.1 LTS complet + noyau 7.0.0-38 | Debian GNU/Linux 13 (trixie), noyau de l'hôte | Alpine Linux, noyau de l'hôte *(commande* `docker run --rm alpine cat /etc/os-release` *à relancer pour la version exacte ; sortie attendue :* `NAME="Alpine Linux"`*)* |

### Réponses aux questions

**Q1. Dans quel rapport le conteneur est-il plus léger et plus rapide que la VM ? Les mesures confirment-elles le tableau du cours ?**

| Critère        | VM        | Conteneur         | Rapport                                 |
| -------------- | --------- | ----------------- | --------------------------------------- |
| Taille         | 8,65 Go   | 243 Mo / 13 Mo    | environ **×35** (nginx) à **×660** (alpine) |
| Démarrage      | 101,7 s   | 3,96 s            | environ **×26**                         |
| Mémoire        | 509 Mi    | 4,1 MiB           | environ **×120**                        |

Les mesures **confirment le sens du tableau du cours** (« plusieurs Go » contre « quelques dizaines à centaines de Mo » ; « dizaines de secondes à minutes » contre « environ une seconde »). Nuance : le démarrage du conteneur (~4 s) est plus lent que la seconde annoncée. Le temps CPU réel est très faible (`user` + `sys` ≈ 0,3 s) : la commande a surtout **attendu**, ce qui s'explique probablement par les ressources limitées de la VM VirtualBox (disque virtuel lent, processeur partagé avec Windows). Le cours précise d'ailleurs que ces ordres de grandeur varient selon le matériel.

**Q2. Pourquoi le premier** `time docker run` **est-il plus long que le second ?**

L'image étant déjà téléchargée, la différence vient des **caches** : au premier lancement, les fichiers de l'image, les métadonnées de la couche `overlayfs` et les binaires nécessaires doivent être lus **depuis le disque**. Au second, ils sont déjà dans le **cache mémoire du système** : 5,13 s contre 3,96 s.

**Q3. Citez un cas où vous choisiriez quand même une VM plutôt qu'un conteneur.**

Quand il faut **un autre système d'exploitation ou un autre noyau** (par exemple une application Windows sur un hôte Linux), quand l'**isolation doit être maximale** (applications de clients différents, code non fiable : un conteneur partage le noyau, une faille noyau peut donc affecter l'hôte), ou pour **migrer tel quel un serveur existant**.

---

## Partie 3 — Conteneuriser l'API fil rouge

### 3.1 Code de l'API

**Commandes** (VM)

```bash
mkdir -p ~/api && cd ~/api
cat > app.py <<'EOF'
import os
import socket
from flask import Flask, jsonify

app = Flask(__name__)

@app.get("/")
def accueil():
    return jsonify(
        message="API fil rouge",
        trinome=os.environ.get("TRINOME", "TRAORE IBRAHIM - KOFFI BONI - DJANGORAN CHRISTIAN"),
        conteneur=socket.gethostname(),
    )

@app.get("/sante")
def sante():
    return jsonify(statut="ok")

if __name__ == "__main__":
    app.run(host="0.0.0.0", port=5000)
EOF
echo "flask>=3,<4" > requirements.txt
```

**Réponse** *(sortie attendue)* : aucune sortie ; le prompt passe à `etudiant@ubuntu-srv:~/api$` et les fichiers `app.py` et `requirements.txt` sont créés.

**Lecture.** `app.py` expose deux routes : `/` (message, trinôme lu dans la variable d'environnement `TRINOME`, et nom du conteneur) et `/sante` (contrôle d'état). `host="0.0.0.0"` est **indispensable en conteneur** : avec `127.0.0.1`, l'API ne serait joignable que depuis l'intérieur du conteneur. Le trinôme est lu dans `TRINOME` .

### 3.2 Dockerfile et `.dockerignore`

**Commandes** (VM, dossier `~/api`)

```bash
cat > Dockerfile <<'EOF'
FROM python:3.12-slim
WORKDIR /app
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt
COPY . .
EXPOSE 5000
CMD ["python", "app.py"]
EOF
printf '__pycache__/\n*.pyc\n.git\n' > .dockerignore
```

**Réponse** *(sortie attendue)* : aucune sortie ; le dossier `~/api` contient `app.py`, `requirements.txt`, `Dockerfile` et `.dockerignore`.

### 3.3 Construire l'image

**Commandes** (VM, dossier `~/api`)

```bash
docker build -t api:1.0 .
docker images api
docker history api:1.0
```

**Réponse** (`docker build -t api:1.0 .`)

```text
[+] Building 67.2s (10/10) FINISHED                                   docker:default
 => [internal] load build definition from Dockerfile                           0.6s
 => => transferring dockerfile: 195B                                           0.1s
 => [internal] load metadata for docker.io/library/python:3.12-slim           18.9s
 => [internal] load .dockerignore                                              0.3s
 => => transferring context: 64B                                               0.0s
 => [1/5] FROM docker.io/library/python:3.12-slim@sha256:12316debbfb8…        13.2s
 => => resolve docker.io/library/python:3.12-slim@sha256:12316debbfb8…         0.6s
 => => sha256:86586d55cc56… 254B / 254B                                        0.3s
 => => sha256:ca30a39e3f58… 12.12MB / 12.12MB                                  3.2s
 => => sha256:f5b253875fcb… 1.55MB / 1.55MB                                    3.5s
 => => extracting sha256:f5b253875fcb…                                         2.2s
 => => extracting sha256:ca30a39e3f58…                                         5.4s
 => => extracting sha256:86586d55cc56…                                         0.2s
 => [internal] load build context                                              0.4s
 => => transferring context: 796B                                              0.1s
 => [2/5] WORKDIR /app                                                         0.5s
 => [3/5] COPY requirements.txt .                                              0.4s
 => [4/5] RUN pip install --no-cache-dir -r requirements.txt                  20.2s
 => [5/5] COPY . .                                                             0.7s
 => exporting to image                                                         9.9s
 => => exporting layers                                                        5.4s
 => => exporting manifest sha256:90954aabdb6e…                                 0.1s
 => => exporting config sha256:8a93e592840a…                                   0.1s
 => => exporting attestation manifest sha256:4e99369d2c92…                     0.2s
 => => exporting manifest list sha256:18488bef4b59…                            0.1s
 => => naming to docker.io/library/api:1.0                                     0.0s
 => => unpacking to docker.io/library/api:1.0                                  3.0s
```

**Réponse** (`docker images api`)

```text
IMAGE     ID             DISK USAGE   CONTENT SIZE   EXTRA
api:1.0   18488bef4b59        200MB         48.5MB
```

**Réponse** (`docker history api:1.0`)

```text
IMAGE          CREATED              CREATED BY                                       SIZE      COMMENT
18488bef4b59   About a minute ago   CMD ["python" "app.py"]                          0B        buildkit.dockerfile.v0
<missing>      About a minute ago   EXPOSE [5000/tcp]                                0B        buildkit.dockerfile.v0
<missing>      About a minute ago   COPY . . # buildkit                              20.5kB    buildkit.dockerfile.v0
<missing>      About a minute ago   RUN /bin/sh -c pip install --no-cache-dir -r…    15.3MB    buildkit.dockerfile.v0
<missing>      About a minute ago   COPY requirements.txt . # buildkit               12.3kB    buildkit.dockerfile.v0
<missing>      About a minute ago   WORKDIR /app                                     8.19kB    buildkit.dockerfile.v0
<missing>      22 hours ago         CMD ["python3"]                                  0B        buildkit.dockerfile.v0
<missing>      22 hours ago         RUN /bin/sh -c set -eux; for src in idle3 p…     16.4kB    buildkit.dockerfile.v0
<missing>      22 hours ago         RUN /bin/sh -c set -eux; savedAptMark="$(a…      41.4MB    buildkit.dockerfile.v0
<missing>      23 hours ago         ENV PYTHON_SHA256=b5796915dc45ff745cfb4e990a…    0B        buildkit.dockerfile.v0
<missing>      23 hours ago         ENV PYTHON_VERSION=3.12.16                       0B        buildkit.dockerfile.v0
<missing>      23 hours ago         ENV GPG_KEY=7169605F62C751356D054A26A821E680…    0B        buildkit.dockerfile.v0
<missing>      23 hours ago         RUN /bin/sh -c set -eux; apt-get update; a…      7.31MB    buildkit.dockerfile.v0
<missing>      23 hours ago         ENV LANG=C.UTF-8                                 0B        buildkit.dockerfile.v0
<missing>      23 hours ago         ENV PATH=/usr/local/bin:/usr/local/sbin:/usr…    0B        buildkit.dockerfile.v0
<missing>      7 days ago           # debian.sh --arch 'amd64' out/ 'trixie' '@1…   87.7MB    debuerreotype 0.17
```

**Lecture.** Le premier build dure **67,2 s** (téléchargement de l'image de base `python:3.12-slim`, 20,2 s pour `pip install`). Les 6 premières lignes de l'historique sont celles du Dockerfile (« About a minute ago »), les 10 autres viennent de l'image de base Python 3.12.16 sur Debian trixie. Les `<missing>` sont normaux (les ID des couches intermédiaires ne sont pas conservés localement).

### 3.4 Lancer l'API

**Commandes** (VM)

```bash
docker run -d --name api -p 5000:5000 -e TRINOME="TRAORE IBRAHIM - KOFFI BONI - DJANGORAN CHRISTIAN" api:1.0
curl -s http://localhost:5000
curl -s http://localhost:5000/sante
docker logs api
```

**Réponse**

```text
c49636c96e0a8e82b473130ba3c124d6351e6d8cca1e3972e0268350840928c4
{"conteneur":"c49636c96e0a","message":"API fil rouge","trinome":"TRAORE IBRAHIM - KOFFI BONI - DJANGORAN CHRISTIAN"}
{"statut":"ok"}
 * Serving Flask app 'app'
 * Debug mode: off
WARNING: This is a development server. Do not use it in a production deployment. Use a production WSGI server instead.
 * Running on all addresses (0.0.0.0)
 * Running on http://127.0.0.1:5000
 * Running on http://172.17.0.3:5000
Press CTRL+C to quit
172.17.0.1 - - [27/Oct/2026 21:26:27] "GET / HTTP/1.1" 200 -
172.17.0.1 - - [27/Oct/2026 21:26:57] "GET /sante HTTP/1.1" 200 -
```

**Lecture.** Le champ `conteneur` (`c49636c96e0a`) est le début de l'ID complet : dans un conteneur, le *hostname* est l'ID court. Le conteneur a l'IP `172.17.0.3` sur le réseau Docker (`172.17.0.2` est déjà prise par `web`). L'avertissement « development server » de Flask est normal pour un TP.

### 3.5 Trois copies de la même image

**Commandes** (VM)

```bash
cd /home/
for p in 5001 5002 5003; do docker run -d --name api-$p -p $p:5000 -e TRINOME="TRAORE IBRAHIM - KOFFI BONI - DJANGORAN CHRISTIAN" api:1.0; done
for p in 5001 5002 5003; do curl -s http://localhost:$p; echo; done
docker rm -f api-5001 api-5002 api-5003
```

**Réponse**

```text
35daaf5f36c9452c5a4cc8c064b23c79d99d8bd58f537b5459cfd6936db47126
30de6e8b16776643ac946f8fc1ddb03058ea9a4d88ba6a7cb391316a5fbc8574
1b56fcfbc188d60d7d5210958c4f96cb54d5eab8e46be241abfc37e7e9d0e770
{"conteneur":"35daaf5f36c9","message":"API fil rouge","trinome":"TRAORE IBRAHIM - KOFFI BONI - DJANGORAN CHRISTIAN"}

{"conteneur":"30de6e8b1677","message":"API fil rouge","trinome":"TRAORE IBRAHIM - KOFFI BONI - DJANGORAN CHRISTIAN"}

{"conteneur":"1b56fcfbc188","message":"API fil rouge","trinome":"TRAORE IBRAHIM - KOFFI BONI - DJANGORAN CHRISTIAN"}

api-5001
api-5002
api-5003
```

**Lecture.** `cd /home/` n'affiche rien *(sortie attendue)*. La boucle lance trois conteneurs de la même image, chacun sur un port différent de la VM (5001, 5002, 5003) mais tous sur le port **5000 à l'intérieur** du conteneur, sans conflit puisque chacun a son propre espace réseau. Le conteneur `api` d'origine reste actif sur le port 5000.

### 3.6 Reconstruire en `api:1.1`

**Commandes** (VM, dossier `~/api`)

```bash
cd ~/api
docker build -t api:1.1 .
docker rm -f api && docker run -d --name api -p 5000:5000 api:1.1
```

**Réponse** (`docker build -t api:1.1 .`)

```text
[+] Building 15.9s (10/10) FINISHED                                   docker:default
 => [internal] load build definition from Dockerfile                           0.2s
 => => transferring dockerfile: 195B                                           0.1s
 => [internal] load metadata for docker.io/library/python:3.12-slim           12.1s
 => [internal] load .dockerignore                                              0.1s
 => => transferring context: 64B                                               0.0s
 => [1/5] FROM docker.io/library/python:3.12-slim@sha256:12316debbfb8…         0.5s
 => => resolve docker.io/library/python:3.12-slim@sha256:12316debbfb8…         0.4s
 => [internal] load build context                                              0.1s
 => => transferring context: 125B                                              0.0s
 => CACHED [2/5] WORKDIR /app                                                  0.0s
 => CACHED [3/5] COPY requirements.txt .                                       0.0s
 => CACHED [4/5] RUN pip install --no-cache-dir -r requirements.txt            0.0s
 => CACHED [5/5] COPY . .                                                      0.0s
 => exporting to image                                                         1.4s
 => => exporting layers                                                        0.1s
 => => exporting manifest sha256:90954aabdb6e…                                 0.0s
 => => exporting config sha256:8a93e592840a…                                   0.0s
 => => exporting attestation manifest sha256:ec9d90de81b1…                     0.3s
 => => exporting manifest list sha256:44dfb7344fa4…                            0.1s
 => => naming to docker.io/library/api:1.1                                     0.0s
 => => unpacking to docker.io/library/api:1.1                                  0.2s
```

**Réponse** (`docker rm -f api && docker run -d --name api -p 5000:5000 api:1.1`)

```text
api
fd4ff0c75f4a5c5cf280527b9bcaae763c22d71588b0ca386c4ec642d8675279
```

**Lecture.** Le build ne dure plus que **15,9 s** (contre 67,2 s) : les étapes `[2/5]` à `[5/5]` sont toutes `CACHED`, Docker réutilise les couches déjà construites. L'étape `COPY . .` est elle aussi en cache car le contenu du dossier est identique à celui de la version 1.0. Le remplacement du conteneur suit le schéma habituel : on ne modifie pas un conteneur, on le **supprime** (`docker rm -f api`, qui affiche `api`) et on en lance un **neuf** depuis la nouvelle image (`&&` : la seconde commande n'est exécutée que si la première réussit).

### Réponses aux questions

**Q1. Quelle est la taille de l'image** `api:1.0` **? Combien de couches** `docker history` **affiche-t-il, et lesquelles viennent du Dockerfile ?**

L'image fait **200 Mo** sur disque (**48,5 Mo** de contenu compressé). `docker history` affiche **16 lignes** : **6 viennent du Dockerfile** (`WORKDIR /app` 8,19 kB, `COPY requirements.txt` 12,3 kB, `RUN pip install` **15,3 Mo**, `COPY . .` 20,5 kB, `EXPOSE 5000` 0 B, `CMD` 0 B) et **10 viennent de l'image de base** `python:3.12-slim` (dont la couche Debian de 87,7 Mo et la couche Python de 41,4 Mo). Les instructions de métadonnées (`ENV`, `EXPOSE`, `CMD`) pèsent 0 B. Le code de l'API ne pèse presque rien (20,5 kB) face à Flask (15,3 Mo) et à l'image de base.

**Q2. Pourquoi copier** `requirements.txt` **et installer les dépendances avant de copier le code ?**

Docker met en cache chaque couche et ne la reconstruit que si l'instruction ou les fichiers qu'elle utilise **ont changé** (et invalide alors toutes les couches suivantes). Dans le build de l'étape 6, les lignes `CACHED [3/5]` et `CACHED [4/5]` montrent que `COPY requirements.txt` et `pip install` (20,2 s au premier build) sont **réutilisés**, d'où un build de 15,9 s au lieu de 67,2 s. Comme le code change souvent et les dépendances rarement, on place le code **après** : une modification de `app.py` ne refait que la dernière étape `COPY . .`, et le téléchargement de Flask n'est pas relancé. Si `COPY . .` était placé avant `pip install`, chaque modification du code invaliderait la couche des dépendances.

**Q3. Que renvoie le champ** `conteneur` **pour les trois copies ? Quelle caractéristique du cloud est illustrée ?**

Trois valeurs **différentes** : `35daaf5f36c9`, `30de6e8b1677`, `1b56fcfbc188` (l'ID court de chaque conteneur), alors que le code, le message et le trinôme sont identiques : ce sont **trois instances distinctes d'une seule image**. L'expérience illustre l'**élasticité rapide** : on ajoute ou retire des instances en quelques secondes pour suivre la charge, et le service est **mutualisé** sur les mêmes ressources. C'est la base de la mise à l'échelle horizontale.

**Q4. Pourquoi passer le nom du trinôme par une variable d'environnement (**`-e`**) plutôt que dans le code ? Quel lien avec les secrets ?**

La **configuration est séparée du code** : la même image fonctionne pour n'importe quel trinôme ou environnement (test, production) en changeant seulement `-e`, sans reconstruire l'image. Même principe pour les **secrets** (mots de passe, clés d'API) : ils ne doivent **jamais être écrits dans le code ni dans l'image**, car toute personne qui récupère l'image peut lire ses couches. On les fournit au démarrage (variables d'environnement, ou mieux, un gestionnaire de secrets).

---

## Partie 4 — Publier l'image et nettoyer

### 4.1 Publier sur Docker Hub (option A)

**Commandes** (VM)

```bash
docker login
docker tag api:1.1 ibrahimbdgl/api-fil-rouge:1.1
docker push ibrahimbdgl/api-fil-rouge:1.1
```

**Réponse** (`docker login`)

```text
USING WEB-BASED LOGIN

Info → To sign in with credentials on the command line, use 'docker login -u <username>'

Your one-time device confirmation code is: BVSW-ZVNJ
Press ENTER to open your browser or submit your device code here: https://login.docker.com/activate

Waiting for authentication in the browser…

WARNING! Your credentials are stored unencrypted in '/home/etudiant/.docker/config.json'.
Configure a credential helper to remove this warning. See
https://docs.docker.com/go/credential-store/

Login Succeeded
```

**Réponse** (`docker tag`) *(sortie attendue)* : aucune sortie ; l'image reçoit un second nom, sans copie de données.

**Réponse** (`docker push`)

```text
The push refers to repository [docker.io/ibrahimbdgl/api-fil-rouge]
be7134ff215e: Pushed
f2be67225471: Mounted from library/nginx
8d4c855670e7: Pushed
a036b5ae0229: Pushed
ca30a39e3f58: Pushed
6d296e2e3bc9: Pushed
86586d55cc56: Pushed
f5b253875fcb: Pushed
4d327922b02c: Pushed
1.1: digest: sha256:44dfb7344fa42e0a2f64ea258fdda1573a73cff0d2a749e8d8baad08f0a7df8c size: 856
```

**Lecture.** La connexion se fait par navigateur avec un code à usage unique. Le format `compte/depot:version` est obligatoire pour pousser vers son propre compte. La couche `f2be67225471` existe déjà sur Docker Hub (« Mounted from library/nginx ») : elle est liée, pas renvoyée. Le **digest** `sha256:44dfb73…` est le même que celui du *manifest list* de l'image `api:1.1` construite en 3.6.

### 4.2 Vérifier que l'image se récupère depuis le registre

**Commandes** (VM)

```bash
docker rm -f api
docker rmi ibrahimbdgl/api-fil-rouge:1.1 api:1.1
docker run -d --name api -p 5000:5000 ibrahimbdgl/api-fil-rouge:1.1
```

**Réponse**

```text
api
Untagged: ibrahimbdgl/api-fil-rouge:1.1
Untagged: api:1.1
Deleted: sha256:44dfb7344fa42e0a2f64ea258fdda1573a73cff0d2a749e8d8baad08f0a7df8c
Unable to find image 'ibrahimbdgl/api-fil-rouge:1.1' locally
1.1: Pulling from ibrahimbdgl/api-fil-rouge
Digest: sha256:44dfb7344fa42e0a2f64ea258fdda1573a73cff0d2a749e8d8baad08f0a7df8c
Status: Downloaded newer image for ibrahimbdgl/api-fil-rouge:1.1
9de92bc6f9825b9c59f94f94bdef2abb98ffeba222616a6f8acfdc4a8c2169cf
```

**Lecture.** Le conteneur `api` est supprimé en premier (Docker refuse de supprimer une image utilisée). `docker rmi` retire les deux noms de l'image, puis l'image elle-même (`Deleted`). Comme elle n'existe plus en local, `docker run` la **télécharge depuis Docker Hub** avant de démarrer le conteneur : le `Digest` est identique à celui du push, on a bien récupéré **exactement la même image**.

### 4.3 Nettoyer

**Commandes** (VM)

```bash
docker rm -f $(docker ps -aq)
docker system df
docker image prune -a -f
docker system df
```

**Réponse** (`docker rm -f $(docker ps -aq)`)

```text
9de92bc6f982
386811689f4d
f5f0d0289262
```

**Réponse** (1er `docker system df`)

```text
TYPE            TOTAL     ACTIVE    SIZE      RECLAIMABLE
Images          5         0         354.8MB   89.08MB (25%)
Containers      0         0         0B        0B
Local Volumes   0         0         0B        0B
Build Cache     11        0         112.9MB   36.86kB
```

**Réponse** (`docker image prune -a -f`)

```text
Deleted Images:
untagged: alpine:latest
deleted: sha256:be2656061b536c1883c4034790e21d0a09c5aaaf51968b89368a3c0371ea2c9a
deleted: sha256:c26612752c78b18faacd63c7d50f4592c2e7b0f25e0e20e82bd74e465c57ab0d
deleted: sha256:964745728d174226819257152a79b34561fe7321472eb9c8e32f461cbc7969c2
untagged: api:1.0
deleted: sha256:18488bef4b59fd1339ce890524a8fdef07e348e18cb812015b66ac8766f829ef
deleted: sha256:4e99369d2c921e40858dd0e365363a28248c69098c18cdef2b6455174f1f2593
untagged: hello-world:latest
deleted: sha256:c567b231f65614688711a75f31518a064258a51d442117a3e771a3a8fdba78ef
deleted: sha256:d1a8d0a4eeb63aff09f5f34d4d80505e0ba81905f36158cc3970d8e07179e59e
deleted: sha256:8e752a1cddeafc02597e756f4a0ec96e29f63ac4bc4af87682daf3f1de843bb7
untagged: nginx:latest
deleted: sha256:4c41e2d0a3a55b325ed572cd0200bdb72af1433c3daa2bca6ef8eea68ce504a2
deleted: sha256:59a9ae22b8933b536391e6c1708fa761297e922809c97675c5a56ba316be62af
deleted: sha256:37460afea904ea26c16f3ee17a46acd49adbe65dc804a852d9319f1ddbf8b975
untagged: ibrahimbdgl/api-fil-rouge:1.1
deleted: sha256:44dfb7344fa42e0a2f64ea258fdda1573a73cff0d2a749e8d8baad08f0a7df8c
deleted: sha256:90954aabdb6e2073f12ab804a80265c35187981999d0516a8594b28a3d610640
deleted: sha256:ec9d90de81b15f60b14c4a968527f80ffafed8ffaf9435e443f022c102544c12

Total reclaimed space: 40.91MB
```

**Réponse** (2e `docker system df`)

```text
TYPE            TOTAL     ACTIVE    SIZE      RECLAIMABLE
Images          0         0         152.1MB   152.1MB (100%)
Containers      0         0         0B        0B
Local Volumes   0         0         0B        0B
Build Cache     11        0         112.9MB   112.9MB
```

**Lecture.** `docker rm -f $(docker ps -aq)` supprime **tous** les conteneurs (`-q` n'affiche que les ID, `-a` inclut les arrêtés) : les trois conteneurs `api`, `web` et `hello-world`. `docker image prune -a -f` supprime **toutes les images inutilisées** (`-a` : pas seulement les images sans nom ; `-f` : sans confirmation). Docker reste installé dans la VM pour la séance 4.

### Réponses aux questions

**Q1. Adresse complète de l'image publiée. Publique ou privée ?**

`docker.io/ibrahimbdgl/api-fil-rouge:1.1`, page du dépôt : `https://hub.docker.com/r/ibrahimbdgl/api-fil-rouge`. Digest : `sha256:44dfb7344fa42e0a2f64ea258fdda1573a73cff0d2a749e8d8baad08f0a7df8c`. Le dépôt est **public** (visibilité par défaut à la création d'un dépôt sur Docker Hub) : n'importe qui peut la récupérer avec `docker run ibrahimbdgl/api-fil-rouge:1.1`, d'où l'interdiction d'y mettre des secrets.

**Q2. Pourquoi étiqueter l'image avec un numéro de version (**`1.1`**) plutôt que toujours** `latest` **?**

`latest` n'est qu'une étiquette **mobile** : elle pointe vers ce qui a été poussé en dernier et change sans prévenir. Avec un numéro de version, on sait **exactement ce qui tourne**, on peut **reproduire** un déploiement, **revenir en arrière** (relancer `1.0` si `1.1` pose problème) et faire coexister plusieurs versions. Avec `latest`, deux serveurs peuvent exécuter des versions différentes selon la date de leur dernier téléchargement.

**Q3. Quel espace disque** `docker image prune` **a-t-il libéré ? Pourquoi nettoyer aussi les registres dans le cloud ?**

`Total reclaimed space: 40.91MB`. Le tableau `docker system df` passe de 5 images (354,8 Mo) à 0 image ; il affiche encore 152,1 Mo, entièrement récupérables, car le magasin de contenu libère certaines données de façon différée. Le **cache de build** (112,9 Mo) n'est pas touché par `image prune` : il se vide avec `docker builder prune`. Dans le cloud, il faut aussi nettoyer les registres parce que **le stockage des images est facturé** (par Go et par mois), que les anciennes versions **s'accumulent** à chaque build, et qu'une vieille image conservée peut contenir des **failles de sécurité non corrigées** qu'on risque de redéployer. On utilise des **règles de cycle de vie** (suppression automatique des images anciennes ou sans étiquette).

---

## Conclusion — ce qu'il faut retenir du TP

Ce TP a montré qu'un **conteneur est une application emballée avec ses dépendances, qui s'exécute comme un simple processus de l'hôte**, là où une VM transporte un système entier.

**Conteneur et noyau.** Le conteneur `web` est un Debian 13 sur une VM Ubuntu 26.04, mais les deux affichent le même noyau `7.0.0-38-generic`, et le processus Nginx (PID 2488) est visible depuis la VM avec `ps`. Le conteneur n'a pas d'OS invité : il partage le noyau de l'hôte, ce qui le rend léger et rapide, mais moins isolé qu'une VM.

**Éphémère et persistant.** Ce qu'on écrit dans un conteneur disparaît avec lui (page modifiée perdue après `docker rm`). Un volume (`-v ~/site:/usr/share/nginx/html:ro`) place les données sur la VM et les conserve.

**Comparaison.** L'image `alpine` pèse 13 Mo et Nginx consomme 4,1 MiB, contre 8,65 Go et 509 Mi pour la VM. Le démarrage passe de 1 min 41 s à environ 4 s, un rapport de ×26 environ, un peu moins favorable que la seconde annoncée par le cours à cause des ressources limitées de la VM.

**Image et cache.** Un Dockerfile produit une image en couches (200 Mo, dont 87,7 Mo de Debian et 15,3 Mo de Flask). Placer `requirements.txt` et `pip install` avant `COPY . .` permet de réutiliser le cache : le build de `api:1.1` passe de 67,2 s à 15,9 s, avec des étapes `CACHED`. Une seule image a servi à lancer trois conteneurs distincts en quelques secondes.

**Publication.** L'image `ibrahimbdgl/api-fil-rouge:1.1` est publiée sur Docker Hub et a été relancée après suppression de toutes les copies locales (même digest `sha256:44dfb73…`). Le nettoyage (`docker image prune -a -f`) a libéré 40,91 Mo. Aucun secret ne doit figurer dans une image.

En une phrase : **un Dockerfile produit une image, publiée dans un registre et lancée en autant de conteneurs qu'on veut, sur n'importe quel serveur qui a Docker.**
