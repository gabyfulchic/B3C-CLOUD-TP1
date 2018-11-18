# B3C-CLOUD-TP1
repo pour le rendu .md du TP1

## Prerequisites

Pour le tp nous étions 3, donc seulement deux pc étaient branchés entre eux.
Celui de Paulin et Celui de Raphaël.

Adressage ip : 

```
Ip de Paulin : 10.33.10.1/24
Ip de Raphaël : 10.33.10.2/24
``` 

Ensuite on a récupéré le Vagranfile sur le github de coreos, nous l'avons lu, et nous avons dû nous renseigner sur la documentation officielle de Vagrant pour le modifier.
Il était question de l'ajout d'un disque de 10Go par VM créée, d'une interface bridgée sur la connexion de l'hôte et de 5 instances.
Pour les détails, le Vagrantfile sera à la racine du projet github.

Pour résumé, on a changé la variable $num_instances pour qu'elle soit égale à 5, puis nous avons ajoutez le disque et l'interface bridgée avec des commandes vb sous vagrant.
Pour la gestion user, il fallait juste changer le user-data.sample en user-data et rajouter un utilisateur/passwd à l'intérieur.

## Docker Swarm

Il faut savoir que si nous avons utilisé des vm sous coreos c'est tout simplement parce que nativement il comprend Docker déjà installé.
Donc pour vérifier que cela il nous suffit de nous connecter en ssh aux 5 instances et de le vérifier en CLI.

```
ssh core@127.0.0.1:2401 | 2402 | 2403 | 2404 | 2405

systemctl status docker

docker info

``` 
### Mise en place 

#### metrics-addr

Comme demandé nous avons ajouté lla ligne suivante dans le vagrantfile afin d'exécuter un script lors du vagrant up et nous avons testé la clause metrics-addr avec la cmd dockerd.

```
config.vm.provision :shell, :path => "scripts/config_docker.sh", :privileged => true

```

Ci-dessous le contenu du script :
```
#!/bin/bash

dockerd=/etc/docker
mkdir -p $dockerd

cat << EOF | tee $dockerd/daemon.json
{
    "experimental": true,
    "metrics-addr": "0.0.0.0:9323"
}
EOF
systemctl restart docker
```

#### Manager / Worker

Sur core-01 lancez la commande
```
docker swarm init --advertise-addr <IP_CURRENT_VM>
```

Vous aves donc initilisé docker swarm et Core-01 est donc le premier manager.

Nous allons promouvoir core-02 et core-03 en tant que managers. Lancez la commande suivante pour générer une commande à lancer sur les deux autres managers.
```docker swarm join-token manager```

Sur les deux derniers core, lancez la commande suivant pour générer une nouvelle commande à rentrer dans les machines core-04 et 05.
```docker swarm join-token worker```

### Weave Cloud

Q1 : Ce container peut lancer des containers grâce au token généré par la solution lors de l'installation sur l'hôte.

### NFS

#### Pré-requis

Nous avons installé un serveur CentOS 7 avec deux cartes réseaux : une host only et une NAT.

Nous avons set un nouvel hostname dans le fichier /etc/hostaname
```[root@server ~]# cat /etc/hostname
server.domain.com
```

Ne pas oublier de mettre à jour le serveur :
```yum update -y && yum upgrade -y```

#### Installation des packages NFS et configuration de celui-ci

Tout d'abord, nous avons installé tous les paquets nfs sur notre serveur :
```yum install nfs-utils```

Créons maintenant le dossier que nous allons partager :
```mkdir /var/partage```

et nous donnons les permissions dessus :
```chmod -R 755 /var/partage
chown nfsnobody:nfsnobody /var/partage
```

Nous démarrons les services nfs et profitons-en pour les activés au reboot :
```
systemctl enable rpcbind &&
systemctl enable nfs-server &&
systemctl enable nfs-lock &&
systemctl enable nfs-idmap &&
systemctl start rpcbind &&
systemctl start nfs-server &&
systemctl start nfs-lock &&
systemctl start nfs-idmap
```

Nous ajoutons au fichier /etc/exports le dossier que l'on partage et l'ip des machins clientes :
```
[root@server ~]# cat /etc/exports
/var/nfsshare   192.168.10.0/24(rw,sync,no_root_squash,no_all_squash)
```

Et nous restartons le service nfs-server :
```
systemctl restart nfs-server
```

Pour terminer, ajoutons les protocoles nécessaires pour le serveur nfs :
```
firewall-cmd --permanent --zone=public --add-service=nfs
firewall-cmd --permanent --zone=public --add-service=mountd
firewall-cmd --permanent --zone=public --add-service=rpc-bind
firewall-cmd --reload
```

#### NFS coté client 

Nous allons ajouter un ignition dans les fichiers de conf du vagrantfile. Pour se faire nous avons créé un fichier "config.ign" et nous avons entré les configs nécessaires à la configuration du NFS côté client.
```
{
  "ignition": {
    "config": {},
    "timeouts": {},
    "version": "2.1.0"
  },
  "networkd": {},
  "passwd": {},
  "storage": {},
  "systemd": {
    "units": [
      {
        "contents": "[Unit]\nBefore=remote-fs.target\n[Mount]\nWhat=192.168.56.101:/var/partage\nWhere=/var/toto\nType=nfs\n[Install]\nWantedBy=remote-fs.target",
        "enable": true,
        "name": "var-www.mount"
      }
    ]
  }
}
```

ou 192.168.56.101 est l'adresse du serveur, /var/partage le fichier à partager et /var/toto le dossier de destination sur les clients coreos.

Faites après un "vagrant up" et vous verrez dès les premières lignes la création du dossier /var/toto.

Q2 : Le principe d'un NFS est de partager des fichiers a travers le réseau. Cela permet d'eviter l'achat d'un NAS qui peut être plus cher que cette configuration. Les limites que cet outil peut atteindre est la limite de stockage d'un petit serveur comme celui ci.

Q3 : Nous pourrions tout mettre dans le vagrantfile afin d'automatiser tout le déploiement au démarrage.

### Registry

Nous montons donc un docker Registry.

Nous nous rendons sur le core-01 et lançons la commande suivante pour créer un nouveau registre.
```
docker service create --name registry --publish published=5000,target=5000 registry:2
```

Ce registre est joignable depuis tous les coreos que nous avons créer.

Nous pouvons donc pull et push des images sur et depuis notre registre pour avoir accès plus rapidement à nos images.

Lorsqu'on curl sur les hôtes on a un retour {} prouvant que tout fonctionne.

```
core@core-01 ~ $ curl 127.0.0.1/v2/
core@core-01 ~ $ {}

core@core-02 ~ $ curl 127.0.0.1/v2/
core@core-02 ~ $ {}
```





