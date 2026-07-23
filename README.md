# M3-B1 — Squelette repo (entretien client + identification sources)

> **Repo template GitHub.** Clique sur **« Use this template »** en haut à
> droite de cette page → **Create a new repository** → nomme-le
> `M3-B1-<client>-<prénom>` sur **ton** compte GitHub personnel (le nom
> du client te sera révélé mardi 9h).

---

## 🗣️ Entretien client

Le projet démarre par un entretien avec Sébastien Marchand (Acerox), qui
cadre le besoin métier, les sources disponibles et les contraintes RGPD.
Les notes complètes (dit vs interprété, 5 catégories) sont dans
[`notes_entretien.md`](./notes_entretien.md).

---

## 🚀 Démarrage (4 commandes)

```bash
# 0. Clone ton repo perso fraîchement créé
git clone git@github.com:<ton-user>/M3-B1-<client>-<prenom>.git
cd M3-B1-<client>-<prenom>

# 1. Environnement virtuel
python -m venv .venv && source .venv/bin/activate     # Linux/macOS
# .venv\Scripts\activate                              # Windows

# 2. Dépendances
pip install -r requirements.txt

# 3. Vérification
jupyter notebook notebooks/M3-B1_template.ipynb       # → s'ouvre dans le navigateur
```

Si ces 4 commandes marchent, ton poste est prêt.

---

## 📁 Structure du repo

```
M3-B1-acerox-franck/
├── data/                                # gitignored (sauf jeux jouets provenance)
│   ├── capteurs_iot.csv                 # source 1 — ~51k lignes
│   ├── erp_export.json                  # source 2 — ~2k ordres (enjeu RGPD : ouvrier_id)
│   ├── logs_machines.log                # source 3 — ~30k lignes de texte brut
│   ├── capteurs_site_A.csv              # versionné — jeu jouet fusion/provenance
│   ├── capteurs_site_B.csv              # versionné — 2e export comparable
│   └── rapports_techniques_2024/        # corpus RAG (bonus) — rapports .md
├── notebooks/
│   ├── M3-B1_franck.ipynb               # exploration des 3 sources
│   ├── M3-B1_fusion_provenance.ipynb    # fusion 2 exports + colonne source
│   ├── M3-B1_rag_prepa_franck.ipynb     # prépa RAG (bonus) — chunk / embedding / ChromaDB
│   └── chroma_acerox/                   # base ChromaDB générée (bonus)
├── ressources/                          # mini-cours d'appui
│   ├── README.md
│   ├── 01_Entretien_client_essentiel.md
│   ├── 02_Cartographie_sources_essentiel.md
│   ├── 03_Risques_RGPD_multisources_essentiel.md
│   ├── 04_Schema_Mermaid_flux_essentiel.md
│   ├── 05_Note_identification_essentiel.md
│   ├── 06_RAG_prepa_essentiel.md
│   └── liens_officiels.md
├── identification_sources.md            # livrable principal (2-3 pages)
├── flux_donnees.md                      # schéma Mermaid + légende
├── notes_entretien.md                   # notes de l'entretien client
├── CLAUDE.md                            # guidage pour l'assistant
├── requirements.txt
├── .gitignore
└── README.md
```

---

## 🗺️ Cartographie des sources

Les 3 sources hétérogènes (`capteurs_iot.csv`, `erp_export.json`,
`logs_machines.log`) sont explorées en lecture seule — format, volume,
fréquence, qualité, risques RGPD — sans aucune transformation.

- **Exploration** : [`notebooks/M3-B1_franck.ipynb`](./notebooks/M3-B1_franck.ipynb) — une cellule d'observations par source (doublons, manquants, capteur défaillant, `ouvrier_id`…).
- **Synthèse** : [`identification_sources.md`](./identification_sources.md) — inventaire, risques et recommandations, lisible par un décideur.
- **Flux de données** : [`flux_donnees.md`](./flux_donnees.md) — schéma Mermaid des flux entre sources.

En bref :

- **`capteurs_iot.csv`** (CSV, ~51 k lignes) — signaux température / vibration / débit, cœur du préventif ; capteur Roubaix L3 défaillant (vibration figée) et 1 000 doublons à écarter. RGPD faible.
- **`erp_export.json`** (JSON, ~2 k ordres) — contexte de production (produit, quantité, ligne), export quotidien ; porte `ouvrier_id`, la donnée personnelle au cœur du risque RGPD.
- **`logs_machines.log`** (texte, ~30 k lignes) — événements INFO/WARN/ERROR à parser, source secondaire ; risque RGPD seulement au croisement avec l'ERP.

---

## 🔎 Préparation RAG

Un quatrième jeu de données, `data/rapports_techniques_2024/` (cinq rapports
techniques en français), contient du texte non structuré préparé pour une
recherche par similarité sémantique dans
[`notebooks/M3-B1_rag_prepa_franck.ipynb`](./notebooks/M3-B1_rag_prepa_franck.ipynb) :
chaque rapport est découpé par titre Markdown, transformé en embeddings via un
modèle local (`all-MiniLM-L6-v2`, sans clé API), puis indexé dans une base
ChromaDB persistante que l'on interroge en langage naturel.

Le travail s'arrête à la récupération des passages. L'exécution nécessite
`chromadb` et `sentence-transformers` (voir `requirements.txt`).

