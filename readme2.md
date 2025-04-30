# Trait Ontology Mapping and Evaluation System

A comprehensive system for mapping trait names and trait IDs of extracted gene-trait relationships and evaluating these mappings using OpenAI's LLM model.

## Features

- **Ontology Parsing**: Parse OBO format files into structured data.
- **Trait Evaluation**: Evaluate trait names against ontology terms using a three-tier matching approach:
  1. Exact name matching
  2. Synonym matching
  3. Semantic similarity matching using semantic embeddings
- **Multiple Embedding Models**: Support for both BERT-based and OpenAI embeddings.
- **Comprehensive Evaluation**: Detailed metrics on match quality and confidence.
- **Evaluation of Semantically Matched Mappings**: Generate responses from LLM to evaluate matched trait names if their confidence score is above 50 and the matched type is semantic.
- **Analysis of Mapping Evaluation Results**: Evaluate matched cases using three metrics and assess the performance of different gene-trait relation tables.

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

## Embedding Process

The embedding process follows a systematic workflow to match trait names from gene-trait relations with standardized trait ontology terms:

### Workflow

1. **Ontology Preparation**
   * Parse the Trait Ontology OBO file
   * Extract all trait terms, IDs, and synonyms
   * Embed all ontology terms and their synonyms

2. **Input Processing**
   * Read the gene-trait relation CSV file
   * Extract trait names for matching

3. **Three-Tier Matching Approach**
   * **Tier 1:** Check for exact name matches between trait names and ontology terms
   * **Tier 2:** Check for synonym matches if exact match fails
   * **Tier 3:** Apply semantic similarity matching using embeddings if previous tiers fail
   
4. **Semantic Matching Process**
   * Embed each trait name from the input table
   * Compare embeddings against all ontology term embeddings
   * Identify the top 5 most similar terms based on cosine similarity
   * Select the match with the highest confidence score

5. **Output Generation**
   * Augment the original table with matching results
   * Add the following columns:
     * `exact_match` (boolean) - Whether an exact match was found
     * `synonym_match` (boolean) - Whether a synonym match was found
     * `semantic_matches` (list) - Top 5 semantically similar matches with scores
     * `match_type` (string) - Type of match (exact, synonym, semantic, or none)
     * `matched_id` (string) - ID of the best matching trait term
     * `matched_name` (string) - Name of the best matching trait term
     * `confidence` (float) - Confidence score of the match (1.0 for exact/synonym matches)

### Example Output

**Semantic Matches Example:**
```python
[("TO:0000975", "grain width", 0.88), 
 ("TO:0002625", "fruit size", 0.84), 
 ("TO:1000011", "fruit position", 0.83), 
 ("TO:0000734", "grain length", 0.82), 
 ("TO:0001108", "grain volume", 0.8)]
```

**Example Record After Embedding:**

| pmid | gene | species | trait_name | exact_match | synonym_match | semantic_matches | match_type | matched_id | matched_name | confidence |
|------|------|---------|------------|-------------|---------------|------------------|------------|------------|--------------|------------|
| 38671351 | TaUBC25 | Triticum aestivum | kernel thickness | FALSE | FALSE | [('TO:0000975', 'grain width', 0.88), ('TO:0002625', 'fruit size', 0.84), ('TO:1000011', 'fruit position', 0.83), ('TO:0000734', 'grain length', 0.82), ('TO:0001108', 'grain volume', 0.8)] | semantic | TO:0000975 | grain width | 0.88 |
| 38063491 | TaBBE64 | Triticum aestivum | wheat stripe rust disease resistance | TRUE | FALSE | [] | exact | TO:0020055 | wheat stripe rust disease resistance | 1 |

## LLM-Based Mapping Evaluation

To evaluate mapping results, the system uses OpenAI's `chatgpt-4o-latest` model. The evaluation is performed in bulk, with 30 pairs per request (default value), based on analysis showing that larger batches degrade response quality.

### Evaluation Prompts

Prompts are created for LLM-based mapping evaluation and stored in the `mapping_eval_prompts` folder inside the `evaluation` folder.

The LLM receives a trait name from the gene-trait relation extraction table and its matched name as a pair. The model outputs the evaluation with two additional fields added to the embedding results:
- `same_biological_concept` 
- `reasoning`

### Example LLM Evaluation Output

The final result of LLM mapping evaluation adds two additional columns:

| Column | Description |
|--------|-------------|
| same_biological_concept | Whether the original trait and matched trait represent the same biological concept (Yes/No) |
| reasoning | Brief explanation of the reasoning behind the assessment |

**Example of Final LLM Evaluation Output:**

| pmid | trait_name | matched_id | matched_name | confidence | same_biological_concept | reasoning |
|------|------------|------------|--------------|------------|-------------------------|-----------|
| 39256758 | Fusarium head blight resistance | TO:0000663 | wheat fusarium head blight resistance | 0.92 | Yes | 'Fusarium head blight resistance' and 'wheat fusarium head blight resistance' describe the same trait, with the latter specifying the crop species. |
| 38671351 | kernel thickness | TO:0000975 | grain width | 0.88 | No | The original trait refers to kernel thickness, while the matched trait refers to grain width, which are related but distinct measurements. |

## Evaluation Results

### Initial Semantic Matching Results

| Model Version | Total | Exact | Synonym | Semantic | No Match |
|---------------|-------|-------|---------|----------|----------|
| GPT-4o Bulk-5 (C) | 124 | 32 | 4 | 48 | 40 |
| GPT-4o-Mini Bulk-5 (A) | 122 | 26 | 10 | 44 | 42 |
| GPT Mini Single (B) | 180 | 34 | 9 | 61 | 76 |

### After LLM Evaluation of Semantic Matches (Ranked by Match Rate)

| Model Version | Total | Valid Matches | Invalid Matches | Match Rate |
|---------------|-------|---------------|-----------------|------------|
| GPT-4o Bulk-5 (C) | 124 | 75 | 49 | 60.00 |
| GPT-4o-Mini Bulk-5 (A) | 122 | 71 | 51 | 58.20 |
| GPT Mini Single (B) | 180 | 93 | 87 | 51.67 |

### Breakdown of Valid Matches

| Model Version | Exact | Synonym | LLM-Validated Semantic | Total Valid |
|---------------|-------|---------|------------------------|-------------|
| GPT Mini Single (B) | 34 | 9 | 50 | 93 |
| GPT-4o Bulk-5 (C) | 32 | 4 | 39 | 75 |
| GPT-4o-Mini Bulk-5 (A) | 26 | 10 | 35 | 71 |

### Unique Trait Name Matching Performance (Ranked by Valid Rate)

| Model Version | Total Unique Traits | Exact | Synonym | Semantic | Semantic Yes | Semantic No | Valid Cases | Invalid Cases | Valid Rate |
|---------------|---------------------|-------|---------|----------|--------------|-------------|-------------|---------------|------------|
| GPT-4o Mini Bulk-5 | 71 | 18 | 6 | 47 | 16 | 31 | 40 | 31 | 56.34 |
| GPT-4o Mini Single | 96 | 18 | 6 | 72 | 24 | 48 | 48 | 48 | 50.00 |
| GPT-4o Bulk-5 | 64 | 17 | 4 | 43 | 12 | 31 | 33 | 31 | 51.56 |

*Note: Valid Cases = Exact + Synonym + Semantic Yes; Invalid Cases = Semantic No; Valid Rate = (Valid Cases / Total Unique Traits) × 100*

*Note: LLM-Validated Semantic matches were validated using `chatgpt-4o-latest` against `text-embedding-3-large` model outputs.*

## Performance Analysis

### Precision and Recall Trade-offs

In ontology mapping tasks, different use cases may prioritize different aspects of performance:

- **Precision**: Focuses on minimizing false positives; crucial when the cost of incorrect matches is high
- **Recall**: Prioritizes finding as many valid matches as possible; important when comprehensive coverage is essential

#### Precision vs. Total Valid Matches

| Model | Valid Cases | Invalid Cases | Total Unique Traits | Precision | Total Valid Matches |
|-------|-------------|---------------|---------------------|-----------|---------------------|
| GPT-4o Mini Bulk-5 | 40 | 31 | 71 | 56.34% | 40 |
| GPT-4o Bulk-5 | 33 | 31 | 64 | 51.56% | 33 |
| GPT-4o Mini Single | 48 | 48 | 96 | 50.00% | 48 |

This analysis reveals important trade-offs:

- **GPT-4o Mini Single** produces the highest absolute number of valid matches (48), making it the preferred choice when maximizing the total number of valid trait mappings is the priority. However, it also generates the most invalid matches (48), resulting in a 50% precision rate.

- **GPT-4o Mini Bulk-5** achieves the highest precision at 56.34%, meaning it has the best ratio of valid to invalid matches. This model identified 40 valid cases out of 71 total unique traits with only 31 invalid cases, making it optimal when reducing false positives is more important than maximizing the total number of matches.

- **GPT-4o Bulk-5** falls between the other models in terms of performance, with moderate precision (51.56%) but the lowest total number of valid matches (33).

#### Practical Implications

The choice between models depends on specific use case requirements:

- Choose **GPT-4o Mini Single** when:
  - The goal is to capture as many valid trait mappings as possible
  - Additional validation steps can be implemented to filter out false positives
  - Comprehensive coverage is more important than precision

- Choose **GPT-4o Mini Bulk-5** when:
  - Higher confidence in each match is crucial
  - The cost of incorrect mappings is significant
  - A better balance between precision and total matches is desired

These results demonstrate that optimizing for either precision or total valid matches requires different approaches in the trait ontology mapping process.
