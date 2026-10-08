# Assistant FAQ - Médiathèque Les Tilleuls

> Projet IA — BTS SIO 2 SLAM — Alexandre Poulenard — octobre 2026
> **URL publique** : https://according-proceed-gmc-bases.trycloudflare.com — code d'accès envoyé à l'enseignant par e-mail

## 1. Concevoir

**Sujet choisi** : n°1 - FAQ médiathèque

La médiathèque municipale Les Tilleuls reçoit chaque semaine de nombreuses demandes répétées : nombre de documents autorisés, durée d'emprunt, frais de retard, horaires d'ouverture, accès au wifi et règles de sécurité. Aujourd'hui, les agents doivent répondre manuellement à des questions simples, ce qui entraîne des pertes de temps et une plus grande chance d'erreur.

L'application permet de répondre rapidement aux questions courantes du règlement, en restant strictement fidèle aux règles de la médiathèque et en citant les articles utilisés. Cela aide les usagers à trouver l'information immédiatement, sans dépendre d'un agent pour chaque demande standard.

**Trois cas d'usage** :

1. En tant qu'usager, je veux connaître le nombre maximum de documents que je peux emprunter, afin de ne pas dépasser les limites du règlement.
2. En tant que lecteur, je veux savoir combien me coûte un retard de retour, afin de calculer la pénalité prévue par l'article 6.
3. En tant qu'agent d'accueil, je veux vérifier une réponse sur un point précis du règlement, afin de garantir une information cohérente et juste.

**Ce que l'application ne fait pas** :

- Elle ne peut pas accéder aux comptes personnels ni aux données internes d'un usager.
- Elle ne répond pas aux questions hors sujet, comme des demandes culinaires, techniques ou générales.
- Elle ne crée pas de règle absente du règlement : si une information manque, elle indique précisément que la réponse n'est pas disponible dans le texte.

## 2. Le modèle et la machine

| | |
|---|---|
| Carte graphique et mémoire vidéo (VRAM) | Nvidia RTX 4070ti |
| Mémoire vive | 12 Go |
| Modèle retenu | `qwen2.5:3b` |
| Pourquoi celui-là | le modèle est assez léger pour tourner sans GPU, tout en restant suffisamment fiable pour traiter le règlement et les consignes de refus |
| Modèle comparé | `qwen2.5:1.5b` |

J'ai choisi un modèle compact pour limiter le coût et la latence, tout en gardant une bonne robustesse sur les cas réglementaires, les calculs et les demandes hors sujet.

## 3. Piloter — le journal

| Séance | Ce qui est fait | Ce qui a bloqué, et comment c'est réglé |
|---|---|---|
| Lundi 05/10 | Analyse du sujet n°1, lecture du règlement complet, écriture de la consigne, préparation de la structure du projet et premiers tests | Le modèle inventait parfois des règles absentes du règlement ; j'ai limité la réponse au texte juridique et imposé la citation des articles |
| Mardi 06/10 | Tests sur les questions pièges, ajout des cas d'évaluation dans `cas.json`, réglage de la sortie et validation du refus des demandes hors sujet | Les demandes de détourner les consignes ou d'accéder à des informations personnelles posaient problème ; j'ai renforcé la consigne avec des refus explicites |

## 4. Mesurer

J'ai préparé un jeu de 10 cas dans `cas.json`, avec au moins deux hors sujet et un cas de tentative de détourner les consignes.

| | Modèle retenu | Modèle comparé |
|---|---|---|
| Réussite (sur 10 cas × 3 essais) | 67 % | … % |
| Temps de réponse médian | 2.3 s | … s |

**Ce que les échecs montrent** :

Le principal échec venait des calculs de pénalités : le modèle ne traduisait pas toujours correctement le plafond de 5 € par document. J'ai donc renforcé la consigne pour demander un calcul explicite et une citation de l'article 6.

Un deuxième échec concernait les questions hors sujet ou les demandes d'informations personnelles. Le modèle pouvait répondre trop facilement. J'ai ajouté une règle stricte : ne répondre qu'à partir du règlement, refuser poliment les demandes non couvertes et ne jamais prétendre avoir accès à un compte usager.

## 5. Sécuriser

| Risque | Ce qui pourrait arriver | Mesure prise dans le projet |
|---|---|---|
| L'URL est publique | n'importe qui utilise votre PC et votre électricité | code d'accès obligatoire sur la route `/api/demander`, limite de requêtes par visiteur |
| Ollama exposé | le modèle devient accessible à des tiers et peut être utilisé sans contrôle | le service est utilisé localement et l'application est protégée par un code d'accès |
| Détournement des consignes | l'utilisateur demande d'ignorer le règlement ou d'inventer des réponses | la consigne impose de ne répondre qu'à partir du document, de citer les articles et de refuser les demandes hors sujet |
| Données personnelles | l'assistant prétend connaître le compte ou les prêts d'un usager | aucune donnée personnelle n'est stockée ni accessible par l'application |
| Secrets dans le dépôt | des identifiants ou codes peuvent être exposés publiquement | le fichier `.env` est conservé hors du dépôt et les réglages sensibles restent séparés du code source |

```bash
>>> docker compose ps
NAME                 IMAGE                           COMMAND                  SERVICE   CREATED             STATUS                       PORTS
projet_ia-app-1      projet_ia-app                   "uvicorn main:app --…"   app       About an hour ago   Up About an hour (healthy)   127.0.0.1:8000->8000/tcp
```
- Seul 127.0.0.1:8000 est publié, le port 11434 n'apparaît pas ;

## 6. Mettre en production — comment refaire

```bash
cp .env.example .env      # puis remplir
docker compose up -d --build
docker compose ps
docker compose logs tunnel
```

## 7. Usage de l'IA pendant le projet

J'ai utilisé l'IA pour reformuler la consigne, tester plusieurs variantes de réponse, vérifier les cas hors sujet et corriger les erreurs de calcul ou d'invention de règles. J'ai gardé les éléments utiles au projet, puis modifié ou refusé les propositions qui contredisaient le règlement ou introduisaient des informations non vérifiables.

