# Notes d'entretien — Sébastien Marchand (Acerox)


**Date** : 2026-07-21 — **Durée** : 30 min — **Présents** : Sébastien Marchand (chef de projet industrialisation Acerox) + Franck (FastIA)

---

## 1. Besoin métier

> Quelle décision Sébastien veut-il améliorer ? Quel KPI métier ?

**Dit** :
- besoin : limiter le nombre de défauts sur la ligne de production
- modèle actuel : modèle analyse les données de capteurs sur les lignes pour donner une info sur la conformité des pièces sorties
- passer d’un modèle réactif (arrêt de la production) à un modèle préventif (détection d’un probable défaut avant l’arrêt de la production)

**Interprété** : 
- avoir un modèle qui détecte des certains seuils des capteurs sur la ligne de production, qui vont prévoir qu'il y aura bientôt des défauts, pour déclencher une maintenance, avant qu'il y ait effectivement des pièces défectueuses.

## 2. Sources et formats

> Qu'a-t-il à disposition ? Où ? Sous quel format ?

**Dit** : 
- des données de capteurs IOT sur différentes lignes de production (CSV)
- JSON de données qui viennet de l'ERP 
- des logs d'événements sur les machines (texte brut)

**Interprété** : 
- différents types de données, pas de politique interne de normalisation des formats de données 
- nécessité de fusion des sources 

## 3. Volumétrie et fréquence

> Combien de données ? À quelle cadence arrivent-elles ?

**Dit** : 
- 1 mois de données 
- IOT une mesure par seconde 
- données ERP : export quotidien
- 3 sites au total, dont 2 qui font de la production
- les mêmes machines sur les différents sites de production 
- 2000 ordres de production par jour


**Interprété** : 
- 1 mois de données qui est représentatif d'un mois lambda 
- volume de données a priori important (dizaines de milliers de ligne)

## 4. Contraintes (RGPD, sécurité, propriété)

> Quelles obligations légales ? Qui possède les données ? Quels accès
> sécurisés ?

**Dit** : 
- des données qui peuvent être sensibles (données personnelles)
- entretien à prévoir avec le DPO 
- coût élevé d'une pièce défectueuse (12000 euros)
- infra qui supporte n'importe quelle taille de modèle

**Interprété** :
- l'analyse RGPD n'a pas été faite, à nous de faire un premier audit, avant de voir en détail avec leur DPO
- le taux de recall sera très important, il ne faut pas manquer une pièce défectueuse car coûteuse pour l'entreprise, quitte à parfois antciper des maintenances
- infra sans contrainte de taille de modèle : l'arbitrage se joue sur la performance/recall, pas sur la taille du modèle
- temps d'inférence évoqué : besoin de prédiction en quasi temps réel pour agir avant l'arrêt de ligne

## 5. Critères de succès

> Comment Sébastien saura-t-il qu'on a réussi ?

**Dit** : 
- réduire à moins de 20% de non-conformité sur la ligne la plus problématique sous 3 mois
- le modèle doit prévenir en amont la non-conformité de pièces pour ne pas avoir d’arrêt de la ligne brutal 
- utilisateurs finaux : l’équipe de maintenance des lignes

**Interprété** : 
- baisse du taux de non-conformité en-dessous d'un certain seuil 
- utilisateurs = équipe maintenance, donc la sortie doit être actionnable par eux (alerte + seuil clair), pas un score abstrait

---

## Questions restées sans réponse

- Quels changements ou évolutions depuis la mise en route du modèle ?
- Limites ou incohérences du modèle actuel ?
- Combien de lignes de données ?
- Combien de colonnes (types de données) ?
- Taille de modèle actuel ?
 
## Mes impressions à chaud (10 min après)

- différents types de données 
- besoin de croiser les sources 
- possibilité de données peu pertinentes pour le modèle final
- possibilité de données sensibles directes ou par croisement 

---

*Notes d'entretien produites par franck, 2026-07-21.*
