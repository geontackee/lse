# Locality Sensitive Hashing with Jaccard Similarity

This project implements Locality Sensitive Hashing (LSH) using Jaccard similarity and MinHash techniques to efficiently find similar sets in a large dataset.

## Overview

The project analyzes a dataset where each row contains a set of numbers and finds pairs of sets with Jaccard similarity above 0.85. Instead of computing similarity for all possible pairs (which would be computationally expensive), it uses LSH with MinHash to identify candidate pairs efficiently.

## Dataset

- **File**: `data/data.txt`
- **Format**: Each line contains space-separated numbers representing a set
- **Size**: Large dataset requiring efficient similarity computation

## Methodology

### 1. Jaccard Similarity
```python
def jaccard_sim(a, b):
    a = set(a)
    b = set(b)
    return len(a & b) / len(a | b)  # |A intersection B| / |A union B|
```

### 2. Performance Analysis
- **Sampling**: 5,000 random sets for performance estimation
- **Average computation time**: ~1.1 × 10⁻⁵ seconds per Jaccard similarity calculation
- **Total estimated time for all pairs**: ~5.5 × 10⁶ seconds (without optimization)

### 3. LSH Parameters Selection
The S-curve equation for LSH:
```python
def s_curve(s, r, b):
    return 1 - (1 - s**r)**b
```

**Selected Parameters**: r = 10, b = 10
- **Probability of collision for s = 0.85**: ~89%
- **Expected false positives**: ~15 candidates
- **Additional verification time**: ~0.00017 seconds

**Alternative considered**: r = 5, b = 20
- Higher collision probability (99.99%) but too many candidates (~8.7M)
- Would cause memory issues

### 4. MinHash Implementation
1. **Hash Function**: MurmurHash3 (mmh3 library)
2. **Signature Creation**: 100 buckets per set
3. **MinHash Process**: For each element in a set:
   - Compute hash value
   - Place in bucket (hash % 100)
   - Keep minimum hash value per bucket

### 5. LSH Banding
- **Bands**: 10 bands (b = 10)
- **Rows per band**: 10 rows (r = 10)
- **Bucket Assignment**: Sets with identical band signatures are placed in the same bucket
- **Candidate Generation**: All pairs within each bucket become candidate pairs

## Results

- **Total candidate pairs found**: 154,983
- **Real similar pairs (Jaccard > 0.85)**: 5 pairs identified
- **Sample real pairs**:
  - (215149, 274946)
  - (77683, 124849)
  - (2242, 49824)
  - (318540, 752337)
  - (167121, 479963)

## Key Benefits

1. **Efficiency**: Reduces computational complexity from O(n²) to O(n) for candidate generation
2. **Scalability**: Handles large datasets without memory overflow
3. **Accuracy**: High probability of finding truly similar pairs while minimizing false positives
4. **Speed**: Dramatically faster than naive pairwise comparison

## Dependencies

```bash
pip install mmh3 matplotlib
```

## Files

- `number_analysis_gtl256.ipynb`: Main analysis notebook
- `data/data.txt`: Input dataset (each line is a set of numbers)
- `README.md`: This documentation

## Algorithm Summary

1. **Data Loading**: Read sets from text file
2. **Performance Estimation**: Sample 5,000 sets to estimate computation time
3. **Parameter Selection**: Choose optimal r and b values based on S-curve analysis
4. **MinHash**: Create signatures for all sets using MurmurHash3
5. **LSH Banding**: Group sets into bands and buckets
6. **Candidate Generation**: Identify pairs within the same buckets
7. **Verification**: Compute exact Jaccard similarity for candidate pairs
8. **Results**: Return pairs with similarity > 0.85

## Technical Details

- **Hash Function**: MurmurHash3 for consistent, fast hashing
- **Signature Length**: 100 hash values per set
- **LSH Configuration**: 10 bands × 10 rows = 100 total hash functions
- **Similarity Threshold**: 0.85 Jaccard similarity
- **Memory Optimization**: Efficient bucket management to handle large datasets

This implementation demonstrates how LSH can make similarity search tractable for large-scale datasets while maintaining high accuracy in finding truly similar pairs.
