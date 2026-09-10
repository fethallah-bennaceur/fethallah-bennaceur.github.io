---
layout: default
title: "Oracle AI DBA local"
description: "Agent IA local spécialisé dans le diagnostic Oracle 19c RAC fondé sur des preuves"
---

# Oracle AI DBA local

**Projet personnel — état mis à jour en septembre 2026**  
**Statut : prototype fonctionnel avancé — dernière baseline complète : 318 tests réussis**

## Présentation

Conception et développement d'un agent d'intelligence artificielle local spécialisé
dans l'administration et le diagnostic des bases **Oracle 19c RAC**.

L'objectif n'est pas de produire un simple chatbot capable de générer du SQL. L'agent
collecte d'abord des preuves techniques vérifiables, les structure, puis utilise un
modèle local pour établir un diagnostic DBA argumenté.

Le système fonctionne sur une machine virtuelle **Oracle Linux 8.10**. Le traitement
reste local grâce à **Ollama** et au modèle **Qwen** actuellement retenu après les essais
comparatifs du projet, sans envoyer les données Oracle à un service d'IA externe. Le tag
Ollama et la variante Qwen exacts restent à recopier depuis la configuration effective de
la VM.

## Principe d'architecture

| Couche | Rôle |
|---|---|
| Interface Web | Saisie des demandes, suivi progressif et reprise des conversations |
| FastAPI / Python | API, orchestration, streaming et contrôle du runtime |
| LLM et navigation dans les Skills | Compréhension de la demande, choix de la méthode DBA, des Skills et des outils |
| Outils Oracle en lecture seule | Collecte SQL, sessions, performances et informations RAC |
| SQLcl | Accès Oracle privilégié pour les requêtes contrôlées |
| ADR, alert logs et traces | Collecte factuelle des éléments de diagnostic Oracle |
| Evidence Engine / Evidence Store | Conservation, pagination et réutilisation des preuves volumineuses |
| Qwen via Ollama | Corrélation, raisonnement DBA et formulation du diagnostic |

## Réalisations principales

### Application et exploitation

- API **FastAPI/Uvicorn** installée comme service Linux.
- Interface Web avec affichage progressif en **SSE**.
- Gestion des demandes longues et annulation côté interface et serveur.
- Persistance et reprise des conversations.
- Contrôles de disponibilité de l'application, d'Oracle et du modèle local.

### Runtime agentique

- Boucle complète : question, sélection d'outil, collecte, réinjection des résultats et
  réponse finale.
- Le modèle joue le rôle de moteur de compréhension, de décision et de raisonnement DBA,
  tandis que Python orchestre les échanges et que les outils fournissent les preuves.
- Journal technique des appels d'outils pour faciliter l'audit et le diagnostic.
- Transport adaptatif des résultats selon leur taille.
- Continuation multi-contexte pour les investigations dépassant une seule fenêtre de
  contexte.

### Oracle et diagnostic

- Exécution de requêtes Oracle en **lecture seule** via SQLcl.
- Outils dédiés aux sessions, blocages, performances SQL et environnement RAC.
- Lecture contrôlée des alert logs, traces et fichiers ADR.
- Accès Linux/SSH limité aux sources nécessaires au diagnostic.
- Recherche d'erreurs Oracle à partir de sources documentaires autorisées, puis analyse
  par le modèle.
- Corrélation de plusieurs sources : données runtime, alert logs, traces et contexte
  documentaire.

### Skills Oracle

- Intégration de **163 ressources Oracle Skills** classées par domaines :
  administration, performance, monitoring, RAC, sauvegarde, sécurité, SQL, PL/SQL,
  architecture et exploitation.
- Ajout de Skills locaux spécialisés dans l'analyse des alert logs.
- Chargement progressif : seuls les Skills pertinents sont injectés dans le contexte.
- Découverte dynamique des articles spécialisés et navigation dans leurs références.

### Alert logs, ADR et Skills de diagnostic Oracle

- Création du Skill local principal `alert-log-analysis/SKILL.md`, séparé des Skills
  Oracle officiels et chargé uniquement pour les incidents concernés.
- Diagnostic des événements Oracle Database, ASM, ACFS et Clusterware dans un
  environnement Oracle 19c Extended RAC.
- Analyse fondée sur les alert logs et traces ADR réellement collectés, avec règles
  strictes de grounding : aucun événement, incident ou lien de causalité n'est inventé.
- Organisation en **7 références spécialisées**, chargées progressivement :
  - erreurs Oracle et incidents ADR ;
  - redo, log switches et checkpoints ;
  - archivage et Fast Recovery Area ;
  - cycle de vie des instances ;
  - processus Oracle d'arrière-plan ;
  - événements RAC étendus : GCS/GES, évictions, interconnexion et quorum ;
  - ASM et événements de stockage.
- Routage « Route, Don't Flood » : le Skill principal sélectionne uniquement la
  référence utile au lieu de charger toute la documentation dans le contexte.
- Validation réelle du routage jusqu'à une référence spécialisée, avec chargement du
  routeur et de l'article ciblé confirmé.

### Diagnostic système Linux et Skill système

- Création et intégration du Skill local `linux-system-analysis/SKILL.md`.
- Collecte contrôlée des indicateurs système avec `top`, `vmstat` et `df`.
- Analyse CPU, load average, run queue, mémoire, swap, I/O wait et systèmes de
  fichiers.
- Mise en relation des symptômes Linux avec les preuves Oracle, sans confondre
  corrélation temporelle et cause démontrée.
- Commandes limitées à une liste d'opérations de diagnostic en lecture seule.
- Validation du Skill dans le catalogue, de son routage GPT et des collectes système
  réelles.

### Gestion des preuves

- **Evidence Store lossless** : conservation intégrale des résultats volumineux.
- Identifiant de preuve, empreinte SHA-256 et métadonnées de traçabilité.
- Lecture paginée avec offsets, indicateur de complétude et reprise contrôlée.
- Protection contre les relectures identiques et les boucles inutiles.
- Transmission des preuves entre plusieurs contextes sans perdre la source technique.

### Sécurité et gouvernance

- Philosophie **Evidence First — Reasoning Second**.
- Distinction explicite entre faits, interprétations, hypothèses et éléments à vérifier.
- Lecture seule par défaut ; aucune réparation automatique de la base ou du système.
- Garde SQL et contrôle des opérations potentiellement dangereuses.
- Contrôles fondamentaux des fonctions dépendant de licences Oracle particulières ;
  l'audit exhaustif de tous les usages reste en cours.
- MCP désactivé par défaut : les outils DBA spécialisés restent prioritaires.
- Aucune invention autorisée de Bug ID, note MOS, patch, SQL_ID ou preuve runtime.

## Validation

### Dernière baseline entièrement démontrée sur la VM

La dernière baseline complète documentée comprend :

- **318 tests réussis** ;
- service applicatif actif ;
- communication Oracle validée ;
- modèle Ollama opérationnel ;
- 163 ressources Skills détectées ;
- Evidence Store, transport adaptatif et continuation multi-contexte testés.
- routage du Skill `alert-log-analysis` et d'une référence spécialisée validé ;
- Skill `linux-system-analysis` et collectes `top`, `vmstat`, `df` validés en lecture
  seule.

Les tests couvrent notamment le routage, les politiques de sécurité, les outils Oracle,
le stockage des preuves, la pagination, les conversations et le runtime agentique.

### Correctif préparé mais pas encore intégré à la baseline officielle

Le correctif `oracle-ai-dba-skill-evidence-completion-fix.patch` impose la lecture
intégrale d'un Skill externalisé jusqu'à `complete=true` avant d'autoriser un outil métier
ou une réponse finale. Il a passé **39 tests ciblés localement**, mais son application, la
suite complète et le test runtime sur la VM ne sont pas encore démontrés. Il n'est donc
pas comptabilisé dans la baseline officielle de 318 tests.

## Compétences démontrées

`Oracle 19c RAC` · `DBA Oracle` · `Python` · `FastAPI` · `SQLcl` · `Linux` ·
`Ollama` · `Qwen` · `IA agentique` · `SSE` · `SSH/SFTP` · `ADR` ·
`Alert logs` · `ADR` · `Diagnostic Linux` · `Tests automatisés` ·
`Sécurité read-only` · `Architecture logicielle`

## État actuel et travail restant

### Déjà réalisé et validé

- Fondations fonctionnelles de l'application, de l'interface et du runtime agentique.
- Saisie longue, streaming SSE, affichage progressif et annulation serveur.
- Persistance et reprise des conversations.
- Exécution Oracle en lecture seule via SQLcl et outils de diagnostic Oracle/Linux.
- Chargement progressif des Skills Oracle et locaux, routage vers un article spécialisé
  et découverte dynamique des chemins réels.
- Evidence Store lossless, pagination adaptative, prévention des relectures et
  continuation multi-contexte.
- Dernière suite complète démontrée : **318 tests réussis, 2 avertissements**.
- Environnement Oracle RAC validé : base `ORCL`, instances `ORCL1` et `ORCL2`.
- Health check applicatif observé sur la connexion à `ORCL1` : application, Oracle,
  Ollama et 163 ressources Skills opérationnels ; MCP désactivé. Ce health check confirme
  l'instance utilisée par la connexion, mais ne constitue pas l'inventaire complet des
  instances RAC.

### Étape actuellement en cours

Le projet se trouve dans la phase d'amélioration de la **qualité et de la fiabilité des
réponses**. Le travail immédiat concerne la complétude obligatoire des Skills
externalisés.

### Prochaine action exacte

Appliquer sur la VM le correctif de complétude des Skills, puis obtenir les quatre preuves
suivantes :

1. application du patch sans erreur ni fuzz ;
2. réussite des tests ciblés sur la VM ;
3. réussite de la suite complète ;
4. test runtime montrant toutes les lectures jusqu'à `complete=true` avant tout outil
   métier ou toute réponse finale.

### Après cette validation

- Exécuter les scénarios de référence : erreurs ORA, alert logs ORCL1/ORCL2, performance
  SQL, RAC/ASM et diagnostic Linux.
- Vérifier la complétude des preuves, la couverture multi-instance et la séparation entre
  faits, hypothèses et recommandations.
- Terminer l'audit exhaustif des `DBMS_*`, des licences Oracle et des autorisations
  d'actions modifiantes.
- Récupérer une archive complète correspondant exactement au code installé sur la VM et
  confirmer le tag Qwen actif.
- Consolider la documentation, corriger ou documenter les deux avertissements FastAPI,
  puis figer une version Git stable de la V1.

## Positionnement

Ce projet illustre la conception d'un outil DBA moderne combinant expertise Oracle,
automatisation Python et intelligence artificielle locale, avec une priorité donnée à
la sécurité, à la traçabilité et aux preuves techniques.

---

**Réalisation : Fethallah Bennaceur — DBA Oracle**
