## GitLab Runner local

Un GitLab Runner permet d’exécuter les jobs CI/CD GitLab.

Dans cet exemple, le runner est installé localement sur la machine avec l’exécuteur Docker.

Architecture :

```text
GitLab -> Runner local -> Conteneur Docker -> Exécution du job
```

## Installation

### Installer GitLab Runner

```bash
curl -L --output gitlab-runner_amd64.deb \
https://gitlab-runner-downloads.s3.amazonaws.com/latest/deb/gitlab-runner_amd64.deb

sudo dpkg -i gitlab-runner_amd64.deb
```

### Enregistrer le runner

```bash
gitlab-runner register
```

Choisir :

* executor : `docker`
* image : `busybox:1.36.1`
* tag : `local`

## Exemple `.gitlab-ci.yml`

```yaml
stages:
  - test

test-local:
  stage: test
  tags:
    - local
  image: busybox:1.36.1
  script:
    - echo "Runner local OK"
    - whoami
    - hostname
```

## Résultat

GitLab envoie le job au runner ayant le tag `local`.

Le runner démarre un conteneur Docker puis exécute le script CI/CD.

---

# Version détaillée README GitHub

# GitLab Runner Local Docker

## Objectif

Ce projet montre comment utiliser un GitLab Runner local avec l’exécuteur Docker afin d’exécuter des pipelines GitLab CI/CD sans infrastructure Kubernetes/OpenShift.

---

# Architecture

```text
                +-------------------+
                |      GitLab       |
                +-------------------+
                          |
                          |
                    Pipeline Job
                          |
                          v
              +----------------------+
              | GitLab Runner Local  |
              | Executor: Docker     |
              +----------------------+
                          |
                          |
                  Docker Container
                          |
                          v
                 Exécution du script
```

---

# Prérequis

* Docker installé
* GitLab Runner installé
* Accès à une instance GitLab
* Un projet GitLab

---

# Installation du GitLab Runner

## Ubuntu / Debian

```bash
curl -L --output gitlab-runner_amd64.deb \
https://gitlab-runner-downloads.s3.amazonaws.com/latest/deb/gitlab-runner_amd64.deb

sudo dpkg -i gitlab-runner_amd64.deb
```

Vérification :

```bash
gitlab-runner --version
```

---

# Création du Runner dans GitLab

Dans GitLab :

```text
Project -> Settings -> CI/CD -> Runners
```

Créer un nouveau runner puis récupérer :

* URL GitLab
* Authentication token

---

# Enregistrement du Runner

```bash
gitlab-runner register
```

Exemple :

```text
Enter the GitLab instance URL:
https://gitlab.example.com

Enter the registration token:
glrt-xxxxxxxx

Enter a description:
runner-local-zakaria

Enter tags:
local

Enter an executor:
docker

Default Docker image:
busybox:1.36.1
```

---

# Vérification

Afficher les runners enregistrés :

```bash
gitlab-runner list
```

---

# Configuration GitLab CI/CD

Créer le fichier `.gitlab-ci.yml`

```yaml
stages:
  - test

test-local:
  stage: test
  tags:
    - local
  image: busybox:1.36.1
  script:
    - echo "Runner local OK"
    - echo "Current user:"
    - whoami
    - echo "Hostname:"
    - hostname
```

---

# Fonctionnement des tags

Le champ :

```yaml
tags:
  - local
```

permet à GitLab de sélectionner le runner possédant le tag `local`.

Sans tag, GitLab peut utiliser :

* un shared runner
* un group runner
* un instance runner

---

# Exécution du pipeline

Push du projet :

```bash
git add .
git commit -m "test runner local"
git push
```

Le pipeline démarre automatiquement.

---

# Vérification du runner utilisé

Dans les logs GitLab :

```text
Running with gitlab-runner ...
on runner-local-zakaria
Preparing the "docker" executor
```

Si vous voyez :

```text
Preparing the "kubernetes" executor
```

alors GitLab utilise encore un shared runner Kubernetes/OpenShift.

---

# Démarrage du runner

Mode manuel :

```bash
gitlab-runner run
```

Mode service :

```bash
sudo gitlab-runner start
```

---

# Configuration du runner

Fichier :

```bash
~/.gitlab-runner/config.toml
```

Exemple :

```toml
[[runners]]
  name = "runner-local-zakaria"
  url = "https://gitlab.example.com"
  token = "glrt-xxxx"
  executor = "docker"

  [runners.docker]
    image = "busybox:1.36.1"
    privileged = false
```

---

# Différence entre les runners

| Type            | Description                             |
| --------------- | --------------------------------------- |
| Project runner  | Disponible uniquement pour un projet    |
| Group runner    | Partagé entre les projets d’un groupe   |
| Instance runner | Disponible pour toute l’instance GitLab |

---

# Conclusion

Le runner local Docker permet :

* de tester rapidement GitLab CI/CD
* de développer des pipelines localement
* d’éviter une infrastructure Kubernetes/OpenShift pour les tests simples
* d’isoler les jobs dans des conteneurs Docker
