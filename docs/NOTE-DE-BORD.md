# Note de bord — application de la doctrine à Thanatos (branche doctrine-thanatos, PR #1)

Niveau appliqué : Thanatos 6.5 Extend Max (boucle complète). Priorités : Règles FSM > consignes explicites > méthode.

## Cadrage
Objectif : relire tout le dépôt avec la méthode, corriger sur la branche les bugs et incohérences trouvés (y compris les conflits doctrine / application quand une solution sûre existe), sans toucher aux limites ni au routage, sans fusion ni publication.
Contraintes : main et site publié inchangés ; aucun secret ; aucun nom de modèle sous-jacent ajouté dans le texte visible ; messages de commit en français.

## Carte du projet (index.html, branche)
- l.13–1060 styles ; l.1066–1135 interface (en-tête + menu du titre, conversation, compositeur, configuration).
- Script principal (l.1137+) : MODES, PLANS, stockage (LS/LSJ), état unique `state`, consigne système (DOCTRINE, LEVEL_RULES, levelPrompt, APP_BRIDGE, systemPrompt, identityPrompt), statut, setMode, clé / limites / déblocage, Markdown sûr, cartes HTML (aperçu, plein écran), fil, sauvegarde de la conversation, envoi (contextualizeRequest, readStream, callApi), configuration, menu du titre, pièces jointes, exports, démarrage.
- Scripts complémentaires : anti-zoom / menu contextuel, cadre « code uniquement », photos, bouton outils.
- Points d'entrée : clic/touche (envoi, menu, Échap), minuteries (rafraîchissement de la clé 60 s), événements storage / visibilitychange / fullscreenchange / message.

## Plan (étape la plus risquée d'abord)
1. Échap : un seul appui ne doit faire qu'une chose (le plus risqué : plusieurs écouteurs globaux, suites fs/menu).
2. Lecture du flux : dernière ligne sans retour à la ligne final perdue.
3. Consigne : articulation doctrine / application (nom du niveau, langue de réponse, ressources externes, réponse partielle).
4. Jumeaux de cohérence : dates de renouvellement (horloge commune), part hebdomadaire affichée (constante unique), message d'erreur photo resté affiché.
5. Tests : compléter prompt.py, nouvelle suite revue.py (correctifs + parcours tactile), relancer les 7 suites sur WebKit et Chromium.
6. Documentation, commits, mise à jour de la PR.

## Constats et corrections (dans l'ordre du plan)
| # | Constat (reproduit par un test avant correction) | Correction | Contrôle |
|---|---|---|---|
| 1 | Échap pendant une génération : quitter le plein écran de l'aperçu, ou fermer le menu du titre, arrêtait aussi la réponse en cours (plusieurs écouteurs globaux agissaient sur le même appui). | Un appui = une seule action : l'écouteur général ignore un appui déjà traité ; le menu du titre marque l'appui comme traité. | revue.py (tactile + bureau), fs.py, menu.py |
| 2 | Flux de réponse : une dernière ligne `data:` reçue sans retour à la ligne final était perdue (fin de réponse tronquée). | Traitement du reste du tampon à la fin du flux (sauf arrêt demandé). | revue.py, run.py |
| 3 | Consigne : la doctrine dit « réponds dans la langue de la personne » alors que le contexte préparé en amont est toujours rédigé en français ; « Tu es Thanatos, la gamme » vs nom du niveau ; « aucune dépendance externe indispensable » vs « aucune ressource externe ». | Articulation complétée (nom du niveau actif, langue de la demande originale, contexte amont interne et jamais mentionné) ; format HTML aligné : aucune ressource externe. DOCTRINE inchangée (copie exacte du fichier docs). | prompt.py |
| 4a | Jumeau de date : la phrase « Renouvellement le 1er … » de la pastille et du message de limite utilisait l'horloge du poste, la carte « Ce mois-ci » l'horloge commune : deux mois différents possibles dans la même fenêtre. | Les deux utilisent `nextMonthUTC()`. | revue.py, limits.py |
| 4b | « 30 % » écrit en dur dans la confirmation de déblocage. | Valeur tirée de `WEEK_SHARE` (texte identique). | limits.py |
| 4c | Erreur « Photo trop volumineuse » écrite directement dans le statut : ne disparaissait jamais. | Passe par le statut commun (masquage automatique). | revue.py |

Recherche de jumeaux : autres `new Date()` / `Date.now()` (horodatage du cache de clé, délai de rafraîchissement, double tap) : mesurent un temps réel écoulé, corrects. Autres écritures directes du statut : aucune. Autres écouteurs Échap : plein écran, menu, configuration, pastille de clé : tous couverts par la règle « un appui = une action ».

## Décisions
- Limites, plans, formules et routage non modifiés (MODES, CONTEXT_MODEL, WEB_SEARCH_DEFAULT, PLANS).
- Aucun texte visible nouveau, hormis le comportement corrigé.

## Reste à arbitrer (non modifié)
- Sobriété en crédits : pré-analyse à chaque demande (jusqu'à 80 Ko de fichiers envoyés en plus) ; recherche web activée par défaut ; ~6,7 Ko de consigne par demande.
- Le nom complet de la marque figure dans l'interface (libellés de la clé) et dans l'en-tête de la doctrine (antérieur à ce travail) : à garder ou non.
- Dépôt public : la doctrine y est lisible par tous.
- Sauvegarde de la conversation : le repli sur les 10 derniers messages peut encore échouer sans avertissement (mineur).

## Vérifications
Relu et exécuté : 7 suites sur WebKit et Chromium (réseau simulé, clé factice), dont le parcours tactile (390 et 320 px). Relu, non exécuté : comportement réel du service de réponses (non appelé).
