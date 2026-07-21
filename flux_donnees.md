# Schéma des flux de données — Acerox Métallurgie

> Schéma Mermaid à compléter. Doit montrer :
> - **Sources** (capteurs IoT, ERP, logs, *bonus rapports `.md`*)
> - **Ingestion** (à concevoir en M3-B2)
> - **BDD pivot** (à modéliser en M3-B2)
> - **Modèle existant** Acerox (placeholder, hors-sujet ici)
>
> Légende explicite : qui produit, qui consomme, contraintes.

## Schéma

```mermaid
flowchart LR
    %% TODO — Compléter avec tes 3 sources retenues + 1 bonus si traité

    SRC1[📡 capteurs_iot.csv<br/>CSV — 51k lignes / 3,6 Mo<br/>~1 relevé / 3-5 min par capteur<br/>continu]
    SRC2[📋 erp_export.json<br/>JSON — 2k ordres / 545 Ko<br/>~70 ordres / jour<br/>batch]
    SRC3[📝 logs_machines.log<br/>texte brut — 30k lignes / 1,8 Mo<br/>~1000 lignes / jour<br/>continu]

    INGEST[🔄 Ingestion<br/>à concevoir en M3-B2]
    BDD[(🗄️ BDD pivot<br/>SQLite)]
    MODEL[🧠 Modèle existant Acerox<br/>prédiction défauts qualité]

    SRC1 -->|temps réel| INGEST
    SRC2 -->|batch quotidien| INGEST
    SRC3 -->|temps réel — parsing à faire| INGEST
    INGEST -->|normalisation + dédup| BDD
    BDD -->|consommée par| MODEL

    %% existant (sources + modèle) = trait plein ; à construire (M3-B2) = pointillé
    classDef existant fill:#e1f5ff,stroke:#0277bd
    classDef tofix fill:#fff4e1,stroke:#c97a00,stroke-dasharray: 5 5
    class SRC1,SRC2,SRC3,MODEL existant
    class INGEST,BDD tofix
```

## Légende

- **Producteurs** : 3 sources Acerox — capteurs IoT (CSV, temps réel), ERP (JSON, batch quotidien), logs machines (texte, temps réel).
- **Consommateur final** : le modèle de prédiction des défauts qualité.
- **Existant (trait plein)** : les 3 sources et le modèle de prédiction.
- **À construire (pointillé)** : l'ingestion (normalisation + déduplication) et la BDD pivot.
- **Contraintes critiques** : `ouvrier_id` de l'ERP = donnée personnelle (RGPD) ; fraîcheur hétérogène (capteurs/logs continus vs ERP journalier) ; qualité (doublons CSV, capteur Roubaix L3 figé, logs à parser).

## Décisions associées

- Source(s) retenues en priorité : `capteurs_iot.csv` (signaux température/vibration/débit) et `logs_machines.log` (les ERROR comme proxy du défaut).
- Source(s) écartées : `erp_export.json` — export **batch quotidien**, fraîcheur incompatible avec un scoring préventif quasi temps réel (+ enjeu RGPD `ouvrier_id`).
- Source bonus (rapports `.md`) traitée ? non — mission RAG optionnelle non traitée.

---

*Schéma produit par franck, 2026-07-21, dans le cadre du brief M3-B1 Dev-Id.*
