# Organisation du dépôt Thanatos

## Contenu du dépôt

| Chemin | Rôle |
|---|---|
| `index.html` | L'application complète, en un seul fichier (styles, interface, logique). Servie telle quelle par GitHub Pages depuis la branche `main`. |
| `README.md` | Présentation courte du dépôt. |
| `docs/Doctrine-Thanatos.pdf` | Doctrine de travail complète (v1.0, 25 octobre 2026, 9 pages) : document de référence. |
| `docs/Doctrine-Thanatos.txt` | Texte extrait du PDF, pour la recherche et la relecture des différences dans GitHub. |
| `docs/prompt-systeme-thanatos.txt` | Version courte (environ 1 000 mots) : la consigne système commune aux trois niveaux. |
| `docs/ORGANISATION.md` | Ce document. |

Il n'y a ni serveur, ni étape de compilation : tout s'exécute dans le navigateur. La clé de connexion n'est jamais dans le dépôt : elle est saisie dans les réglages et reste dans le stockage local de l'appareil.

## Plan de `index.html`

1. **Styles** (`<style>` en tête du fichier).
2. **Interface** : l'en-tête avec le logo et le titre-menu des niveaux (`#modeBtn`, `#modeMenu`), la zone de conversation, le compositeur, les réglages.
3. **Script principal** (premier `<script>` après l'interface) :
   - `MODES` : les trois niveaux, avec l'identifiant enregistré (`lite`, `pro`, `max`), le nom affiché (`label`), le nom court, l'identifiant de routage (`model`) et l'identité.
   - **La consigne système** : `DOCTRINE`, `LEVEL_RULES`, `levelPrompt()`, `APP_BRIDGE`, `systemPrompt()`, `identityPrompt()` (voir plus bas).
   - `setMode()` : changement de niveau (titre, menu, carte de travail, `<title>`, enregistrement dans `thanatos.mode`).
   - Les limites et le crédit (abonnements, pourcentages), la connexion, l'envoi des demandes, la lecture des réponses, les aperçus de jeux et pages, les exports.
4. **Petits scripts complémentaires** en fin de fichier : cadre « code uniquement » (note ajoutée à une demande d'image ou de vidéo), photos jointes, etc. Ils ajoutent des notes à la demande de l'utilisateur via `__thanatosHooks.notes` ; ils ne touchent pas à la consigne système.

## Où vit la consigne système

- **Source de référence** : `docs/prompt-systeme-thanatos.txt`, la version courte de la doctrine (`docs/Doctrine-Thanatos.pdf`).
- **Copie utilisée par l'application** : la constante `DOCTRINE` dans `index.html`. C'est une copie exacte du fichier texte, intégrée au fichier pour que l'application reste un fichier unique, sans chargement externe. Un test automatique vérifie que les deux textes sont identiques ; toute mise à jour se fait donc dans `docs/`, puis à l'identique dans `index.html`.

## Comment chaque niveau la reçoit

### Avant cette modification

Chaque demande envoyait un seul message système, `identityPrompt()` :

- l'identité du niveau actif (« Tu es Thanatos 5.5 Lite / Thanatos 6.0 Pro / Thanatos 6.5 Extend Max… »), sans mention de la technique utilisée ;
- l'orientation code, et le format de l'application : pour un jeu, une page ou une application web, une courte présentation puis un seul bloc HTML complet, sans afficher le code.

Ce texte était **le même pour les trois niveaux**, au nom près. Il n'y avait aucune méthode de travail et aucune règle propre à un niveau.

### Après cette modification

Chaque demande envoie toujours **un seul** message système, désormais construit par `systemPrompt()`, dans cet ordre :

1. `identityPrompt()` : identité du niveau et format de l'application. Seul changement : le fichier HTML livré ne dépend d'aucune ressource externe (polices, images, sons et scripts intégrés), comme le demande la doctrine.
2. `DOCTRINE` : la consigne système complète, identique pour les trois niveaux (priorités, Règles FSM, profondeur selon le niveau, ambiguïté, méthode, applications, jeux, cinématiques, contrôles avant livraison, livraison).
3. `levelPrompt()` : `NIVEAU ACTIF : <nom du niveau>`, suivi de la règle de ce niveau seulement (`LEVEL_RULES`, tirée des sections 2, 12 et 14 de la doctrine) :
   - **Thanatos 5.5 Lite** : réponse directe aux demandes simples ; au-delà, découpage fin, une modification à la fois, contrôle après chaque étape ; si la demande dépasse le périmètre, proposer Thanatos 6.0 Pro ou Thanatos 6.5 Extend Max, ou plusieurs demandes courtes.
   - **Thanatos 6.0 Pro** : boucle complète dès que plusieurs fonctions ou un bug non trivial sont en jeu ; plan court écrit ; vérification de l'ensemble avant livraison.
   - **Thanatos 6.5 Extend Max** : boucle complète systématique sur les tâches lourdes : carte du projet, plan écrit, étape la plus risquée d'abord, note de bord, double relecture, recherche des bugs jumeaux.
4. `APP_BRIDGE` : l'articulation entre la doctrine et le format de l'application. Les Règles FSM priment ; le format HTML de l'onglet Code s'applique tel quel ; le compte rendu de livraison tient dans les trois lignes finales, placées après le bloc HTML. Elle précise aussi : le nom est celui du niveau actif (« Thanatos » désigne la gamme) ; la réponse est rédigée dans la langue de la demande originale, même si le contexte préparé en amont est en français ; ce contexte reste interne et n'est jamais mentionné.
   Règles propres à l'application, ajoutées après les essais réels (la doctrine reste copiée à l'identique) :
   - **Première personne** : « je » ; tutoiement par défaut, vouvoiement si la personne vouvoie (clients = entreprises).
   - **Chiffres** : aucun nombre pour une valeur propre à la personne (mesure, statistique, prix, temps), même en exemple ; emplacement visible `[À COMPLÉTER : …]`.
   - **Secrets et données personnelles collés** : jamais recopiés, même dans un exemple (`.env`, commande, configuration, courriel) ; seulement des valeurs fictives bien visibles.
   - **Résultats web** : jamais présentés comme les données de la personne.
   - **Périmètre** : ni renommage ni changement du type d'un module (`export`) sans demande.
   - **Certitude** : rien d'affirmé sans exécution ; un résultat d'exemple est « attendu » et recalculé.
   - **Fin de chaque réponse** : exactement trois lignes, `Changé : …`, `Supposé : …`, `Non exécuté : …`.

Le niveau est lu au moment de chaque envoi : si l'utilisateur change de niveau dans le menu du titre, la demande suivante reçoit l'identité et la règle du nouveau niveau. La consigne n'est jamais enregistrée dans l'historique de la conversation, jamais affichée, et jamais incluse dans les exports.

**Ce qui ne change pas** : le routage de chaque niveau, les limites et le crédit, les identifiants enregistrés (`lite`, `pro`, `max`), les descriptions du menu, le titre-menu, la pré-analyse interne de la demande (`contextualizeRequest`, qui ne reçoit pas la doctrine) et les notes ajoutées par les petits scripts.

## Tests

Les tests de bout en bout (WebKit et Chromium, réseau simulé, clé factice) se trouvent hors du dépôt, dans l'environnement de test. La suite `prompt.py` couvre ce câblage :

- copie exacte de la consigne ;
- un seul message système, dans le bon ordre ;
- la règle du seul niveau actif, et le changement de niveau en cours de session ;
- les règles propres à l'application (première personne, `[À COMPLÉTER : …]`, secrets collés, résultats web, périmètre, certitude) et les trois lignes finales, en dernier dans la consigne ;
- le routage inchangé ;
- la pré-analyse sans la doctrine ;
- rien d'affiché à l'écran ;
- aucune clé dans le dépôt.

## Journal de travail

La passe « méthode » (carte, plan, étape la plus risquée d'abord, double relecture, bugs jumeaux) est consignée dans [NOTE-DE-BORD.md](NOTE-DE-BORD.md).
