# Backlog heavy-ops

## Contexte du projet

Le projet represente un outil interne avec login, dashboard, table volumineuse, analytics et parametres.

## Format attendu

Completer au minimum 3 user stories.
Remplacer chaque champ entre crochets par votre contenu.

## User story 1

- Contexte: En tant que *superviseur d'exploitation*, je veux *limiter la frequence de rafraichissement du tableau de bord*, afin de *reduire les requetes inutiles et la charge navigateur*.
- Objectif: Passer d'un rafraichissement toutes les 5 secondes a un rafraichissement adapte au besoin, avec au moins 50 % d'appels en moins sur une session de 10 minutes.
- Bonne pratique d eco-conception ciblee: Reduction du polling et mise en cache locale des donnees non critiques.
- KPI associe: Nombre d'appels API par minute et consommation CPU moyenne pendant la navigation.
- Repo ou ecran concerne: heavy-ops / page Dashboard et endpoint /api/dashboard.
- Critere de reussite: Le dashboard reste lisible avec des donnees a jour, tout en diminuant de moitie le volume de requetes repetées mesure sur une session type.
- Niveau de priorite: haute

## User story 2

- Contexte: En tant *qu'analyste metier*, je veux *charger la file active par lots*, afin de *consulter rapidement les dossiers sans subir un temps de chargement excessif*.
- Objectif: Afficher un premier jeu de donnees utile en moins de 2 secondes et limiter le volume initial rendu a 20 lignes visibles.
- Bonne pratique d eco-conception ciblee: Pagination ou chargement progressif au lieu d'un chargement complet de gros datasets.
- KPI associe: Temps jusqu'au premier rendu utile et nombre de lignes chargees au demarrage.
- Repo ou ecran concerne: heavy-ops / ecran TablePage et endpoint /api/records.
- Critere de reussite: La table permet de naviguer entre les dossiers sans charger toute la liste d'un coup, avec une latence percue plus faible et un debut d'affichage rapide.
- Niveau de priorite: haute

## User story 3

- Contexte: En tant que *responsable de pilotage*, je veux *afficher seulement les indicateurs utiles dans l'espace analytics*, afin de *reduire la densite visuelle et les calculs de visualisation superflus*.
- Objectif: Conserver au maximum 4 graphiques prioritaires et 6 indicateurs synthese, avec un ecran plus rapide a parcourir.
- Bonne pratique d eco-conception ciblee: Priorisation de l'information et reduction des elements decoratifs ou redondants.
- KPI associe: Nombre de composants graphiques affiches et temps de rendu de l'ecran analytics.
- Repo ou ecran concerne: heavy-ops / page AnalyticsPage et donnees /api/analytics.
- Critere de reussite: L'utilisateur identifie les signaux critiques en quelques secondes, avec moins de visuels affiches et moins de re-renders inutiles.
- Niveau de priorite: moyenne

## Notes

- Vous pouvez ajouter d autres user stories si necessaire.
- Le niveau de detail attendu doit permettre une priorisation exploitable.
