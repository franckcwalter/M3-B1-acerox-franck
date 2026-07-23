# Note d'identification des sources — Acerox Métallurgie

> Auteur : Franck — Date : 2026-07-21

## 1. Contexte

entreprise de métallurgie, avec des lignes de production  
~800 salariés, 3 sites de production  
possède déjà un modèle qui détecte les défauts sur la ligne de production  
demande :
- enrichir leur modèle de détection des défauts 


## 2. Demande métier reformulée

Ce que Sébastien a demandé : 
- passer d'un modèle réactif (arrêt de la prod) à un modèle préventif (prévenir les productions défectueuses)
- enrichir notre modèle avec des nouvelles sources

Ce que je comprends qu'il cherche vraiment : 
- avoir un modèle qui détecte des certains seuils des capteurs sur la ligne de production, qui vont prévoir qu'il y aura bientôt des défauts, pour déclencher une maintenance, avant qu'il y ait effectivement des pièces défectueuses
- idéal serait d'avoir un modèle qui anticipe les défauts de qualité 24h à l'avance sur la ligne qui produit le plus de défauts  (pas possibiles sur un délai plus court car certaines données ne sont données qu'une fois par jour) pour déclencher des maintenance 
- KPI : ramener le taux de non-conformité Roubaix L3 sous la moyenne des autres sites 

## 3. Inventaire des sources


| Source | Format | Volume | Fréquence                                                                                                                                                                                                                                                                          | Qualité observée | Risques RGPD | Pertinence métier                                                              |
|---|---|---|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|---|---|--------------------------------------------------------------------------------|
| `capteurs_iot.csv` | CSV | ~3,6 Mo — 51 000 lignes × 7 colonnes | ~1 relevé / 3-5 min par capteur (8 capteurs) — période du 01 au 29 avr. 2026 (1 mois)                                                                                                                                                                                              | 749 NaN sur `vibration_mms` ; couverture inégale (Lyon 2× moins, LINE-1 ~4× LINE-4) ; capteur Roubaix L3 défaillant (vibration figée à 12.0, écart-type 0) ; **pas de cible défaut** | RAS en propre (aucune donnée perso) ; vigilance au croisement avec l'ERP | Source très importante : signaux température/vibration/débit pour le préventif |
| `erp_export.json` | JSON | ~545 Ko — 2 000 ordres × 9 colonnes | événementiel — ~70 ordres lancés / jour, période du 01 au 29 avr. 2026 (1 mois) ; données collectées 1x par jour                                                                                                                                                                   | `statut` déséquilibré (`termine` 1 559 vs autres <200) ; 109 `ouvrier_id` manquants | ⚠️ `ouvrier_id` = donnée personnelle (identifie un employé) — cœur du risque | Contexte de production (produit, quantité, ligne) ; croisement CSV via `site`+`line_id`+fenêtre temporelle |
| `logs_machines.log` | texte brut (log) | ~1,8 Mo — 30 000 lignes | événementiel — ~1 000 lignes/jour, période du 01 au 29 avr. 2026 (1 mois) ; 3 niveaux d'événements : INFO (ex. `machine_started`, `shift_changed`), WARN (ex. `vibration_threshold_approached`, `temperature_drift_detected`), ERROR (ex. `emergency_stop`, `temperature_critical`) | texte brut à parser (regex) ; INFO 22 501 / WARN 5 758 / ERROR 1 741 ; ~60 ERROR/jour, aucun pic ; capteur Roubaix L3 (`SROU-L3-T01`) défaillant → écarter les 132 `vibration_overlimit` qui en viennent (garder les 282 autres) | Aucun id employé en propre ; risque seulement au croisement (`operator_login` + `ouvrier_id` ERP) | Source secondaire : les WARN sont des signaux précurseurs et les ERROR une cible proxy possible (à valider) |

## 4. Recommandations

- **Ingérer en priorité** `capteurs_iot.csv` (51 000 relevés température/vibration/débit) — dédoublonner les **1 000 lignes** strictement en double avant tout usage.
- **Exclure les 5 194 relevés Roubaix L3 figés à 12.0** (capteur défaillant, écart-type 0) et signaler la panne à la maintenance Acerox **avant** tout entraînement — sinon on entraîne sur du bruit.
- **Ingérer `erp_export.json` pour le contexte** (produit, quantité, ligne) et **hasher `ouvrier_id` dès l'ingestion** — vérifier d'abord s'il est utile au modèle (probablement non → le retirer).
- **Reporter `logs_machines.log` à une 2ᵉ itération** : parsing coûteux, à valoriser après preuve sur capteurs + ERP (les **1 741 ERROR** comme cible proxy à valider).

## 5. Points à clarifier avec Sébastien

1. Pourquoi Lyon n'a-t-il qu'un seul capteur ? 
2. Pourquoi 109 ordres ERP sans `ouvrier_id` ?
3. Où est la vérité terrain des défauts (aucune cible dans les 3 sources) ? Est-ce que les ERROR sont un proxy des défauts ? 
4. Le capteur défaillant peut-il être remplacé ? 
5. Quelle latence entre l'acquisition de la donnée et la prédiction (24h OK ou ça doit être en quasi temps réel ?) car ça détermine quelles données on peut utiliser. 

## 6. Limites de cette note

- Pas d'analyse statistique fouillée des sources (M3-B1 = identification,
  pas EDA complète)
- Pas d'AIPD juridique formelle (recommandation : escalader au DPO Acerox)
- Le proxy de défaut n'est pas validé : on assimile les ERROR des logs à des défauts qualité, mais rien
  ne prouve que vibration_overlimit/temperature_critical == pièce réellement défectueuse
- observations effectuées sans réelle fusion des sources 
- pas d'évaluation de l'impact du capteur défaillant sur le modèle actuel

---

*Note produite par Franck, 2026-07-21, dans le cadre du brief M3-B1 Dev-id.*
