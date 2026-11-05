# Orchestrateur ou integration continue ? (TP06)

| Critere                         | Prefect (orchestrateur)                                                                 | GitHub Actions (CI)                                                                  |
|---------------------------------|-----------------------------------------------------------------------------------------|--------------------------------------------------------------------------------------|
| Ce qui declenche l'execution    | Une planification (cron `0 6 * * 1`, chaque lundi 6 h), un Quick run dans l'interface ou `python flow_mlops.py` | Un evenement Git : chaque `git push` (et le bouton "Run workflow" via `workflow_dispatch`) |
| Ou s'execute le traitement      | Sur notre poste : processus `serve` ou worker du work pool `pool-tp`, avec les vraies donnees DVC | Sur une machine Linux ephemere de GitHub (`ubuntu-latest`), donnees regenerees par script |
| Qui surveille, et ou            | Le serveur Prefect : interface http://127.0.0.1:4200 (Runs, Deployments, Work Pools), etat et journaux de chaque tache | GitHub : onglet Actions du depot, statut (vert/rouge) affiche sur chaque commit, journal de chaque etape |
| Ce qui se passe en cas d'echec  | La tache `valider` leve une erreur, le flow run passe a Failed et `notifier` ne s'execute pas ; `rejouer_pipeline` est reessayee 2 fois | L'etape "Verifier la regle de qualite" sort avec le code 1, le job passe au rouge et le commit est marque en echec ; les rapports sont quand meme conserves |
| Type de tache adapte            | Traitements recurrents et longs sur les donnees de production : reentrainement, scoring, rapports planifies | Verifications rapides du code a chaque modification : tests, linters, controle de qualite reproductible |
| Ce qui est versionne dans Git   | `flow_mlops.py`, `planifier.py`, `prefect.yaml` (definition du workflow et du deploiement) ; l'historique des runs reste dans la base du serveur | `.github/workflows/ci.yml`, `verifier_qualite.py`, `requirements-ci.txt` ; l'historique des runs reste sur GitHub |

## Conclusion (5 lignes)

1. Prefect transforme le pipeline DVC en un workflow observable, planifie chaque lundi et stoppe de lui-meme si F1 < seuil_f1 (0.25 < 0.82 observe).
2. GitHub Actions rejoue le meme pipeline a chaque push et refuse un commit qui degrade le modele (run rouge, puis vert apres `git revert`).
3. L'orchestrateur execute et surveille des traitements recurrents ; la CI valide le code avant qu'il n'atteigne la production.
4. L'architecture serveur / work pool / worker separe la definition des workflows de leur execution, ce qui permet de passer a l'echelle.
5. Les deux outils sont complementaires : aucun ne remplace l'autre dans une chaine MLOps.
