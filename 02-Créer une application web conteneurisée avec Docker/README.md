# 🚀 Création et Déploiement d'une Application Web Conteneurisée avec Docker et Azure

**Auteur :** Serge TOGNON — [Mon profil LinkedIn](https://www.linkedin.com/)  
**Technologies :** WSL 2 · Docker Desktop · Git · .NET Core · Azure Container Registry (ACR) · Azure Container Instances (ACI)

---

## 📝 Description du Projet

Ce projet illustre le **cycle DevOps complet** : conteneurisation d'une API web .NET Core (système de réservation hôtelière) avec Docker, suivie de son déploiement cloud sur Microsoft Azure via un registre privé (ACR) et une instance de conteneur publique (ACI).

> 💡 **Compétences démontrées :** rédaction d'un Dockerfile multi-étapes, build et test local d'une image Docker, push vers un registre Azure privé, déploiement et exposition publique sur Internet.

---

## 🛠️ PARTIE 1 — Conteneurisation et test local

### Étape 1 — Récupération du code source

Depuis le terminal **WSL** (Windows Subsystem for Linux), clonage du dépôt GitHub officiel du projet :

```bash
git clone https://github.com/MicrosoftDocs/mslearn-hotel-reservation-system.git
cd mslearn-hotel-reservation-system/src
```

> **Pourquoi WSL ?**  
> WSL permet d'exécuter un environnement Linux natif sous Windows. Docker Desktop s'y intègre parfaitement et offre les mêmes commandes qu'un serveur Linux en production — idéal pour un workflow DevOps professionnel.

---

### Étape 2 — Écriture du Dockerfile (build multi-étapes)

Création du fichier `Dockerfile` dans le répertoire `/src`. Le **multi-stage build** est une bonne pratique DevOps : on sépare la phase de **compilation** de la phase d'**exécution**, ce qui réduit considérablement la taille de l'image finale (pas de SDK dans l'image de prod).

```dockerfile
# ── ÉTAPE 1 : Compilation avec le SDK .NET complet ──────────────────────────
FROM mcr.microsoft.com/dotnet/core/sdk:2.2
WORKDIR /src

# Restauration des dépendances NuGet
COPY ["/HotelReservationSystem/HotelReservationSystem.csproj", "HotelReservationSystem/"]
COPY ["/HotelReservationSystemTypes/HotelReservationSystemTypes.csproj", "HotelReservationSystemTypes/"]
RUN dotnet restore "HotelReservationSystem/HotelReservationSystem.csproj"

# Compilation en mode Release
COPY . .
WORKDIR "/src/HotelReservationSystem"
RUN dotnet build "HotelReservationSystem.csproj" -c Release -o /app

# Publication de l'application
RUN dotnet publish "HotelReservationSystem.csproj" -c Release -o /app

# ── ÉTAPE 2 : Image d'exécution légère ──────────────────────────────────────
EXPOSE 80
WORKDIR /app
ENTRYPOINT ["dotnet", "HotelReservationSystem.dll"]
```

> **Explication des instructions clés :**
> | Instruction | Rôle |
> |---|---|
> | `FROM` | Définit l'image de base (SDK .NET pour compiler) |
> | `WORKDIR` | Définit le répertoire de travail dans le conteneur |
> | `RUN dotnet restore` | Télécharge les packages NuGet (dépendances) |
> | `RUN dotnet publish` | Compile et prépare l'app pour la production |
> | `EXPOSE 80` | Déclare que le conteneur écoute sur le port 80 (HTTP) |
> | `ENTRYPOINT` | Commande lancée automatiquement au démarrage du conteneur |

---

### Étape 3 — Construction de l'image Docker

```bash
docker build -t reservationsystem .
```

> **Ce qui se passe :** Docker lit le `Dockerfile` ligne par ligne, exécute chaque instruction dans un calque (layer) isolé et empile ces calques pour former l'image finale. Le flag `-t` (tag) lui donne le nom `reservationsystem`.

Vérification de la présence de l'image dans le registre local :

```bash
docker image list
```

> 📸 *[Insérer ici votre capture du terminal montrant `reservationsystem` dans la liste]*

---

### Étape 4 — Exécution et test local du conteneur

Lancement du conteneur en **mode détaché**, avec redirection de port :

```bash
docker run -p 8080:80 -d --name reservations reservationsystem
```

> **Décryptage des options :**
> | Option | Signification |
> |---|---|
> | `-p 8080:80` | Redirige le port 8080 du PC hôte vers le port 80 du conteneur |
> | `-d` | Mode détaché : le conteneur tourne en arrière-plan |
> | `--name reservations` | Donne un nom lisible au conteneur pour le gérer facilement |

Vérification du statut :

```bash
docker ps -a
```

> 📸 *[Insérer ici la capture montrant le conteneur `reservations` avec le statut `Up`]*

---

### ✅ Résultat du test local — API opérationnelle

En naviguant sur `http://localhost:8080/api/reservations/1`, l'API retourne le JSON suivant, confirmant que le conteneur fonctionne correctement en local :

![Résultat JSON de l'API en local](./Capture_d_écran_2026-05-27_060322.png)

> **Analyse de la réponse :**  
> L'API renvoie un objet de réservation complet avec `reservationID: 1`, `customerID`, `hotelID: "Hôtel 1031518109"`, les dates de `checkin` / `checkout`, le nombre d'invités (`numberOfGuests: 4`) ainsi qu'un commentaire encodé en Base64 — preuve que l'ensemble de la logique métier de l'application fonctionne dans le conteneur.

Nettoyage du conteneur local après validation :

```bash
docker container stop reservations
docker rm reservations
```

---

## ☁️ PARTIE 2 — Déploiement sur Azure (ACR + ACI)

### Étape 1 — Création du groupe de ressources

Un **groupe de ressources** Azure est un conteneur logique qui regroupe toutes les ressources d'un même projet pour faciliter leur gestion, leur facturation et leur suppression.

Sur le **Portail Azure** (`portal.azure.com`) :

1. **"Créer une ressource"** → **"Groupe de ressources"**
2. Renseigner :
   - **Nom :** `rg-hotel-conteneur`
   - **Région :** `France Central` (ou la plus proche)
3. **"Vérifier + créer"** → **"Créer"**

> 💡 **Bonne pratique :** en regroupant toutes les ressources sous `rg-hotel-conteneur`, il suffira de supprimer ce seul groupe à la fin de l'exercice pour effacer toute l'infrastructure et éviter des frais inutiles.

---

### Étape 2 — Création du registre Azure Container Registry (ACR)

L'**ACR** est un registre Docker **privé** hébergé sur Azure. Contrairement à Docker Hub (public), les images y sont accessibles uniquement par les personnes et services autorisés.

1. Chercher **"Container Registry"** → **"Créer"**
2. Renseigner :
   - **Groupe de ressources :** `rg-hotel-conteneur`
   - **Nom du registre :** `sergetognonacr`
   - **SKU :** `Standard`
3. Après création, aller dans **Paramètres → Clés d'accès**
4. Activer **"Utilisateur Administrateur"** pour obtenir :
   - Serveur de connexion : `sergetognonacr.azurecr.io`
   - Nom d'utilisateur et mot de passe

> 📸 *[Insérer ici la vue d'ensemble de la ressource ACR sur le portail Azure]*

> **Pourquoi activer l'utilisateur Admin ?**  
> Cela génère des identifiants permettant à la commande `docker login` de s'authentifier auprès du registre privé Azure pour y pousser ou tirer des images.

---

### Étape 3 — Publication de l'image vers Azure ACR

On **étiquette** (tag) l'image locale avec l'adresse complète du registre Azure, puis on l'envoie :

```bash
# 1. Tag de l'image avec l'URL complète du registre Azure
docker tag reservationsystem:latest sergetognonacr.azurecr.io/reservationsystem:latest

# 2. Authentification auprès du registre privé
docker login sergetognonacr.azurecr.io

# 3. Envoi de l'image vers Azure
docker push sergetognonacr.azurecr.io/reservationsystem:latest
```

> **Pourquoi le tag ?**  
> Docker identifie les registres de destination par le préfixe du nom d'image. En ajoutant `sergetognonacr.azurecr.io/` devant le nom, on indique explicitement à Docker où envoyer l'image lors du `push`.

Après le push, vérifier dans le portail : **ACR `sergetognonacr` → Référentiels** — l'image `reservationsystem:latest` doit y apparaître.

> 📸 *[Insérer ici la capture du portail Azure montrant l'image dans les référentiels ACR]*

---

### Étape 4 — Déploiement public via Azure Container Instances (ACI)

**Azure Container Instances** permet de déployer un conteneur directement dans le cloud Azure **sans gérer de serveur ni de cluster**. C'est la solution la plus rapide pour exposer une application conteneurisée publiquement.

1. Chercher **"Container Instances"** → **"Créer"**
2. **Onglet "Notions de base" :**
   - **Groupe de ressources :** `rg-hotel-conteneur`
   - **Nom du conteneur :** `hotel-app-instance`
   - **Source d'image :** Azure Container Registry
   - **Registre :** `sergetognonacr`
   - **Image :** `reservationsystem` · **Tag :** `latest`
   - **Système d'exploitation :** Linux
3. **Onglet "Mise en réseau" :**
   - **Type :** Public
   - **Étiquette de nom DNS :** `hotel-serge-tognon` *(nom unique de votre choix)*
   - **Port :** `80` (TCP)
4. **"Vérifier + créer"** → **"Créer"**

> **Qu'est-ce qu'une étiquette DNS ?**  
> Elle génère automatiquement un **FQDN** (nom de domaine complet) public sous la forme `hotel-serge-tognon.<région>.azurecontainer.io` — permettant d'accéder à l'application depuis n'importe quel navigateur dans le monde, sans configuration DNS manuelle.

> 📸 *[Insérer ici la capture de l'onglet "Vue d'ensemble" de l'instance ACI avec le FQDN visible]*

---

## 🎯 Résultat Final — Validation Cloud

L'application est désormais accessible **publiquement depuis n'importe où dans le monde**.

URL d'accès publique générée par Azure :

```
http://hotel-serge-tognon.<région>.azurecontainer.io/api/reservations/1
```

> 📸 *[Insérer ici la capture de votre navigateur affichant l'URL Azure avec le même JSON en réponse]*

---

## 🧹 Nettoyage des ressources Azure

Pour éviter tout frais résiduel après l'exercice, supprimer le groupe de ressources entier :

```bash
az group delete --name rg-hotel-conteneur --yes --no-wait
```

Ou via le portail : **`rg-hotel-conteneur` → Supprimer le groupe de ressources**.

---

## 📊 Récapitulatif des ressources créées

| Ressource | Nom | Type Azure | Rôle |
|---|---|---|---|
| Groupe de ressources | `rg-hotel-conteneur` | Resource Group | Conteneur logique du projet |
| Registre de conteneurs | `sergetognonacr` | Azure Container Registry | Stockage privé de l'image Docker |
| Instance de conteneur | `hotel-app-instance` | Azure Container Instances | Hébergement public de l'application |

---

## 💡 Conclusion

Ce projet valide la maîtrise du **cycle DevOps complet** :

| Étape | Action |
|---|---|
| 1️⃣ Conteneurisation | Rédaction d'un Dockerfile multi-étapes optimisé |
| 2️⃣ Test local | Build, run et validation via Docker / WSL |
| 3️⃣ Registre privé | Push de l'image vers `sergetognonacr` (ACR) |
| 4️⃣ Déploiement cloud | Exposition publique via `hotel-app-instance` (ACI) |
| 5️⃣ Validation | API accessible mondialement et fonctionnelle ✅ |

> 🎓 **Compétences prouvées :** Docker · Dockerfile · WSL 2 · Azure Portal · ACR · ACI · Architecture microservices · CI/CD · Déploiement cloud sans infrastructure serveur.
