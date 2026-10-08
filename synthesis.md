# ProfAff v2 : note de synthèse

*Date : 8 octobre 2026 · Point de comparaison : commit `eba3191` · Branche de travail : `auto/prompts-20261008`*

---

## 1. En bref

- **ProfAff v2** est un site web qui recense des mesures de liaison entre protéines : quelle protéine se fixe sur quelle autre, et avec quelle intensité.
- Le projet vient d'être repris. Un **audit complet** a été fait : lecture du code, puis **test du site en conditions réelles** sur une copie de la base (138 pages parcourues).
- L'audit a révélé des **failles de sécurité**, des **recherches très lentes** et des **bugs d'affichage**.
- **Déjà fait** : les 5 chantiers de sécurité de base (P01 à P05), un chantier de performance (P45), 10 bugs corrigés (P44) et un environnement Docker reproductible.
- **Reste à faire** : 42 chantiers en file d'attente. Ils couvrent la modernisation technique du code, puis de nouvelles analyses scientifiques (modules M1 à M21), un espace privé pour les chercheurs et la préparation de la publication.
- **Ce qui ne change pas** : aucune formule scientifique et aucune valeur mesurée n'a été modifiée. Les points qui relèvent d'un choix scientifique sont signalés et laissés en l'état.

## 2. Pourquoi ce travail

- Le site repose sur un code ancien : des pages PHP qui mélangent calcul et affichage, pas de tests automatiques, une structure de base de données non versionnée, une bibliothèque de graphiques obsolète.
- Ce que l'audit a trouvé concrètement :
  - des requêtes pouvaient être détournées par une adresse web soigneusement construite (injection SQL) ;
  - une recherche avancée pouvait tourner plus de 5 minutes, et la requête continuait côté serveur pendant des heures après le départ de l'utilisateur ;
  - des identifiants de base de données étaient écrits dans le code et figurent toujours dans l'historique Git.
- « Audit dynamique » signifie que le site a été lancé pour de vrai, sur une copie de la base réelle (dump du 26 août 2025), et non seulement relu.

## 3. Les grands choix stratégiques

- **Licences (J1)** : code sous licence « usage non commercial » (PolyForm Noncommercial 1.0.0) ; données sous CC BY-NC-SA 4.0 (partage autorisé avec citation, sans usage commercial, à l'identique). À faire valider par le service valorisation de l'université avant toute mise en ligne publique.
- **Statistiques (J2)** : on garde les valeurs d'origine et le calcul habituel par défaut. Les nouveaux indicateurs s'ajoutent à côté. On ne corrige que ce qui est mathématiquement faux, et chaque correction est tracée (ancienne valeur, nouvelle valeur, raison).
- **Architecture de calcul (J3)** : chaque tâche va à l'outil le mieux adapté. PHP pour les pages web, Python pour les calculs lourds et statistiques, Celery et Redis pour les tâches longues en arrière-plan, JavaScript seulement pour l'affichage.
- **Espace privé (J4)** : oui. Les chercheurs pourront déposer leurs propres données, visibles d'eux seuls, avec connexion sécurisée et export ou suppression conformes au RGPD.
- **Publication (J5)** : un article scientifique, sans dépôt de brevet.

## 4. Les outils, expliqués simplement

| Outil | À quoi il sert | Image pour la réunion |
|---|---|---|
| **Docker** | Emballe le site et sa base dans un « conteneur » : tout le monde obtient le même environnement en une commande. | Une valise prête à l'emploi |
| **MySQL** | La base de données qui stocke les mesures. | Un grand classeur structuré |
| **Redis** | Une file d'attente rapide où l'application dépose les tâches longues. | Le bon de commande posé au passe d'un restaurant |
| **Celery** | Le gestionnaire qui prend les tâches dans Redis, les fait exécuter par des programmes Python (les « workers ») et range le résultat. | Le chef de cuisine qui organise les commandes |
| **Python** (numpy, scipy…) | Langage et bibliothèques de calcul scientifique. | La calculatrice scientifique du laboratoire |
| **Tests automatiques et CI** | Vérifient à chaque modification que les résultats n'ont pas bougé. | Le contrôle qualité avant expédition |
| **Twig** | Moteur de gabarits : sépare la logique du code de la mise en page. | Le gabarit de mise en page |
| **Plotly / d3** | Graphiques interactifs dans le navigateur. | Un papier millimétré interactif |
| **AlphaFold3** | Prédiction 3D de complexes protéiques (import et visualisation prévus). | La maquette 3D d'un assemblage |
| **UniProt, STRING, PDB, ELM** | Bases publiques de référence : séquences, réseaux d'interactions connues, structures 3D, motifs de liaison. | Les encyclopédies du domaine |

**Exemple Redis.** Aujourd'hui, un calcul lourd (par exemple une analyse portant sur 1,2 million de lignes) bloque la page jusqu'au bout et peut saturer le serveur. Avec Redis et Celery, le site dépose la tâche et répond tout de suite « travail n° 4521 en cours » ; l'utilisateur revient consulter le résultat quand il est prêt, sans rester devant une page qui tourne.

## 5. Ce qui a changé sur le site

- **Accueil** : la liste de suggestions de molécules ne charge plus 4,7 Mo à chaque visite, mais environ 4 ko, après 2 caractères saisis.
- **Recherche avancée** : requête réécrite, passée de plus de 5 minutes à environ 1 seconde. Export CSV corrigé (colonne « longueur » vide), tri par nom et par longueur ajouté, pagination qui conserve la recherche.
- **Explorateur de protéines** : une molécule qui n'a qu'un seul fragment ouvre directement la bonne page. Une recherche par identifiant exact ou par nom de gène fonctionne. Le segment « site » inutilisé est mis à 0 avec un message visible.
- **Fiche détail** : le filtre par fragment ou par site s'applique à chaque partenaire. Avant, le second filtre ne donnait jamais de résultat et la page restait vide.
- **Profils fragment et molécule** : libellés des barres corrigés (ils étaient décalés) ; valeurs manquantes affichées « N/A » au lieu de 0 ; ex aequo triés de façon stable, ce qui évite les doublons entre pages.
- **Re-seuillage (rethres)** : les séquences des partenaires sont de nouveau trouvées (avant, elles étaient vides pour les 5 499 points de l'exemple de l'audit). Message explicite si aucune donnée n'existe.
- **Complexome** : vérifie que le profil en mémoire correspond à la demande et affiche un message sinon. Une concentration nulle ne provoque plus de division par zéro.
- **Comparateur** : requête SQL invalide corrigée, erreurs enregistrées au lieu d'être avalées ; tri sur la valeur numérique et non sur le texte.
- **Statistiques** : le cache de 24 h expire bien au bout de 24 h (avant : 1 h) ; la page ne recompte plus toute la base à chaque visite.
- **Outils** : calculateur de charge (séquence FASTA nettoyée, choix du jeu de pKa) ; calculateur de volume (unités adaptées) ; logos (ne plantent plus, message image en cas d'erreur) ; gdraw (longueur de protéine validée) ; recherche de motif (sensible à la casse, pagination, réponse 503 en cas de délai dépassé) ; réseau (liens cassés corrigés) ; partitionneur (saisies validées).
- **Exports CSV** : mêmes filtres que la page affichée.
- **Sécurité générale** : secrets lus depuis l'environnement, messages d'erreur sans détails techniques, en-têtes de sécurité, cache d'administration protégé par un jeton, limites sur la taille des requêtes.
- **Infrastructure** : installation par Docker (PHP 8.3, MySQL 8.4) ; compte de base de données en lecture seule pour le site.

## 6. Exemples avant / après

| Sujet | Avant | Après |
|---|---|---|
| Recherche « avec données expérimentales » | plus de 300 s, la requête continue des heures | environ 1 s |
| Complexome (une molécule) | 9 à 17 s | moins de 1 s (index à appliquer par un administrateur) |
| Liste de suggestions de l'accueil | 4,7 Mo | 4 ko |
| Cache de 24 h | expire au bout de 1 h | respecté |
| Valeur combinée « 1 550,00 » | lue comme 1 (le graphique la place au bas de l'axe) | lue comme 1 550 |
| Export CSV d'un exemple | 62 lignes, dont des lignes hors sélection | 29 lignes, identiques à l'affichage |
| Valeur manquante | affichée 0 | affichée N/A |

**Point d'attention pour la réunion** : certains chiffres affichés changent (valeurs manquantes, tri, graphiques, exports). Aucune valeur de base ni aucune formule n'a été modifiée.

## 7. Points de vigilance et décisions à prendre

- **Identifiants dans l'historique Git** : le mot de passe administrateur MySQL doit être changé, puis l'historique purgé. Action humaine.
- **Ne pas rendre le dépôt public avant les correctifs** : le dossier `Claude output` décrit les failles.
- **Choix scientifiques en attente (non modifiés)** : calcul de la p-valeur combinée (minimum × nombre), écart-type de population au lieu d'échantillon, exclusion des non-détections des moyennes, complexome sans compétition entre partenaires, stabilité numérique de la page d'équations.
- **Index de performance** : `db/hotfix_2026-10_indexes.sql` doit être exécuté une fois par un administrateur.
- **Audit de qualité des données (P15)** : à lancer par vous sur une copie de la base, puis à valider avec les résultats.
- **Validation juridique** : licences et autorisations des logos.
- **Historique Git** : deux commits sans message clair (« tmp », « work in progress ») et des changements de fins de ligne sur plus de 200 fichiers. À nettoyer avant tout partage.
- **BUG-02 (calculateur de volume)** : test de non-régression ajusté, mais pas de commit dédié. À vérifier.
- **Actions humaines à venir** : soumissions sur le serveur AlphaFold, attribution du DOI Zenodo, soumission de l'article.

**Message clé pour la réunion** : nous avons audité le site, corrigé les failles de sécurité les plus graves et les lenteurs majeures, sans toucher aux résultats scientifiques. La suite prévoit la modernisation du code, puis de nouvelles analyses et la publication.

---

## Annexe A. Modifications depuis le commit `eba3191` (détail technique)

16 commits. Hors fins de ligne, 144 fichiers sont modifiés ou créés (environ 4 475 lignes ajoutées et 2 538 supprimées). Le diff brut en compte 221, ce qui est gonflé par la conversion des fins de ligne.

| Commit | Date | Contenu |
|---|---|---|
| `44038e1` | 07/10 | Lot P01 à P05 (secrets, injections SQL, XSS, durcissement, corrections confirmées), voir `CHANGELOG.md` |
| `d690941` | 08/10 | `CLAUDE.md` et `.claude/settings.json` (garde-fous tokens et secrets) |
| `290d1ce` | 08/10 | Commit « tmp » (sans message explicite) |
| `df1f0f2` | 08/10 | Commit « work in progress » (sans message explicite) |
| `5d59752` | 08/10 | P45 : index, réécriture des `EXISTS`, durée maximale des requêtes (10 s, export 60 s) |
| `0c48530` | 08/10 | BUG-11 : séquences des partenaires dans rethres |
| `7abe7cb` | 08/10 | BUG-06 : explorateur de protéome (une molécule résolue) |
| `38b213b` | 08/10 | Correctif de syntaxe du script de test |
| `459814e` | 08/10 | BUG-01 : calculateur de charge |
| `f5bb629` | 08/10 | BUG-03 et BUG-04 : logos |
| `149ae34` | 08/10 | BUG-04 : libellés d'axes des logos |
| `b0847fd` | 08/10 | BUG-05 : gdraw valide la longueur |
| `3a32f75` | 08/10 | BUG-07 : recherche de motif (casse, pagination, 503) |
| `18a2cd9` | 08/10 | Correctif de regex du test |
| `5da95c8` | 08/10 | BUG-08 : explorateur, segment site normalisé |
| `32fc4d3` | 08/10 | BUG-09 (LIKE) et BUG-10 (plotcomplex, session expirée) |

**Non commité** : `tests/p44/smoke_p44.sh` (regex du test BUG-02 ajustée) ; trois fichiers du calculateur de volume, dont les différences observées ne portent que sur les fins de ligne.

**Fichiers nouveaux notables** : `utils/selection.php`, `utils/html.php`, `utils/QueryGuard.php`, `utils/logo/logo_error.php`, `js/index-suggest.js`, `config/local.example.php`, `db/README.md`, `db/hotfix_2026-10_indexes.sql`, `db/grants_readonly.example.sql`, `docker-compose.yml`, `docker/`, `tests/p44/`.

**Données de référence** : `Census` 1 211 036 lignes, sans clé primaire ; `Proteome` 8 094 056 lignes, sans index ; `Molecule` 13 892 lignes ; 96,5 % des lignes de `Census` sont des non-détections (pKd = 0).

## Annexe B. Les 47 prompts (ordre de la file `queue.json`)

Les 5 premiers sont déjà appliqués. Les 42 autres sont en file d'attente pour le lancement autonome (`tools/run_prompts.ps1`, un commit et un tag par prompt). « Plan » signifie qu'un plan est écrit avant exécution. « Opus » désigne le modèle le plus puissant, réservé aux tâches de conception ou de calcul sensible.

| Rang | ID | Objectif, en clair | Dépend de | Remarque |
|---|---|---|---|---|
| — | P01 | Secrets sortis du dépôt, configuration par variables d'environnement | — | Appliqué |
| — | P02 | Protection contre l'injection SQL | P01 | Appliqué |
| — | P03 | Protection contre le XSS (affichage sécurisé) | — | Appliqué |
| — | P04 | Durcissement : erreurs, cache d'administration, limites, en-têtes | P01 | Appliqué, reste la CSP |
| — | P05 | Corrections de bugs confirmés, à faible risque | — | Appliqué |
| 1 | P45 | Performance et anti-surcharge : index, requêtes, durée maximale | — | Fait (`5d59752`) ; index à exécuter par un admin |
| 2 | P44 | Deuxième lot de 11 bugs | — | Fait sauf BUG-02 à vérifier |
| 3 | P36 | M15 : séquences de référence, base ELM, audit de couverture | — | |
| 4 | P42 | M21 : tableau de bord comparatif multi-appâts et multi-espèces | P36 | |
| 5 | P06 | Outillage : Composer, tests, analyse statique, CI, chemins unifiés | P02 | |
| 6 | P07 | Schéma de base versionné, migrations, index | P06 | |
| 7 | P08 | Environnement Docker de développement et données de démonstration fictives | P07 | Jamais de données réelles |
| 8 | P09 | Réécriture du cache (durée de vie respectée, écritures sûres) | P04, P06 | |
| 9 | P15 | Audit de qualité des données | — | Exécuté par vous sur une copie |
| 10 | P46 | Socle de calcul Python, Celery et Redis | P08 | Décision J3 |
| 11 | P18 | Socle statistique : fonctions pures et testées | P06 | |
| 12 | P39 | M18 : réseau STRING en direct, centré sur un appât | — | |
| 13 | P40 | M19 : visualiseur de complexes PDB colorés par affinité | — | |
| 14 | P10 | Contrôleur frontal et mise en page Twig (1re étape de migration) | P06, P09 | Opus, plan |
| 15 | P11 | Repositories, ProfileService et tests de non-régression | P08, P10 | Opus, plan |
| 16 | P12 | Migration de l'explorateur et de la fiche détail | P11 | |
| 17 | P13 | Migration des profils fragment et molécule | P11, P12 | Opus, plan |
| 18 | P14 | Suppression de JpGraph et GD, graphiques modernes | P12, P13, J1 | |
| 19 | P16 | Front-end : assemblage, CSS, accessibilité | P12, P13 | |
| 20 | P17 | Modèle de données v2 et migration depuis `Census` | P07, P15, P18 | Opus, plan |
| 21 | P19 | API REST v1, OpenAPI, téléchargements | P11 | |
| 22 | P21 | M1 : complexome compétitif exact | P11, P18, P46 | Opus, plan |
| 23 | P22 | M2 : interactome différentiel par contexte, simulations « que se passe-t-il si » | P21 | |
| 24 | P23 | M3 : interrupteurs de modification (variation d'affinité) | P18, P46 | |
| 25 | P24 | M4 : moteur de motifs d'affinité (PSAM) | P36, P46 | Opus, plan |
| 26 | P25 | M5 : recherche de motifs sur tout le protéome | P24, P36, P46 | Opus |
| 27 | P26 | M6 : sélectivité, familles de protéines, paralogues | P18, P46 | |
| 28 | P28 | M8 : re-seuillage v2 avec taux de fausses découvertes | P11, P46 | |
| 29 | P29 | M9 : score de confiance par interaction et tableau de bord qualité | P15, P17, P18, P46 | Opus |
| 30 | P31 | M11 : simulateur de planification d'expériences | P21, P46 | |
| 31 | P30 | M10 : effets des variants sur les motifs | P23, P24, P46 | |
| 32 | P33 | M13 : visualiseur séquence–structure | P17, P46 | |
| 33 | P27 | M7 : analyse de réseau et comparaison à des bases publiques | P17, P19, P46 | |
| 34 | P37 | M16 : import de modèles AlphaFold3 et visualiseur | P36 | |
| 35 | P38 | M17 : classifieur de mode de liaison structural | P37 | Opus, plan |
| 36 | P41 | M20 : résolveur de sous-complexes (Kd et stœchiométrie) | — | Opus, plan |
| 37 | P43 | Intégration de M15 à M21, pipeline, non-régression | P36 | |
| 38 | P34 | M14 : prédicteur ML calibré (piste de recherche) | P24, P25, P46 | Opus ; arrêt si pas meilleur que la référence |
| 39 | P32 | M12 : espace privé, authentification, RGPD | P04, P17 | Opus, plan ; décision J4 = oui |
| 40 | P35 | Publication des données : DOI, citation, exports PSI-MI, package IMEx | P17, P19 | DOI = action humaine |
| 41 | P20 | Documentation alignée sur le code | P06 à P19 | |
| 42 | P47 | Paquet de publication : manuscrit, figures, reproductibilité | tous | Dernier |
