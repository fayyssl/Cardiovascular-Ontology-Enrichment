# Enrichissement automatique d'une ontologie cardiovasculaire (CVDO) via RAG + Multi-LLM

Pipeline qui extrait automatiquement des relations sémantiques (triplets) depuis des
recommandations cliniques cardiovasculaires, les valide par consensus multi-modèles,
puis les intègre dans une ontologie biomédicale (CVDO) alignée sur SNOMED-CT.

Projet issu d'un mémoire de recherche en NLP appliqué à la santé (M1 TAL/NLP).

## Le problème

Enrichir manuellement une ontologie médicale (relations facteur-de-risque, traitement,
symptôme, complication, surveillance entre entités cliniques) est lent et coûteux.
Ce pipeline automatise l'extraction de ces relations à partir de guidelines cliniques
en langage naturel, avec un contrôle qualité qui limite le risque d'hallucination
propre aux LLM.

## Approche

```
Guideline clinique (PDF)
        │
        ▼
[1] Extraction de phrases cliniques pertinentes (regex + mots-clés + NER biomédical)
        │
        ▼
[2] Indexation RAG (ChromaDB + embeddings BioBERT)
        │
        ▼
[3] Extraction multi-LLM (GPT-4o, DeepSeek, Claude) sur chaque requête sémantique
        │
        ▼
[4] Consensus (≥ 2/3 modèles d'accord) + normalisation des synonymes + déduplication
        │
        ▼
[5] Validation symbolique (relations autorisées, filtrage des entités vagues)
        │
        ▼
[6] Alignement SNOMED-CT + export CSV/RDF
        │
        ▼
[7] Intégration dans l'ontologie CVDO (alignement multi-niveaux : exact →
    nettoyage de suffixes → similarité cosinus BioBERT ≥ 0.98)
```

Deux notebooks composent le pipeline :

- **`notebooks/01_extraction_pipeline.ipynb`** — extraction des phrases, RAG,
  extraction multi-LLM, consensus, validation, alignement SNOMED-CT.
- **`notebooks/02_ontology_integration.ipynb`** — intègre les triplets validés
  dans l'ontologie OWL (CVDO), avec alignement sémantique multi-niveaux
  (inspiré de Wu et al., 2025).

## Résultats

Sur un corpus de guidelines cliniques sur l'hypertension artérielle :

| Métrique | Valeur |
|---|---|
| Triplets extraits (après consensus + validation) | **126** |
| Consensus 3/3 modèles | 40 (32 %) |
| Consensus 2/3 modèles | 86 (68 %) |

Distribution par type de relation :

| Relation | Nombre |
|---|---|
| `est_traite_par` (a pour traitement) | 45 |
| `est_facteur_de_risque_de` (facteur de risque) | 45 |
| `est_complication_de` (complication) | 20 |
| `est_surveille_par` (surveillance) | 15 |
| `est_symptome_de` (symptôme) | 1 |

Les résultats complets sont dans [`data/results/triplets_consensus.csv`](data/results/triplets_consensus.csv).

## Stack technique

- **RAG** : ChromaDB + embeddings BioBERT (`dmis-lab/biobert-base-cased-v1.2`)
- **Extraction** : consensus multi-LLM (GPT-4o, DeepSeek, Claude Haiku)
- **NER biomédical** : `d4data/biomedical-ner-all`
- **Ontologie** : RDFlib (OWL), validation symbolique, alignement SNOMED-CT
- **Alignement sémantique** : similarité cosinus BioBERT (approche multi-niveaux)

## Installation

```bash
git clone <url-du-repo>
cd cvdo-repo
pip install -r requirements.txt
cp .env.example .env
# renseigner OPENAI_API_KEY, DEEPSEEK_API_KEY, ANTHROPIC_API_KEY dans .env
```

Les notebooks ont été développés sur Google Colab (cellules `google.colab.files`
pour l'upload/téléchargement). Pour une exécution locale, remplacer ces cellules
par une lecture/écriture de fichiers classique.

## À propos du corpus source

Le pipeline s'applique à des guidelines cliniques cardiovasculaires publiques.
**Les documents sources eux-mêmes ne sont pas inclus dans ce dépôt** — seul le code
et les résultats structurés (triplets extraits) sont versionnés. Si vous souhaitez
reproduire le pipeline, fournissez vos propres documents sources dans
`data/input/` (dossier non versionné, cf. `.gitignore`) et vérifiez les droits de
réutilisation de vos sources avant toute redistribution.

## Limites

- Le seuil de consensus (≥ 2/3 LLM) limite mais n'élimine pas le risque d'erreur
  d'extraction ; une validation experte (HITL) reste recommandée avant intégration
  clinique réelle.
- L'alignement SNOMED-CT est fondé sur un mapping simplifié (5 relations →
  5 codes), pas une correspondance ontologique complète.
- Le pipeline a été testé sur un domaine clinique restreint (hypertension
  artérielle) ; sa généralisation à d'autres pathologies cardiovasculaires
  reste à valider.

## Licence

MIT — voir [LICENSE](LICENSE).
