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
