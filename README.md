# SemEval 2025 Task 9 - Détection Expliquable de Risques Sanitaires

[![Université de Sherbrooke](https://img.shields.io/badge/UdeS-IFT714-blue)](https://www.usherbrooke.ca)
[![NLP](https://img.shields.io/badge/NLP-Multi--task%20Learning-green)]()
[![RoBERTa](https://img.shields.io/badge/Model-RoBERTa--large-orange)]()
[![Statut](https://img.shields.io/badge/Statut-Archivé%20%E2%80%94%20démo%20uniquement-lightgrey)]()

Ce projet a été réalisé dans le cadre du cours **IFT714 - Traitement automatique des langues naturelles** à l'**Université de Sherbrooke**, session Hiver 2026, sous la forme d'un **projet de recherche expérimentale**.
> Équipe : A. Skrzypczak, R. Bécard, S. Maurel, K. Jemmali

Le projet suit le format d'un article de conférence ACL (4 pages) et comprend 4 livrables :
- 📄 Proposition de projet
- 📊 Rapport d'avancement
- 🎤 Présentation orale en classe
- 📝 Rapport final

L'objectif pédagogique est de mener une **enquête expérimentale rigoureuse** sur un problème NLP réel :
identifier une tâche, explorer la littérature (ACL, EMNLP, SemEval…), soulever les limites actuelles, proposer une solution inovante,
expérimenter empiriquement et analyser les résultats de manière critique.

Nous avons choisi le challenge **[SemEval 2025 Task 9 - Food Hazard Detection](https://food-hazard-detection-semeval-2025.github.io/)**
comme terrain d'application, en y apportant une contribution originale autour de l'**explicabilité extractive**.

> 📂 **Rapport final (format ACL)** → [Groupe_05_Rapport_Final](https://files.adrien-skr.dev/api/public/dl/cy1hF0G4?inline=true)

> 📂 **Dépôt technique (code)** → [IFT714-semeval-2025-task-9](https://github.com/AdrienSkr/IFT714-semeval-2025-task-9)

---

## Contexte

La contamination alimentaire cause environ **420 000 décès par an** dans le monde.  
Automatiser la détection des dangers dans les rapports de rappels de produits permettrait aux agences sanitaires de réagir plus rapidement - mais encore faut-il que le modèle soit **explicable**, pour que les experts puissent valider les alertes instantanément.

Ce projet s'inscrit dans le challenge **[SemEval 2025 Task 9 - Food Hazard Detection](https://food-hazard-detection-semeval-2025.github.io/)**, qui propose de classer les dangers et produits à partir de 6 644 rapports d'incidents annotés.

---

## Objectif de recherche

> **Peut-on prédire la catégorie de risque alimentaire (ST1) tout en extrayant simultanément les segments textuels qui justifient cette prédiction ?**

Les approches existantes du challenge se concentrent sur la performance de classification. Nous proposons une alternative en intégrant directement une **justification extractive** (étiquetage BIO) dans la boucle d'entraînement multi-tâches, afin de réduire le *fossé de fidélité* entre la prédiction et le raisonnement interne du modèle.

---

## Approche & Architecture

Notre modèle est un système **multi-tâches (MTL) basé sur RoBERTa-large** avec une architecture à deux branches :

```
RoBERTa-large (encodeur partagé)
├── Branche Hazard
│   ├── Tête ST1 → classification de catégorie de danger (Focal Loss)
│   └── Tête BIO → extraction du segment justificatif (SoftDice Loss)
└── Branche Product
    ├── Tête ST1 → classification de catégorie de produit
    └── Tête BIO → extraction du segment justificatif
```

**Arbitrages techniques clés :**
- **Focal Loss** pour la classification : compense le fort déséquilibre des classes (longue traîne de 1 142 produits en ST2)
- **SoftDice Loss** pour l'extraction BIO : maximise l'intersection entre segments prédits et réels, robuste aux classes majoritaires "O"
- **Incertitude homoscédastique (Kendall et al., 2018)** : équilibre automatiquement les 4 pertes sans hyperparamètre manuel
- **Annotation BIO par LLM** (Gemini 2.0 Flash, DeepSeek V3, Qwen 380B) via OpenRouter, avec *soft labeling* multi-modèles pour gérer la subjectivité

---

## Résultats

| Métrique | Baseline (BERT) | Modèle final |
|---|---|---|
| F1 Macro ST1 (Hazard) | 0.7087 | **0.7937** |
| ST1+BIO Combined | - | **0.7606** |
| Bio Hazard Dice | - | 0.6972 |
| Bio Product Dice | - | 0.7577 |
| Suffisance (explicabilité) | - | 66.90% |
| Complétude (explicabilité) | - | 57.12% |

Notre modèle surpasse significativement la baseline tout en fournissant une justification extractive robuste. L'analyse d'explicabilité révèle que le modèle extrait en moyenne **17.89% du texte** pour justifier sa prédiction.

**Limite principale :** Le modèle exploite encore la redondance contextuelle des documents plutôt que de s'appuyer exclusivement sur les segments extraits - le fossé de fidélité persiste partiellement (36.65% d'échantillons fidèles).

---

## Ma contribution

Ce projet a été réalisé en équipe de 4. Mes contributions principales (**A. Skrzypczak**) :

- **Architecture modulaire** : mise en place du pipeline configurable (YAML/Hydra), DataLoader, évaluation, système de logging
- **Expérimentations ST1** : entraînement TF-IDF/LR, BERT, comparaison des approches, affinage des hyperparamètres (epochs, batch size, learning rate)
- **Extension du MultiTaskTrainer** pour les données BIO (alignement des tokens, SoftDice Loss, checkpointing, logging)
- **Stratégie d'annotation BIO** : orchestration de l'annotation LLM (vLLM, OpenRouter), export au format BIO avec soft labeling
- **Expérimentation du modèle ST1+BIO** depuis Google Colab
- **Documentation** : README, interprétation des résultats, contributions aux rapports de projet

---

## Ce que j'ai appris

- Concevoir une architecture multi-tâches NLP de bout en bout (de la donnée brute aux métriques d'explicabilité)
- Gérer le déséquilibre de classes extrême en NLP (longue traîne, Focal Loss, augmentation ciblée)
- Annoter un corpus sans ground truth via ensemble de LLMs + soft labeling
- Utiliser Hydra pour des expérimentations reproductibles à grande échelle sur Colab/GPU cloud
- Lire et implémenter des papiers de recherche récents (NeurIPS, ACL, SemEval)

---

## Références clés

- [SemEval 2025 Task 9](https://food-hazard-detection-semeval-2025.github.io/) - Randl et al., 2025
- Kendall et al. (2018) - *Multi-task learning using uncertainty to weigh losses*
- DeYoung et al. (2020) - *ERASER: A benchmark to evaluate rationalized NLP models*

---

## Contact

**Adrien Skrzypczak** - [LinkedIn](https://www.linkedin.com/in/adrienskrzypczak/) · [GitHub](https://github.com/AdrienSkr)

> 📌 *Projet archivé - réalisé à des fins académiques, aucune nouvelle fonctionnalité prévue.*
