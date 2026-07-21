# Note d'identification des sources — Acerox Métallurgie

> Auteur : Franck — Date : 2026-07-21

## 1. Contexte

entreprise de métallurgie, avec des lignes de production   
possède déjà un modèle qui détecte les défauts sur la ligne de production  
demande :
- enrichir leur modèle de détection des défauts 
- passer d'un modèle réactif (arrêt de la prod) à un modèle préventif (prévenir les productions défectueuses)


## 2. Demande métier reformulée

Ce que Sébastien a demandé : passer d'un modèle réactif (arrêt de la prod) à un modèle préventif (prévenir les productions défectueuses)

Ce que je comprends qu'il cherche vraiment : avoir un modèle qui détecte des certains seuils des capteurs sur la ligne de production, qui vont prévoir qu'il y aura bientôt des défauts, pour déclencher une maintenance, avant qu'il y ait effectivement des pièces défectueuses. 

## 3. Inventaire des sources


| Source | Format | Volume | Fréquence | Qualité observée | Risques RGPD | Pertinence métier                                                              |
|---|---|---|---|---|---|--------------------------------------------------------------------------------|
| `capteurs_iot.csv` | CSV | ~3,6 Mo — 51 000 lignes × 7 colonnes | ~1 relevé / 3-5 min par capteur (8 capteurs) — période du 01 au 29 avr. 2026 (1 mois) | 749 NaN sur `vibration_mms` ; couverture inégale (Lyon 2× moins, LINE-1 ~4× LINE-4) ; capteur Roubaix L3 défaillant (vibration figée à 12.0, écart-type 0) ; **pas de cible défaut** | RAS en propre (aucune donnée perso) ; vigilance au croisement avec l'ERP | Source très importante : signaux température/vibration/débit pour le préventif |
| `erp_export.json` | JSON | ~545 Ko — 2 000 ordres × 9 colonnes | événementiel — ~70 ordres lancés / jour, période du 01 au 29 avr. 2026 (1 mois) | `statut` déséquilibré (`termine` 1 559 vs autres <200) ; 109 `ouvrier_id` manquants | ⚠️ `ouvrier_id` = donnée personnelle (identifie un employé) — cœur du risque | Contexte de production (produit, quantité, ligne) ; croisement CSV via `site`+`line_id`+fenêtre temporelle |
| `logs_machines.log` | texte brut (log) | ~1,8 Mo — 30 000 lignes | événementiel — ~1 000 lignes/jour, période du 01 au 29 avr. 2026 (1 mois) ; 3 niveaux d'événements : INFO (ex. `machine_started`, `shift_changed`), WARN (ex. `vibration_threshold_approached`, `temperature_drift_detected`), ERROR (ex. `emergency_stop`, `temperature_critical`) | texte brut à parser (regex) ; INFO 22 501 / WARN 5 758 / ERROR 1 741 ; ~60 ERROR/jour, aucun pic ; capteur Roubaix L3 (`SROU-L3-T01`) défaillant → écarter les 132 `vibration_overlimit` qui en viennent (garder les 282 autres) | Aucun id employé en propre ; risque seulement au croisement (`operator_login` + `ouvrier_id` ERP) | Source secondaire : les WARN sont des signaux précurseurs et les ERROR une cible proxy possible (à valider) |

## 4. Recommandations

- le csv avec les capteurs IOT est la source la plus importante, elle contient les signaux température/vibration/débit qui serviront à détecter les possibiles  
- ensuite, logs_machines.log  contiennent un proxy du défaut (les ERROR)
- le json de l'ERP peut être utile pour le contexte (produit, ligne), à utiliser en anonymisant ouvrier_id pour éviter les problèmes RGPD

## 5. Points à clarifier avec Sébastien

1. Pourquoi Lyon n'a-t-il qu'un seul capteur ? 
2. Pourquoi 109 ordres ERP sans `ouvrier_id` ?
3. Où est la vérité terrain des défauts (aucune cible dans les 3 sources) ? Est-ce que les ERROR sont un proxy des défauts ? 
4. Roubaix LINE-3 vibre 3× la normale et concentre les erreurs : est-ce une vraie anomalie machine, ou un capteur déréglé ? 
5. Confirmer la fréquence réelle de livraison de l'export ERP (quotidien?).

## 6. Limites de cette note

- Pas d'analyse statistique fouillée des sources (M3-B1 = identification,
  pas EDA complète)
- Pas d'AIPD juridique formelle (recommandation : escalader au DPO Acerox)
- Le proxy de défaut n'est pas validé : on assimile les ERROR des logs à des défauts qualité, mais rien
  ne prouve que vibration_overlimit/temperature_critical == pièce réellement défectueuse
- observations effectuées sans réelle fusion des sources 
- décalage de fraîcheur entre sources : capteurs en continu vs ERP en export journalier (contrainte pour un usage temps réel)

---

*Note produite par Franck, 2026-07-21, dans le cadre du brief M3-B1 Dev-id.*
