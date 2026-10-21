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