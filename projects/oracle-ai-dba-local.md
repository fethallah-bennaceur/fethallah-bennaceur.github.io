---
layout: default
title: "Oracle AI DBA local"
description: "Agent IA local spécialisé dans le diagnostic Oracle 19c RAC fondé sur des preuves"
---

# Oracle AI DBA local

**Projet personnel — août 2026**  
**Statut : prototype fonctionnel avancé — 318 tests automatisés validés**

## Présentation

Conception et développement d'un agent d'intelligence artificielle local spécialisé
dans l'administration et le diagnostic des bases **Oracle 19c RAC**.

L'objectif n'est pas de produire un simple chatbot capable de générer du SQL. L'agent
collecte d'abord des preuves techniques vérifiables, les structure, puis utilise un
modèle local pour établir un diagnostic DBA argumenté.

Le système fonctionne sur une machine virtuelle **Oracle Linux 8.10**. Le traitement
reste local grâce à **Ollama** et **GPT-OSS 20B**, sans envoyer les données Oracle à un
service d'IA externe.

## Principe d'architecture

| Couche | Rôle |
|---|---|
| Interface Web | Saisie des demandes, suivi progressif et reprise des conversations |
| FastAPI / Python | API, orchestration, streaming et contrôle du runtime |
| Routeurs d'intention et de Skills | Sélection de la méthode DBA et des outils adaptés |
| Outils Oracle en lecture seule | Collecte SQL, sessions, performances et informations RAC |
| SQLcl | Accès Oracle privilégié pour les requêtes contrôlées |
| ADR, alert logs et traces | Collecte et analyse des éléments de diagnostic Oracle |
| Evidence Engine / Evidence Store | Conservation, pagination et réutilisation des preuves volumineuses |
| GPT-OSS 20B via Ollama | Corrélation, raisonnement et formulation du diagnostic |

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
- Le modèle joue le rôle de moteur de décision, tandis que les outils déterministes
  fournissent les mesures.
- Journal technique des appels d'outils pour faciliter l'audit et le diagnostic.
- Transport adaptatif des résultats selon leur taille.
- Continuation multi-contexte pour les investigations dépassant une seule fenêtre de
  contexte.

### Oracle et diagnostic

- Exécution de requêtes Oracle en **lecture seule** via SQLcl.
- Outils dédiés aux sessions, blocages, performances SQL et environnement RAC.
- Lecture contrôlée des alert logs, traces et fichiers ADR.
- Accès Linux/SSH limité aux sources nécessaires au diagnostic.
- Recherche et analyse d'erreurs Oracle à partir de sources documentaires autorisées.
- Corrélation de plusieurs sources : données runtime, alert logs, traces et contexte
  documentaire.

### Skills Oracle

- Intégration de **163 ressources Oracle Skills** classées par domaines :
  administration, performance, monitoring, RAC, sauvegarde, sécurité, SQL, PL/SQL,
  architecture et exploitation.
- Ajout de Skills locaux spécialisés dans l'analyse des alert logs.
- Chargement progressif : seuls les Skills pertinents sont injectés dans le contexte.
- Découverte dynamique des articles spécialisés et navigation dans leurs références.

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
- Validation explicite pour les fonctions dépendant de licences Oracle particulières.
- MCP désactivé par défaut : les outils DBA spécialisés restent prioritaires.
- Aucune invention autorisée de Bug ID, note MOS, patch, SQL_ID ou preuve runtime.

## Validation

La dernière baseline complète documentée comprend :

- **318 tests réussis** ;
- service applicatif actif ;
- communication Oracle validée ;
- modèle Ollama opérationnel ;
- 163 ressources Skills détectées ;
- Evidence Store, transport adaptatif et continuation multi-contexte testés.

Les tests couvrent notamment le routage, les politiques de sécurité, les outils Oracle,
le stockage des preuves, la pagination, les conversations et le runtime agentique.

## Compétences démontrées

`Oracle 19c RAC` · `DBA Oracle` · `Python` · `FastAPI` · `SQLcl` · `Linux` ·
`Ollama` · `GPT-OSS` · `IA agentique` · `SSE` · `SSH/SFTP` · `ADR` ·
`Tests automatisés` · `Sécurité read-only` · `Architecture logicielle`

## Prochaines évolutions

- Finaliser la lecture obligatoire et complète des Skills volumineux.
- Terminer l'audit global des fonctions Oracle soumises à licences ou restrictions.
- Renforcer les évaluations de qualité et la couverture des diagnostics RAC complexes.
- Consolider la version stable, l'observabilité et la documentation d'exploitation.

## Positionnement

Ce projet illustre la conception d'un outil DBA moderne combinant expertise Oracle,
automatisation Python et intelligence artificielle locale, avec une priorité donnée à
la sécurité, à la traçabilité et aux preuves techniques.

---

**Réalisation : Fethallah Bennaceur — DBA Oracle**
