# lab-s-creation-utilisateurs.
Pratique de la création et gestion des comptes utilisateurs(Internes et externes) et des groupes et ajouter des membres.
La gestion des identités dans Microsoft Entra ID (anciennement Azure Active Directory) est l’ensemble des services, processus et mécanismes permettant de créer, protéger, administrer et contrôler les identités numériques des utilisateurs, invités, applications et appareils au sein d’une organisation.

LAB 1 — CRÉATION DES UTILISATEURS
=================================

__Objectif__ : Créer 3 utilisateurs internes avec toutes les informations nécessaires.

__ACTIONS DANS LE PORTAIL :__
1. Connectez-vous au portail : https:https://portal.azure.com
2. Naviguez vers : Entra ID Tous les utilisateurs
3. Cliquez sur « Nouvel utilisateur »  « Créer un utilisateur » (pas inviter !)
4. Renseignez les informations suivantes pour chaque utilisateur :
   
__UTILISATEUR 1 : Alice Admin__
- Nom : Alice Admin
- Nom d’utilisateur : alice@firstaad56666874.onmicrosoft.com
- Groupe d’affichage : Utilisateurs
- Rôles : Aucun
- Mot de passe : générer automatiquement (forcer changement au premier login)
- Paramètres facultatifs :
• Département : Finance
• Poste : Administratrice Junior
• Manager : Aucun
<img width="1328" height="928" alt="Alice3" src="https://github.com/user-attachments/assets/3c6c7ca7-434c-4589-a202-79fe77e91f7c" />
<img width="1352" height="931" alt="Alice2" src="https://github.com/user-attachments/assets/a6e48cf8-5f03-4ba5-991b-2b75c893fd15" />
<img width="1456" height="919" alt="Alice1" src="https://github.com/user-attachments/assets/2870944c-5fb5-4008-bb69-c40dce3b3840" />


__UTILISATEUR 2 : Bob User__

- Nom : Bob User
- Nom d’utilisateur : bob@firstaad56666874.onmicrosoft.com
- Département : Marketing
- Poste : Collaborateur
- Mot de passe : généré automatiquement
  <img width="1309" height="957" alt="Bob3" src="https://github.com/user-attachments/assets/3845882c-ab2c-40f5-9f3a-fd6044f66205" />
<img width="1310" height="973" alt="Bob2" src="https://github.com/user-attachments/assets/4dcebe1e-5594-4b74-b118-196ee259ede8" />
<img width="1291" height="884" alt="Bob1" src="https://github.com/user-attachments/assets/6917c877-96a8-4550-923b-9813cf03b77b" />

  
__UTILISATEUR 3 : Claire Support__
- Nom : Claire Support
- Nom d’utilisateur : claire@firstaad56666874.onmicrosoft.com
- Département : IT
- Poste : Support Technique
- Mot de passe : généré automatiquement
  
  <img width="1275" height="961" alt="Claire2" src="https://github.com/user-attachments/assets/bde0ba8a-a264-4350-8481-0d997a04f640" />
<img width="1359" height="957" alt="Claire1" src="https://github.com/user-attachments/assets/dc50366a-a987-47f3-92b4-510e66ec5b21" />
<img width="1263" height="942" alt="Claire 3" src="https://github.com/user-attachments/assets/25673a19-0af9-4e57-a8bc-daf461c097d3" />


LAB 2 — CRÉATION DES GROUPES ET TYPES D’APPARTENANCE
====================================================
__Objectifs__ : Créer des groupes Sécurité, M365 et un groupe dynamique (licence P1).

__ACTIONS DANS LE PORTAIL :__
1. Naviguez vers entra ID - Groupes- < Tous les groupes>
2. Cliquez sur « Nouveau groupe »
   
__GROUPE 1 : Sécurité – Employés__

- Type de groupe : Sécurité
- Nom : SEC-Employes
- Description : Groupe regroupant tous les employés internes
- Type d’appartenance : Assigné
- Membres à ajouter : Alice, Bob, Claire
  <img width="1250" height="936" alt="SEC2" src="https://github.com/user-attachments/assets/847f18be-4412-47de-a194-51e3fa0fc437" />
<img width="1215" height="959" alt="SEC1" src="https://github.com/user-attachments/assets/7de525a7-e44e-439e-b378-234fa9cd89c8" />


  __GROUPE 2 : Microsoft 365 – Projet Alpha__
  
- Type : Microsoft 365
- Nom : M365-ProjetAlpha
- Description : Ressources collaboratives pour le projet Alpha
- Type d’appartenance : Assigné
- Membres : Alice, Bob
<img width="1320" height="968" alt="Alpha2" src="https://github.com/user-attachments/assets/330ab77d-1c0e-4b3a-b132-d5b4a58db8f6" />
<img width="1269" height="1040" alt="Alpha1" src="https://github.com/user-attachments/assets/feaff414-827c-4a3e-b706-5340a1e76c15" />

  
  
__GROUPE 3 : Groupe Dynamique IT__

- Type : Sécurité
- Nom : SEC-Dynamique-IT
- Appartenance : Dynamique (Utilisateur)
-  Règle dynamique :
(user.department -eq "IT")
- Résultat attendu : Claire rejoint automatiquement ce groupe.

  __LAB 3 — INVITATION D’UTILISATEURS EXTERNES (B2B)__
=====================================================

__Objectif__ : Inviter un utilisateur invité et lui attribuer des permissions.

ACTIONS :
1. Identité - Utilisateurs -Tous les utilisateurs - « Nouvel utilisateur »
2. Sélectionnez « Inviter un utilisateur externe »
3. Renseignez :
- Adresse email : sergedieudonnee@gmail.com
- Nom : Serge TOGNON
- Message : Bienvenue dans notre tenant de test
- Envoyer automatiquement l’invitation : OUI
4. Ajouter cet invité dans le groupe « M365-ProjetAlpha »
  
  <img width="1233" height="946" alt="serge5" src="https://github.com/user-attachments/assets/c9a32f89-8112-463c-8b65-0b588665fc45" />
<img width="1271" height="923" alt="serge4" src="https://github.com/user-attachments/assets/ed48d8c2-b0a5-422b-b72e-ef8fa1a8598b" />
<img width="1256" height="936" alt="serge3" src="https://github.com/user-attachments/assets/7b53d128-e5e2-4978-a856-26e7088294bc" />
<img width="1283" height="970" alt="serge2" src="https://github.com/user-attachments/assets/d4658e50-3e97-46f6-b4e9-f0c7869eb7e4" />
<img width="1309" height="927" alt="serge1" src="https://github.com/user-attachments/assets/71624451-7418-41b8-89a3-3feff8d487ce" />

     __TOUS LES UTILISATEURS ET GROUPES DE FIRST AAD__

<img width="1228" height="921" alt="users" src="https://github.com/user-attachments/assets/e8eb2ecd-f50e-4496-9492-f156941bd70e" />
<img width="1230" height="942" alt="grps" src="https://github.com/user-attachments/assets/6cd1d41b-ef53-46ce-8089-783c92e862a7" />

