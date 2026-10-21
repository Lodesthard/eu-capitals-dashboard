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
