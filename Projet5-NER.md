# Cahier des charges du projet : Reconnaissance d’Entités Nommées (NER) pour la Darija marocaine

**Extraction d’entités marocaines à partir de textes hétérogènes et annotation semi-automatique**

---

## 1. Contexte général

La **Reconnaissance d’Entités Nommées (Named Entity Recognition – NER)** est une tâche fondamentale du Traitement Automatique du Langage Naturel (NLP). Elle consiste à identifier et classifier dans un texte des mentions d’entités telles que des personnes, lieux, organisations, dates, etc.

Dans le contexte de la **Darija marocaine**, la NER pose des défis spécifiques :

* Absence de normalisation orthographique.
* Mélange Darija / Arabe standard / Français / Anglais (*code-switching*).
* Usage fréquent de l’**Arabizi** (alphabet latin + chiffres).
* Références locales (villes, institutions, marques) peu couvertes par les modèles standards.

Ce projet vise à constituer un **corpus annoté de haute qualité** via une méthodologie stricte (Pipeline **A → B → C**) et une **labellisation semi-automatique assistée par LLM**.

---

## 2. Objectifs du projet

### 2.1 Objectifs pédagogiques

À l’issue du projet, l’étudiant devra être capable de :

* Collecter et nettoyer des données textuelles réelles en Darija.
* Appliquer un schéma d’entités riche et adapté au contexte marocain.
* Mettre en œuvre une annotation semi-automatique (*LLM + Humain*).
* Entraîner et comparer des modèles NER (supervisés et/ou fine-tuning).

### 2.2 Compétences visées

* NLP pour langues peu dotées (*Low-resource languages*).
* Annotation structurée (*Token / Span level*).
* MLOps : tracking expérimental et reproductibilité.

---

## 3. Périmètre et Organisation

* **Travail de groupe** : projet réalisé en binôme ou en équipe.
* **Langage** : Python.
* **Langue cible** : Darija marocaine (script arabe et latin/arabizi).
* **Organisation des données** :

  * Le dataset global est pré-divisé.
  * Chaque groupe est responsable de son propre lot de données (`data per groups/`).
  * Exemple : *Groupe 1 →* `ner_group_1.csv`.

### Outils obligatoires

* **Labellisation** : Label Studio (ou équivalent).
* **Tracking expérimental** : MLflow ou Weights & Biases.

---

## 4. Définition de la tâche et Schéma d'Annotation (ÉTAPE A)

### 4.1 Nature de la tâche

Identifier les mentions d’entités dans le texte et les classer selon un schéma prédéfini de **14 classes**.

### 4.2 Schéma d’entités (14 classes)

| Entité      | Description                                | Exemples (Arabe/Natif)  | Exemples (Arabizi/Latin) |
| ----------- | ------------------------------------------ | ----------------------- | ------------------------ |
| PERSON      | Personnes réelles ou fictives              | شيكسبير، السلطان سليمان | shiksbir, sultan slimane |
| NORP        | Nationalités, groupes religieux/politiques | مغاربة، فرنسيين         | mgharba, fransawyin      |
| GPE         | Pays, villes, provinces                    | المغرب، الرباط، مكناس   | lmaghrib, rabat, meknes  |
| LOC         | Lieux géographiques naturels               | الصحراء، المحيط الأطلسي | ss7ra, lb7ar             |
| ORG         | Entreprises, institutions, clubs           | الوداد، ONCF، CIH Bank  | wydad, cih bank          |
| GOV         | Administrations gouvernementales           | وزارة الصحة             | wizarat ssi7a            |
| MEDIA       | Médias, chaînes TV, réseaux sociaux        | قناة أجي تفهم، 2M       | aji tfham, 2m            |
| EVENT       | Événements culturels, sportifs             | كأس العرش، موازين       | kas l3arch, mawazine     |
| WORK_OF_ART | Livres, films, chansons                    | مسرحية تاجر البندقية    | tajir lbnd9ia            |
| LAW         | Lois, traités, conventions                 | معاهدة لالة مغنية       | mo3ahadat lala mghnia    |
| LANGUAGE    | Langues nommées                            | العربية، الفرنسية       | l3arbiya, lfransiya      |
| DATE        | Temps absolu ou relatif                    | دابا، اليوم، القرن 17   | daba, lyoum, l9arn 17    |
| ORDINAL     | Rang, ordre                                | الأول، الثاني           | lwl, tani                |
| CARDINAL    | Nombres                                    | ثمانية أجزاء            | 8 ajza2, tmnya           |

### 4.3 Règles d’or de l’annotation

* **Span maximal** : annoter l’entité complète.

  * ✅ `[جامعة محمد الخامس] (ORG)`
  * ❌ `جامعة [محمد] (PERSON) الخامس`
* **Code-switching** : annoter normalement les termes étrangers.

  * `viva [Raja Casablanca] (ORG)`
* **Arabizi accepté** : toujours annoter.

  * `knt f [Casa] (GPE)`
* **Temps relatif = DATE**.

  * `[daba] (DATE)`, `[ssimana jaya] (DATE)`

---

## 5. Sources de données

* YouTube transcription

**Volume indicatif** : ~20 000 phrases (réparties entre les groupes).

### Nettoyage

* Suppression des URLs techniques, spam, emojis excessifs.
* Pas de normalisation agressive.
* Anonymisation des données sensibles.

---

## 6. Pipeline d'Annotation (ÉTAPE B & C)

### Phase 1 : Gold Standard (ÉTAPE B)

* 1000 phrases aléatoires par groupe.
* Annotation **100 % manuelle**.
* Sert de référence qualité.

### Phase 2 : Annotation Semi-Automatique (ÉTAPE C)

#### 1. Pré-annotation par LLM

Format JSON strict :

```json
{
  "text": "مشيت لكازا فماتش ديال الوداد",
  "entities": [
    {"start": 6, "end": 10, "label": "GPE"},
    {"start": 18, "end": 24, "label": "ORG"}
  ]
}
```

#### 2. Validation humaine (Label Studio)

* Correction des labels.
* Ajustement des spans.
* Ajout des entités manquées.

#### 3. Contrôle qualité

* Calcul du taux de correction des pré-annotations.

---

## 7. Modélisation NER

### 7.1 Approches attendues

* **Baseline** : dictionnaires, CRF simple.
* **Deep Learning** :

  * Fine-tuning mBERT, XLM-R.
  * Modèles régionaux : CamelBERT, DziriBERT.
  * BiLSTM-CRF (optionnel).

### 7.2 Évaluation

* Precision, Recall, F1-score.
* Analyse par type d’entité.
* Matrice de confusion (ex : GPE vs LOC).

---

## 8. Suivi expérimental et Livrables

### 8.1 Tracking

* Utilisation obligatoire de **MLflow** ou **Weights & Biases**.

### 8.2 Livrables

* **Corpus NER final** (CoNLL ou JSONL).
* **Code source** (pipeline complet).
* **Rapport technique** :

  * Méthodologie.
  * Statistiques du corpus.
  * Analyse des performances.
* **Dashboard de tracking**.

---

## 9. Critères d’évaluation

| Critère            | Pondération |
| ------------------ | ----------- |
| Qualité du corpus  | 50 %        |
| Pipeline technique | 15 %        |
| Modélisation       | 10 %        |
| Tracking & MLOps   | 10 %        |
| Analyse & rapport  | 15 %        |