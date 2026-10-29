# TP 3 : Premiers conteneurs avec Docker

Compte rendu du TP 3 du cours **Introduction au Cloud** (L3 SI RIST & BDGL), encadré par le Dr. Bakayoko.

**Trinôme :** TRAORE Ibrahim, KOFFI Boni, DJANGORAN Christian

## Objectif

Comprendre ce qu'est un conteneur, le comparer à une machine virtuelle, puis conteneuriser et publier une petite API.

## Ce qui a été réalisé

- Installation de Docker sur une VM Ubuntu
- Lancement et inspection de conteneurs (`hello-world`, Nginx)
- Conteneur éphémère et données persistantes avec un volume
- Comparaison conteneur / VM (taille, démarrage, mémoire)
- Création d'une API Flask, d'un Dockerfile et d'une image en couches
- Publication de l'image sur Docker Hub, puis nettoyage

## Image publiée

```bash
docker run -d -p 5000:5000 ibrahimbdgl/api-fil-rouge:1.1
```

Puis ouvrir http://localhost:5000

## Outils

Windows 11 · VirtualBox · Ubuntu Server 26.04 LTS · Docker 29.1 · Python / Flask · Docker Hub

## Contenu

Le fichier [TP3.md](TP3.md) contient les commandes, leurs sorties, le tableau comparatif, les réponses aux questions et la conclusion.
