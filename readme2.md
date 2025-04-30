# Trait Ontology Extraction, Mapping and Evaluation System

A comprehensive system for extracting gene-trait relationships from scientific literature, mapping trait names to standardized ontology terms, and evaluating these mappings using advanced LLM models.

## System Overview

Our system employs a three-step process:

1. **Gene-Trait Relationship Extraction**: Extract gene-trait relationships from scientific publications using LLM models
2. **Trait Ontology Mapping**: Map extracted trait names to standardized Trait Ontology (TO) terms and IDs
3. **Mapping Evaluation**: Evaluate the quality of semantic mappings using LLM-based validation

## Features

- **Relationship Extraction**: Extract gene-trait relationships from scientific publications using various LLM models
- **Ontology Parsing**: Parse OBO format files into structured data
- **Trait Evaluation**: Map trait names to ontology terms using a three-tier matching approach:
  1. Exact name matching
  2. Synonym matching
  3. Semantic similarity matching using embeddings
- **Multiple LLM Models**: Support for several OpenAI models with detailed comparative analysis
- **Comprehensive Evaluation**: Detailed metrics on match quality and confidence
- **LLM-Based Mapping Validation**: Generate responses from advanced LLMs to evaluate semantically matched traits

## Workflow

Our system follows a three-step workflow for extracting, mapping, and evaluating trait information:

![workflow-chart](https://github.com/user-attachments/assets/d355f6e3-50a7-4754-99dc-42c3f95fc033)


## Input Format

The system accepts gene-trait extraction tables in CSV format with the following columns:

```
pmid, gene, species, trait_name, relation_type, evidence, method
```

Example entries:

| pmid | gene | species | trait_name | relation_type | evidence | method |
|------|------|---------|------------|---------------|----------|--------|
| 37897039 | TaLAX1 | Triticum aestivum | regeneration efficiency | enhances | Overexpression of TaLAX1 markedly enhances regeneration efficiency... | gene overexpression |
| 39062606 | TaGOGAT3-D | Triticum aestivum | nitrogen use efficiency | influences | The expression level of TaGOGATs and the enzyme activity... | expression analysis |
| 37907517 | TaGRAS27 | Triticum aestivum | abiotic stress tolerance | improves | The study revealed a significant increase in the expression... | RNA-seq and qRT-PCR |
| 38002936 | Yr72 | Triticum aestivum | stripe rust resistance | confers | The common wheat landraces AUS27506 and AUS27894 displayed... | bulked segregant analysis |



## Experimental Results

### Model Comparison for Gene-Trait Extraction

| Model | Total Entries | PMIDs with Relationships | Processing Time | Cost |
|-------|---------------|--------------------------|-----------------|------|
| Gemini 2.0 Flash | 374 | 110/200 | 33.50s | $0.0311 |
| GPT-4o-mini | 172 | 76/200 | 41.36s | $0.0212 |
| GPT-4.1-nano | 188 | 106/200 | 24.24s | $0.0119 |
| GPT-4.1-mini | 176 | 68/200 | 40.65s | $0.1732 |
| ChatGPT-4o-latest | 253 | 90/200 | 27.77s | $1.1419 |

### Initial Trait Mapping Results

| Model | Total Traits | Exact Matches | Synonym Matches | Semantic Matches |
|-------|--------------|---------------|-----------------|------------------|
| Gemini 2.0 Flash | 374 | 196 (52.41%) | 24 (6.42%) | 154 (41.18%) |
| GPT-4o-mini | 172 | 91 (52.91%) | 8 (4.65%) | 73 (42.44%) |
| GPT-4.1-nano | 188 | 99 (52.66%) | 6 (3.19%) | 83 (44.15%) |
| GPT-4.1-mini | 176 | 96 (54.55%) | 11 (6.25%) | 69 (39.20%) |
| ChatGPT-4o-latest | 253 | 158 (62.45%) | 8 (3.16%) | 87 (34.39%) |

### Unique Trait Name Statistics

| Model | Total Unique Traits | Exact | Synonym | Semantic |
|-------|---------------------|-------|---------|----------|
| Gemini 2.0 Flash | 140 | 44 (31.43%) | 10 (7.14%) | 86 (61.43%) |
| GPT-4o-mini | 83 | 28 (33.73%) | 5 (6.02%) | 50 (60.24%) |
| GPT-4.1-nano | 87 | 28 (32.18%) | 3 (3.45%) | 56 (64.37%) |
| GPT-4.1-mini | 78 | 26 (33.33%) | 6 (7.69%) | 46 (58.97%) |
| ChatGPT-4o-latest | 93 | 36 (38.71%) | 5 (5.38%) | 52 (55.91%) |

### Semantic Mapping Evaluation Results

| Model | Total Unique Semantic Pairs | Same Biological Concept | Different Biological Concept |
|-------|----------------------------|--------------------------|------------------------------|
| Gemini 2.0 Flash | 86 | 53 (61.6%) | 33 (38.4%) |
| GPT-4o-mini | 50 | 29 (58.0%) | 21 (42.0%) |
| GPT-4.1-nano | 56 | 36 (64.3%) | 20 (35.7%) |
| GPT-4.1-mini | 46 | 32 (69.6%) | 14 (30.4%) |
| ChatGPT-4o-latest | 52 | 32 (61.5%) | 20 (38.5%) |

### Overall Valid vs. Invalid Cases

| Model | Total Unique Traits | Valid Cases (Exact + Synonym + Valid Semantic) | Invalid Cases | Valid Rate |
|-------|---------------------|-----------------------------------------------|---------------|------------|
| Gemini 2.0 Flash | 140 | 107 | 33 | 76.43% |
| GPT-4o-mini | 83 | 62 | 21 | 74.70% |
| GPT-4.1-nano | 87 | 67 | 20 | 77.01% |
| GPT-4.1-mini | 78 | 64 | 14 | 82.05% |
| ChatGPT-4o-latest | 93 | 73 | 20 | 78.49% |

## Cost Analysis

| Model | Extraction Cost | Mapping Evaluation Cost | Total Cost per 200 PMIDs |
|-------|----------------|------------------------|--------------------------|
| GPT-4.1-nano | $0.0119 | N/A | $0.0119+ |
| GPT-4o-mini | $0.0212 | N/A | $0.0212+ |
| Gemini 2.0 Flash | $0.0311 | N/A | $0.0311+ |
| GPT-4.1-mini | $0.1732 | N/A | $0.1732+ |
| ChatGPT-4o-latest | $1.1419 | N/A | $1.1419+ |

*Note: Mapping evaluation costs use chatgpt-4o-latest for all models; exact costs for this step are not included in the table but should be factored into total project costs.*

## Performance Analysis

### Key Findings

1. **Extraction Volume**: 
   - Gemini 2.0 Flash extracted the most gene-trait relationships (374)
   - ChatGPT-4o-latest extracted the second most (253)
   - GPT-4o-mini extracted the fewest (172)

2. **Precision vs. Recall**:
   - GPT-4.1-mini achieves the highest valid rate (82.05%) with the lowest number of invalid cases (14)
   - Gemini 2.0 Flash provides the most valid mappings in absolute numbers (107) due to its higher extraction volume
   - ChatGPT-4o-latest has the second highest number of valid mappings (73) with a valid rate of 78.49%

3. **Cost Efficiency**:
   - GPT-4.1-nano offers the best cost-performance balance, with the lowest cost ($0.0119) while maintaining competitive performance (77.01% valid rate)
   - Gemini 2.0 Flash provides excellent value with high extraction volume at relatively low cost ($0.0311)
   - ChatGPT-4o-latest is significantly more expensive than other models

4. **Semantic Matching Quality**:
   - GPT-4.1-mini has the highest quality semantic matches (69.6% same biological concept)
   - GPT-4o-mini has the lowest quality semantic matches (58.0% same biological concept)

### Practical Implications

The choice between models depends on specific use case requirements:

- Choose **Gemini 2.0 Flash** when:
  - Maximizing extraction volume is the priority
  - Good balance between cost and performance is needed
  - High number of total valid mappings is desired

- Choose **GPT-4.1-mini** when:
  - Higher precision is crucial (highest valid rate at 82.05%)
  - The cost of incorrect mappings is significant
  - Budget constraints are moderate

- Choose **ChatGPT-4o-latest** when:
  - High-quality extractions are needed
  - Budget is less constrained
  - A good balance of precision and recall is desired

- Choose **GPT-4.1-nano** when:
  - Cost efficiency is the primary concern
  - A good balance of performance and economy is needed

These results demonstrate that different models offer distinct advantages in the trait ontology mapping process, allowing selection based on project-specific priorities.
