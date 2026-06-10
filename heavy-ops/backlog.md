# Backlog heavy-ops

## Contexte

Outil interne : login, dashboard, table volumineuse, analytics, parametres.

## Referentiels

**RGESN** (DINUM 2024) · **GR491** (INR 2022) · **WSG 1.0** (W3C 2023) · **Opquast** (2020+)

---

## US1 — Reduction du polling dashboard

**Contexte** : superviseur d'exploitation / reduire les requetes inutiles et la charge navigateur
**Bonne pratique** : reduction du polling, cache HTTP par route, Page Visibility API — RGESN 5.2, RGESN 5.7, GR491 BP-029, WSG 1.0 §4.1.4
**Priorite** : haute

| # | Tache | Fichier |
|---|---|---|
| T1.1 | Dissocier les intervals : `dashboard` 30 s, `records` 60 s, `settings`/`analytics` one-shot | `OpsApp.tsx` |
| T1.2 | Ecouter `visibilitychange` pour suspendre les intervals quand l'onglet est masque | `OpsApp.tsx` |
| T1.3 | Supprimer le middleware global `Cache-Control: no-store` ; appliquer TTL par route (dashboard 25 s, records 55 s, settings 300 s) | `index.ts` |
| T1.4 | Persister settings et analytics dans `sessionStorage` (one-shot, pas de re-fetch) | `OpsApp.tsx` |

| KPI | Avant | Apres | Gain |
|---|---|---|---|
| Requetes API / min (actif) | 48 (4 × 12) | 4 | **−92 %** |
| Requetes API / min (onglet masque) | 48 | 0 | **−100 %** |
| Requetes settings+analytics / 10 min | 120 | 1 | **−99 %** |

---

## US2 — Pagination de la file active

**Contexte** : analyste metier / premier rendu utile sans charger tout le dataset
**Bonne pratique** : pagination serveur, cache memoire, compression HTTP — RGESN 4.4, RGESN 5.6, GR491 BP-032, WSG 1.0 §4.2.1
**Priorite** : haute

| # | Tache | Fichier |
|---|---|---|
| T2.1 | Ajouter `?page&limit` sur `/api/records`, reponse enveloppee `{ data, total, pageCount }` | `index.ts` |
| T2.2 | Gerer un etat `page` dans `TablePage`, requeter `/api/records?page=N&limit=20`, afficher paginateur | `OpsApp.tsx` |
| T2.3 | Charger `records.json` et `analytics.json` une seule fois au boot (cache memoire), supprimer `readFileSync` dans les handlers | `index.ts` |
| T2.4 | Installer et activer le middleware `compression` (gzip/brotli) avant toutes les routes | `index.ts` |

| KPI | Avant | Apres | Gain |
|---|---|---|---|
| Taille `/api/records` initial | 192 KB | ~4 KB (20 lignes) | **−98 %** |
| Taille apres gzip | 192 KB | ~1,2 KB | **−99 %** |
| Lignes transferees au demarrage | 180 | 20 | **−89 %** |
| I/O disque par requete | 1 `readFileSync` | 0 | **−100 %** |

---

## US3 — Reduction des graphiques analytics

**Contexte** : responsable de pilotage / supprimer les series jamais affichees et les re-renders superflus
**Bonne pratique** : slice serveur, suppression props redondantes, memoisation — RGESN 4.4, RGESN 3.4, GR491 BP-034, WSG 1.0 §4.1.6, Opquast 227
**Priorite** : moyenne

| # | Tache | Fichier |
|---|---|---|
| T3.1 | Limiter `/api/analytics` a 4 series et 6 indicateurs cote serveur (slice avant `res.json`) | `index.ts` |
| T3.2 | Retirer la prop `urgentRecords` de `AnalyticsPage` et supprimer le widget "Dossiers les plus exposes" (doublon du Dashboard) | `OpsApp.tsx` |
| T3.3 | Encapsuler `DashboardPage` et `AnalyticsPage` dans `React.memo` | `OpsApp.tsx` |

| KPI | Avant | Apres | Gain |
|---|---|---|---|
| Series JSON envoyees (`/api/analytics`) | 16 | 4 | **−75 %** |
| Noeuds DOM mini-bars `<span>` | 288 (16×18) | 72 (4×18) | **−75 %** |
| Re-renders `AnalyticsPage` / min | 12 | 2 | **−83 %** |
| Props `urgentRecords` transmises inutilement | 180 records | 0 | **−100 %** |

---

## US4 — Cache assets et compression globale

**Contexte** : admin systeme / eliminer les re-telechargements d'assets et reduire la bande passante
**Bonne pratique** : cache immutable sur assets hashs, compression gzip — RGESN 5.6, RGESN 5.7, GR491 BP-029, Opquast 90
**Priorite** : haute

| # | Tache | Fichier |
|---|---|---|
| T4.1 | Remplacer `maxAge: 0` par `maxAge: '7d', immutable: true` sur `/assets` (noms Vite hashs) | `index.ts` |
| T4.2 | (partage avec T1.3) Supprimer le middleware `no-store` global | `index.ts` |

| KPI | Avant | Apres | Gain |
|---|---|---|---|
| Transfert JSON / session 10 min | ~5 MB | < 200 KB | **−96 %** |
| Assets re-telecharges sur F5 | 100 % | 0 % (304 / cache) | **−100 %** |
| Taille `records.json` (gzip) | 192 KB | ~13 KB | **−93 %** |

---

## Recapitulatif global

| KPI | Avant | Apres | Gain |
|---|---|---|---|
| Requetes API / min | 48 | 4 | **−92 %** |
| Transfert JSON / session 10 min | ~5 MB | < 200 KB | **−96 %** |
| Lignes au 1er rendu `/records` | 180 | 20 | **−89 %** |
| Series graphiques envoyees | 16 | 4 | **−75 %** |
| I/O disque par requete | 1 readFileSync | 0 | **−100 %** |
