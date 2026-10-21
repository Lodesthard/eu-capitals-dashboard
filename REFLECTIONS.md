# Reflections — TD1

## 1. Before building anything — pondération personnelle météo vs CO₂

Je ne mettrais pas les deux critères à égalité : pour une semaine de vacances, je
donnerais environ 60 % au CO₂ et 40 % à la météo, parce qu'un vol long-courrier
pèse beaucoup plus lourd que quelques degrés de différence. Je n'ignorerais pas la
météo pour autant : une destination proche mais sous la pluie toute la semaine ne
répond pas vraiment à la question « où partir en vacances ».

## 2. Stage 1 — Fiabilité de l'acquisition (26 appels vs 1)

Ma supposition : 26 connexions successives depuis la même IP (un runner GitHub
partagé par beaucoup d'utilisateurs) multiplient les occasions d'échec — chaque
handshake TLS peut bloquer, et l'API peut limiter le débit (rate limiting) d'une IP
qui envoie des rafales de requêtes. Une seule requête = un seul point de défaillance
au lieu de 26, et une seule tentative à réessayer. C'est confirmé par le commentaire
de `fetch_weather_batch` : en test, les premières capitales passaient puis les
suivantes tombaient en timeout à chaque retry ; la requête groupée règle le problème
et est en plus environ 100 fois plus rapide.

## 3. Stage 1 — Provenance du facteur d'émission (CM3)

Avant publication, je remplacerais 0,15 kg CO₂e/km par une valeur sourcée et datée :
les facteurs officiels de l'année en cours (DEFRA/DESNZ « UK Government GHG
Conversion Factors », ou la Base Empreinte de l'ADEME), en précisant s'ils incluent
le forçage radiatif (effet hors CO₂ des avions, qui peut presque doubler le chiffre).
J'utiliserais aussi un facteur différent selon la distance (court / moyen /
long-courrier) plutôt qu'un seul nombre, et j'afficherais la source, l'année et la
méthode sur le dashboard, avec un rappel annuel pour mettre la valeur à jour.

## 4. Stage 2 — Relatif vs absolu

Non, la température réelle d'une capitale moyenne ne change pas : c'est une mesure.
Oui, son *score* change : si on ajoute une capitale plus froide que toutes les
autres, le minimum de la liste baisse et la capitale moyenne se retrouve plus haut
sur l'échelle 0-1. Un KPI normalisé est donc relatif à l'ensemble comparé, pas une
propriété intrinsèque de la ville : un score de 0,8 veut dire « meilleure que la
plupart des autres de cette liste, cette semaine », pas « il fait beau ».

## 5. Stage 3 — Prédiction avant exécution

Prédiction : **Madrid**. Fin octobre, Madrid est généralement ensoleillée et sèche,
et à ~1 050 km de Paris son score CO₂ reste correct (les capitales encore plus au
sud — Nicosie, La Valette, Athènes — sont deux à trois fois plus loin), alors que
les capitales proches (Bruxelles, Luxembourg, Amsterdam) devraient être pénalisées
par la pluie et le froid.

## 6. Stage 6 — La prédiction était-elle juste ?

Oui : le premier run réel (21/10/2026, 03:29 UTC, origine Paris) classe **Madrid**
en tête (score final 0,755 : météo 0,804, CO₂ 0,706 ; 18,8 °C de moyenne, 0 mm de
pluie, 10 h de soleil/jour, 1 053 km). Ça montre que la formule 50/50 est
prévisible : elle récompense la destination qui n'est mauvaise sur *aucun* des deux
axes. Ce qui m'a surpris, c'est le reste du classement : Lisbonne (2e) a une
meilleure météo que Madrid mais perd à cause des 400 km supplémentaires, et
Bruxelles arrive 4e avec une météo médiocre (0,30) uniquement grâce à son CO₂
parfait (1,00) — alors que Nicosie, la meilleure météo de la liste (0,96), tombe à
la 19e place car c'est la plus éloignée, donc CO₂ = 0 par construction de la
normalisation. Avec ma pondération personnelle (60 % CO₂), Madrid resterait
premier mais Bruxelles passerait devant Rome.

## 7. Stage 6 — Monitoring : quel signal sur la page publique ?

Le seul signal visible est la ligne « Data last updated … » en bas de page, qui
affiche `generated_at`. Si l'Action s'arrête (désactivation après 60 jours sans
activité) ou échoue chaque semaine, cette date cesse d'avancer — mais le visiteur
doit la lire, et savoir que les données doivent avoir moins de 7 jours. Ce n'est
pas suffisant : le dashboard continue d'afficher une recommandation qui a l'air
valide. J'ajouterais une alerte calculée côté page (bandeau rouge si
`generated_at` date de plus de 8 jours, recommandation marquée « périmée »), et
côté pipeline une notification en cas d'échec du workflow (e-mail GitHub ou issue
ouverte automatiquement), plus un commit ou un keepalive pour éviter la
désactivation à 60 jours.

## 8. Stage 7 — Changement d'origine (Rennes)

**Fichiers modifiés :**

- `scripts/fetch_and_score.py` — c'est le seul endroit où l'origine intervient dans
  le calcul : les constantes `PARIS_LAT, PARIS_LON` deviennent `ORIGIN_NAME`,
  `ORIGIN_LAT, ORIGIN_LON` (Rennes, 48.1173, -1.6778), utilisées pour la distance
  et pour l'exclusion par nom. Rennes n'est pas dans `capitals.csv`, donc
  l'exclusion ne retire plus rien : Paris devient une destination et la liste passe
  de 26 à 27. J'ai gardé l'exclusion (générique) pour que le script reste correct
  si on choisit une capitale comme origine. Le script écrit aussi `"origin"` dans
  `rankings.json`.
- `docs/index.html` — le titre affichait « from Paris » en dur et le graphique
  « all 26 destinations » en dur. La page lit maintenant `data.origin` et
  `dest.length` depuis le JSON : l'origine n'est définie qu'à un seul endroit (le
  script), et la page ne peut plus contredire les données.
- `data/capitals.csv` n'a pas besoin de changer : c'est la liste des destinations,
  pas l'origine.

**Résultat :** Madrid reste 1er depuis Rennes (0,805), suivi de Lisbonne (0,793) et
de **Paris (0,710)**, qui n'existait pas dans le classement précédent et arrive
directement 3e grâce à un CO₂ minimal (308 km). Les distances et le CO₂ ont tous
changé (Madrid : 1 053 → 871 km, 316 → 261 kg CO₂e A/R), et l'écart entre
Madrid et Lisbonne se resserre (0,028 → 0,012) parce que Rennes est plus à l'ouest.

**Absolu ou relatif ?** Relatif. Le score CO₂ dépend de la distance depuis
l'origine, et la normalisation dépend de l'ensemble des destinations comparées
(ajouter Paris a déplacé le minimum de distance). Ici le gagnant est le même que
depuis Paris, mais seulement parce que les deux origines sont proches et que Madrid
a une semaine exceptionnellement sèche et ensoleillée : depuis Helsinki ou Athènes,
le podium serait différent. La « meilleure destination » est une réponse à une
question posée depuis un point de départ, une semaine donnée, avec une pondération
donnée — pas un fait sur les capitales.
