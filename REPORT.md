# Compte rendu - TP DataOps 1

Binôme : Eya khelil

## Q1
requirements.txt liste les bibliothèques et leurs versions minimales dans un
fichier versionné avec le projet. N'importe qui peut recréer exactement le même
environnement sur une autre machine avec `pip install -r requirements.txt`
(reproductibilité). Avec un simple `pip install pandas`, chacun installerait
peut-être une version différente et le code pourrait se comporter autrement.
## Q2
Le rapport affiche 8 lignes au lieu de 5 pays. Tunisia apparaît sous 3 formes
(" Tunisia", "Tunisia", "tunisia") et Morocco sous 2 formes ("Morocco",
"MOROCCO"). pandas compare les textes caractère par caractère, donc chaque
écriture devient un groupe séparé.
## Q5
1. Chaque étape est testable séparément (on peut tester clean_orders sans lire de fichier).
2. Le code est plus lisible et plus facile à modifier : on change une règle à un seul endroit sans toucher au reste.
## Q6
Le bon total est 7655.90. L'ancien script disait 12771.10, soit 5115.20 de trop,
donc environ 67 % d'erreur (5115.20 / 7655.90 = 66,8 %).

## Q7
La commande 1019 (un clavier à 4500 au lieu de 45) a causé l'essentiel de
l'erreur : 4500 sur 5115 d'écart. Elle a été attrapée par la règle 4
(invalid_unit_price, prix supérieur à 1000).

## Q8
On garde les lignes rejetées dans un fichier (quarantaine) pour pouvoir
comprendre pourquoi elles ont été rejetées, les faire corriger à la source,
les retraiter ensuite, et garder une trace (audit). Si on les supprimait, on
perdrait des commandes sans le savoir.
## Q9
Quand j'ai remplacé > 0 par > -5 dans la règle 3, le test
test_negative_quantity_is_rejected a échoué : la commande avec la quantité -2
n'était plus rejetée, donc clean contenait [1, 2] au lieu de [2]. Les 4 autres
tests ont continué à passer. Le test a donc détecté la régression.

## Q10
- Un test unitaire vérifie que ma logique (mon code) est juste, sur un petit
  tableau écrit à la main dont je connais la bonne réponse.
- Un contrôle de données vérifie que les données du jour sont acceptables, sur
  les vraies données, à chaque exécution du pipeline.
## Q11
|  | Ancien script | Votre pipeline |
| --- | --- | --- |
| Jour 2 : message d'erreur ? | Non, aucune erreur | Oui : PIPELINE STOPPED, taux de rejet 100 % (6 lignes sur 6) |
| Jour 2 : total affiché | 270200 (faux) | Aucun total, rien n'est publié |
| Jour 3 : message d'erreur ? | Oui, mais un KeyError: 'unit_price' brut, difficile à comprendre | Oui : PIPELINE STOPPED: Missing columns: ['unit_price'] |

## Q12
Jour 2 : les prix ne sont plus dans la même unité. Ils sont multipliés par 100
(44900 au lieu de 449.0), probablement exprimés en centimes. Jour 3 : la
colonne unit_price a été renommée en price, donc le schéma a changé.

## Q13
Le jour 2 est le plus dangereux. L'ancien script ne plante pas et affiche un
total (270200) qui a l'air normal mais qui est environ 100 fois trop grand
(le vrai total serait 2702.00). Personne ne sait qu'il y a un problème et le
mauvais chiffre peut être publié et utilisé pour décider. Au jour 3, le script
plante avec une erreur visible : on sait au moins qu'il y a un problème.

## Q14
Le code de sortie 1 peut être lu par un orchestrateur (cron, Airflow), par
une CI (GitHub Actions) ou par un script de supervision, pour arrêter la suite
du traitement et déclencher une alerte, sans qu'un humain lise la console.
Un code 0 veut dire succès, un code différent de 0 veut dire échec.


## TP2

### Q1
Non, pendant que je travaillais sur la branche, main ne contenait pas la règle 6.
Elle est apparue sur main avec la commande `git merge --no-ff feature/country-rule`.
C'est utile parce que le rapport tourne chaque nuit depuis main : on peut
développer et tester une règle sans risquer de casser ou de changer le rapport
de la nuit. main ne reçoit que du code déjà testé.

### Q2
Une pull request permet qu'un collègue relise le code et le commente avant la
fusion, elle garde une trace de la discussion, et elle peut lancer les tests
automatiquement (CI) avant d'accepter la fusion. On peut aussi protéger main pour
interdire de la modifier sans relecture. Un git merge local ne fait rien de tout ça.

### Q3
Ma ligne md5 dans data/raw.dvc : `md5: <colle ici TA valeur, qui finit par .dir>`
Elle identifie le contenu exact du dossier data/raw (une empreinte calculée à
partir de tous les fichiers). Si je change un seul caractère dans un des CSV,
l'empreinte change complètement, et DVC sait alors que les données ont changé.

### Q4
C'est le fichier data/.gitignore, créé par dvc add. Il contient /raw, donc Git
ignore le dossier data/raw. Seul le petit fichier pointeur data/raw.dvc est
suivi par Git.
### Q5
Les fichiers de dvc-storage sont nommés par leur empreinte md5 (le hash de leur
contenu), répartie en dossiers de deux caractères (par exemple 2b/3c16...). C'est
une bonne idée parce que deux versions différentes d'un même fichier ont des
empreintes différentes : elles ne s'écrasent jamais et peuvent coexister dans le
stockage. Et deux fichiers identiques ne sont stockés qu'une fois.
### Q6
git diff data/raw.dvc n'a montré que deux lignes modifiées (md5 et size). C'est
suffisant pour Git parce que Git ne suit pas les données, il suit seulement le
petit fichier pointeur. Le md5 identifie le contenu exact du dossier, donc
changer le md5 revient à dire "ce commit utilise d'autres données". Il faut
quand même commiter, sinon Git n'enregistre pas quelle version des données va
avec quelle version du code : sans commit, impossible de revenir en arrière
avec un tag ou de partager la nouvelle version.

### Q7
Non, le code n'a pas changé entre data-v1 et data-v2, seules les données ont
changé (5 commandes ajoutées). Total avec data-v1 : 7655.90. Total avec
data-v2 : 8835.40. L'écart de 1179.50 correspond exactement aux 5 commandes en
retard (458 + 135 + 449 + 78 + 59.50).
### Q8
`git checkout data-v1 -- data/raw.dvc` a seulement remis le petit fichier
pointeur (le md5) tel qu'il était à data-v1 : les vrais CSV n'ont pas bougé.
`dvc checkout` lit ce pointeur et remplace les fichiers de data/raw par la
version correspondante, qu'il va chercher dans le cache DVC.

### Q9
Si on oublie dvc checkout, le pointeur dit "v1" mais les fichiers sont encore
ceux de v2. dvc status signale "modified: data\raw", mais le pipeline tourne
quand même, sans erreur, sur les données de v2 et affiche 8835.40 au lieu de
7655.90. C'est dangereux pour un rapport : on croit avoir reconstruit l'ancien
chiffre alors qu'on a recalculé le nouveau, sans aucun message d'alerte.
### Q10
git clone a donné au collègue le code (pipeline, tests, run_pipeline.py), tout
l'historique Git, les tags et le petit fichier pointeur data/raw.dvc, mais pas
les données. dvc pull lui a donné les vrais fichiers CSV de data/raw, récupérés
depuis le remote DVC, dans la version indiquée par le pointeur (ici data-v2).

### Q11
Si j'avais oublié dvc push à l'étape 4, le remote ne contiendrait que la
version v1. Le pointeur de v2 (md5 94103799...) serait dans Git, mais ses
fichiers n'existeraient pas dans dvc-storage. Le dvc pull du collègue
échouerait avec une erreur (fichiers introuvables dans le remote), il n'aurait
pas le bon data/raw et ne pourrait pas obtenir le total 8835.40.

### Q12
- "Ça marche sur ma machine" : c'est la douleur la plus réduite. Avec git clone
  + dvc pull (étape 6), un collègue obtient le même code, les mêmes données et le
  même total. Le remote DVC (étape 3) rend les données partageables.
- Erreurs silencieuses : en partie. Le pointeur .dvc, dvc status (étape 4) et les
  tags (étape 5) permettent de savoir quelle version des données a produit quel
  chiffre et de voir quand elles ont changé. Mais oublier dvc checkout reste
  silencieux (Q9).
- Livraison lente : un peu, avec les branches (étape 1) qui permettent de
  tester une règle sans toucher à main.
- Étapes manuelles : pas vraiment réduites, tout est encore lancé à la main
  (le bonus dvc repro irait dans ce sens).
  ## TP3

### Q1
En chargeant tout en texte, on ne perd aucune valeur : une quantité -2 ou une
date 2026-13-45 arrivent telles quelles. Si DuckDB devinait les types, il
pourrait échouer au chargement (une date impossible), ou convertir à sa façon.
Le jour où une quantité vaut "deux", DuckDB aurait deviné "entier" sur les
premières lignes puis planté en plein chargement, ou aurait tout converti en
texte sans prévenir. Là, le chargement ne plante jamais, et le choix des types
et des règles est fait ensuite, dans le staging, en SQL, où on le voit et où on
le teste.

### Q2
Le total naïf en SQL (13950.6) est aussi faux que celui de Python : le langage
n'a aucun rapport avec la fiabilité. Un SQL sans règles de nettoyage calcule
sans broncher sur des données sales (doublon, prix 4500, quantité négative).
Ce qui rend un pipeline fiable, ce sont les contrôles (règles, tests, arrêt de
la chaîne) et le fait de savoir d'où vient chaque chiffre, pas la technologie.

### Q3
Le brut est conservé tel quel dans raw.orders. Si une règle est fausse, on
corrige le SQL et on recalcule tout à partir du brut, sans retélécharger ni
redemander le fichier. On peut aussi comparer l'avant et l'après, retrouver
l'origine d'un chiffre (_source_file, _loaded_at) et rejouer l'historique avec
la nouvelle règle. Avec un ETL, on a seulement le résultat nettoyé et la donnée
d'origine est perdue.
### Q4
Dans profiles.yml il faudrait changer le type de l'adaptateur (snowflake ou
bigquery au lieu de duckdb) et remplacer le chemin du fichier par les infos de
connexion : compte, utilisateur, base de données, schéma, entrepôt de calcul.
Le mot de passe ne doit jamais être écrit en clair dans le fichier, ni dans Git.
On le lit dans une variable d'environnement (par exemple {{ env_var('DBT_PASSWORD') }})
ou dans un gestionnaire de secrets, et en CI on le met dans les secrets du
dépôt. On peut aussi garder profiles.yml hors du dépôt.
### Q5
Le staging ne doit pas décider quelles lignes sont mauvaises : son rôle est de
donner les bons types et de réparer ce qui peut l'être. Supprimer la commande
1013 dès le staging la ferait disparaître sans trace, sans motif et sans qu'on
puisse la retrouver. En la gardant avec une date vide, on laisse la couche
suivante la rejeter explicitement avec un motif (invalid_date), et elle finit
dans la quarantaine.

### Q6
Une vue est une requête enregistrée, pas des données stockées : à chaque lecture,
DuckDB relit raw.orders et réapplique la transformation. C'est acceptable pour
du staging parce que la transformation est légère (des conversions de types), la
table brute est petite, et on a toujours des données à jour sans rien stocker en
double. On réserve les tables aux résultats lus souvent ou coûteux à calculer
(les marts).
### Q7
Dans target/compiled/shop/models/intermediate/int_orders_checked.sql,
{{ ref('stg_orders') }} a été remplacé par le vrai nom complet de la vue dans
DuckDB : "warehouse"."main"."stg_orders". dbt transforme le Jinja en SQL pur,
et grâce à ref il sait que stg_orders doit être construit avant ce modèle.

### Q8
row_number() over (partition by order_id order by _row_number) regroupe les
lignes qui ont le même order_id, les classe selon leur position dans le fichier
(_row_number), et leur donne 1, 2, 3... La première ligne de chaque commande
reçoit 1, les répétitions reçoivent 2 ou plus : le "> 1" repère donc les
doublons et garde la première, comme keep="first" en pandas. On a eu besoin de
_row_number parce qu'une table SQL n'a pas d'ordre garanti : sans cette colonne
on ne saurait pas quelle ligne est "la première".

### Q9
L'analyste crée models/marts/revenue_by_product.sql, avec un select qui lit
{{ ref('orders_clean') }} et fait sum(quantity * unit_price) group by product.
Il ne touche à aucun fichier Python : dbt run construit le nouveau modèle après
orders_clean, grâce à ref.
### Q11
| Contrôle du TP1 (Python) | Son équivalent dbt |
| --- | --- |
| check_schema | La déclaration de la source (sources.yml) et le modèle de staging : si une colonne manque, stg_orders échoue (voir jour 3). Les tests not_null et accepted_values vérifient aussi le contenu |
| Les règles 1 à 6 dans clean_orders | Le case when de int_orders_checked.sql (une ligne par règle) |
| check_reject_rate | Le test singulier assert_reject_rate_below_20_percent.sql |
| rejected_orders.csv | La table orders_rejected |
| PIPELINE STOPPED, rien n'est publié | dbt build : un test en échec met FAIL, les modèles en aval sont SKIP, code de sortie 1 |
### Q10
Quand j'ai cassé la règle 5, le test unique_orders_clean_order_id a échoué
(Got 1 result) : le doublon de la commande 1010 n'était plus rejeté et est
entré dans orders_clean. Ont été SKIP : le modèle revenue_by_country et le test
unique_revenue_by_country_country, qui dépendent de orders_clean. dbt build
finit avec ERROR=1 SKIP=2 et un code de sortie 1. C'est ce qu'on veut : le
rapport du directeur n'est pas reconstruit avec des données sales, donc il
garde les chiffres de la veille (c'est l'arrêt de la chaîne du TP1).
### Q12
Les données du test unitaire venaient du fichier YAML lui-même (la section
given : des lignes écrites à la main dans int_orders_checked.yml). dbt remplace
temporairement ref('stg_orders') par ces lignes, sans lire raw.orders ni aucun
CSV. C'est important pour la CI de la séance 4 : sur une machine vierge (le
runner GitHub Actions), il n'y a ni warehouse.duckdb, ni data/raw (les données
sont sous DVC), et le test fonctionne quand même. Il est aussi rapide et ne
dépend pas des données du jour.
### Q13
Si country arrive vide dans la source, le graphe montre tout ce qui est en aval
de raw.orders : stg_orders, int_orders_checked (règle unknown_country, toutes
les lignes seraient rejetées), orders_clean, orders_rejected et
revenue_by_country. Les tests accepted_values et le test du taux de rejet
(assert_reject_rate_below_20_percent) arrêteraient la chaîne. Avec le graphe,
j'ai la réponse en quelques secondes. Sans lui, dans un dossier de 200 scripts
SQL, il faudrait ouvrir les scripts un par un et chercher qui lit la colonne
country, ce qui peut prendre des heures et risque d'oublier une dépendance.
### Q14
|  | Pipeline Python (TP1) | Pipeline dbt (TP3) |
| --- | --- | --- |
| Jour 2 : quel contrôle arrête la chaîne ? | check_reject_rate (taux de rejet 100 %, limite 20 %) | Le test assert_reject_rate_below_20_percent (FAIL 1) |
| Jour 3 : quel message d'erreur ? | PIPELINE STOPPED: Missing columns: ['unit_price'] (check_schema) | Binder Error: Table "src" does not have a column named "unit_price", Candidate bindings: "price" (échec de stg_orders) |
| Le rapport de la veille est-il conservé ? | Oui, revenue_by_country.csv n'est pas réécrit | Oui, revenue_by_country vaut toujours 8835.40 |
| Code de sortie | 1 | 1 |

### Q15
C'est un problème : orders_rejected a été SKIP au jour 2, donc la quarantaine
contient encore les rejets du jour 1 et rien sur les 6 lignes rejetées du
jour 2. Quelqu'un qui la lit croirait que le jour 2 n'a posé aucun problème,
alors que c'est justement le jour où on aurait besoin de voir les lignes
rejetées pour comprendre (les prix en centimes). Pour garder la quarantaine
tout en bloquant le rapport : faire porter le test du taux de rejet sur
orders_clean et orders_rejected (au lieu de int_orders_checked). Les deux
tables sont alors construites avant le test, et seul revenue_by_country, qui
dépend de orders_clean, est SKIP. On peut aussi garder un historique de la
quarantaine en ajoutant les rejets de chaque jour avec _source_file et
_loaded_at, au lieu de remplacer la table.
