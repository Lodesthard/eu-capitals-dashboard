# EU capitals — sustainable vacation dashboard

TD1 — Informatique décisionnelle.

**Décision supportée :** quelle capitale de l'UE offre les meilleures conditions de
vacances la semaine prochaine, tout en minimisant les émissions de CO₂ du trajet
depuis la ville d'origine ?

## Pipeline

| Étape | Fichier | Rôle |
|---|---|---|
| Acquisition (statique) | `data/capitals.csv` | Coordonnées des 27 capitales, committées une fois |
| Acquisition (live) + transformation + KPI | `scripts/fetch_and_score.py` | Météo 7 jours via [Open-Meteo](https://open-meteo.com) (une seule requête groupée), distance haversine, CO₂ aller-retour, normalisation 0-1, score final 50/50 météo/CO₂ |
| Stockage & versioning | `docs/data/rankings.json` | Résultat généré et committé par l'Action à chaque run |
| Visualisation | `docs/index.html` | Recommandation, classement en barres, nuage météo vs CO₂ (Chart.js) |
| Automatisation | `.github/workflows/update-data.yml` | Run chaque lundi 06:00 UTC + déclenchement manuel |

Le dashboard est servi par GitHub Pages depuis le dossier `/docs`.

Réponses aux questions du TD : [`REFLECTIONS.md`](REFLECTIONS.md).
