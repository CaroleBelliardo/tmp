# Claude output — audit de ProfAffv2

Livrables de l'audit complet (sécurité, architecture, performance/données, statistiques, modules d'analyse, valorisation) et pack de prompts pour réaliser les améliorations.

> **Ne pas versionner ce dossier tant que les correctifs de sécurité ne sont pas déployés** : il décrit précisément les failles. Ajoutez `Claude output/` à `.git/info/exclude` en attendant (cela n'altère pas `.gitignore`).

## Par où commencer (15 minutes)

> **Mise à jour du 2026-10-08** (audit dynamique sur une instance avec les données réelles) : voir `prompts/README_prompts.md`, section « Mise à jour du 2026-10-08 ». Commencer par `P45` puis `P44`.

1. `reports/00_synthese_executive.md` — verdict, 10 constats prioritaires, opportunités scientifiques, décisions attendues.
2. `reports/01_audit_securite.md` §5 — runbook secrets (rotation du mot de passe, vérification de la visibilité GitHub) : **première action, humaine**.
3. `reports/07_feuille_de_route.md` — phases et jalons de décision J1–J5.
4. `prompts/README_prompts.md` — méthode d'exécution des prompts et tableau P01–P35.

## Contenu

| Dossier / fichier | Rôle |
|---|---|
| `reports/00_synthese_executive.md` | Synthèse d'une page-et-demie |
| `reports/01_audit_securite.md` | SEC-01…14, runbook secrets, liste de contrôle avant mise en production |
| `reports/02_audit_architecture_qualite.md` | ARC-01…11, FE-01…07, architecture cible, ordre de migration |
| `reports/03_audit_performance_donnees.md` | DATA-01…07, PERF-01…11, schéma reconstitué, plan d'optimisation, protocole de mesure |
| `reports/04_audit_scientifique_statistiques.md` | Revue calcul par calcul, cadre statistique cible, 7 questions pour les responsables scientifiques, références |
| `reports/05_modules_analyse_innovants.md` | Modules M1–M14 (fiches, validation, dépendances, options d'implémentation) |
| `reports/06_valorisation_et_conformite.md` | Licences, FAIR, publication, pistes économiques, RGPD/gouvernance |
| `reports/07_feuille_de_route.md` | Phases 0–4, jalons, indicateurs |
| `reports/08_registre_constats.csv` | 99 constats exploitables (72 initiaux + 27 ajoutés le 2026-10-08) par outil (UTF-8 ; séparateur virgule — dans Excel : Données > À partir d'un fichier texte/CSV) |
| `sql/data_quality_audit.sql` | Profilage en lecture seule à exécuter sur une copie de la base |
| `sql/indexes_proposal.sql` | Index composites et vue de symétrie (à valider par `EXPLAIN`) |
| `sql/schema_v2_draft.sql` | Brouillon du modèle v2 (censure explicite, protéoformes, jeux de données, `partner_profile`) |
| `prompts/` | `00_CONTEXTE_COMMUN.md`, `README_prompts.md`, `P01…P35` |
| `proposed/CLAUDE.md.proposed` | À copier à la racine du dépôt sous le nom `CLAUDE.md` |
| `proposed/settings.json.proposed` | Règles `deny` anti-gaspillage de tokens, à copier dans `.claude/settings.json` |
| `tools/` | Instance de test isolée (Docker, données réelles), robot de parcours `crawl.py` et ses 138 cas (audit dynamique du 2026-10-08) |
| `_backup_2026-10-08/` | Copies du registre et des prompts avant la mise à jour du 2026-10-08 |

## Limites de l'audit

Analyse statique : PHP n'a pas pu être exécuté et la base n'a pas été consultée ; le schéma est reconstitué (rapport 03 §1) et les hypothèses sur les données sont à confirmer avec `sql/data_quality_audit.sql`. Les références bibliographiques vérifiées via PubMed sont marquées comme telles dans le rapport 04 §5 ; les autres sont signalées « à vérifier ». Les points juridiques (licences, RGPD, propriété intellectuelle) sont des alertes à faire valider par le service de valorisation, pas des avis.

## Modification du dépôt hors de ce dossier

Un seul changement : le fichier vide `.git/index.lock` (verrou périmé créé par un `git status`) a été renommé en `.git/index.lock.claude-stale-empty` car la suppression n'est pas permise sur ce montage ; vous pouvez le supprimer.
