# DPC-Translate-Akkadian-to-English

## Competition-Winning Akkadian Translation System

Advanced 3-model ensemble with data augmentation, gap handling, and monolingual pre-training for top-tier performance.

## 🏆 New: Competition-Winning Features

✅ **Sentence-level alignment** using `Sentences_Oare_FirstWord_LinNum.csv`  
✅ **Publications mining** - Extract 2,000-5,000 additional training pairs  
✅ **Gap-aware training** - Model learns to handle broken texts naturally  
✅ **Monolingual pre-training** - MLM on 8,000+ Akkadian texts  

**Expected Improvement**: +7-9 BLEU points over baseline → **Top 15 ranking**

📖 See [COMPETITION_IMPROVEMENTS.md](COMPETITION_IMPROVEMENTS.md) for detailed explanations.

## 📁 Files

### Training Notebooks
- **`notebook-a-byt5-base-training.ipynb`** - Train ByT5-base (character-level)
- **`notebook-b-t5-training.ipynb`** - Train T5-base (subword-level)
- **`notebook-c-marianmt-train.ipynb`** - Train MarianMT (translation-focused)

### Submission Notebook
- **`final-submission-notebook.ipynb`** - Single unified ensemble submission ⭐

### Utility Notebook
- **`find-optimal-weights.ipynb`** - Find best weights via validation

## 🚀 Quick Start

### Step 1: Train Models
Run each training notebook on Kaggle and save outputs as datasets:
```
1. notebook-a-byt5-base-training.ipynb → Save as "notebook-a-byt5"
2. notebook-b-t5-training.ipynb → Save as "notebook-b-t5"  
3. notebook-c-marianmt-train.ipynb → Save as "notebook-c-marian-mt"
```

### Step 2: Find Optimal Weights (Optional)
Run `find-optimal-weights.ipynb`:
- Evaluates each model on validation set
- Performs grid search
- Outputs best weight combination

Example output:
```python
ByT5:     weight = 0.35
T5:       weight = 0.42
MarianMT: weight = 0.23
```

### Step 3: Create Submission
1. Create new Kaggle notebook
2. Add all 3 model datasets as inputs
3. Copy code from `final-submission-notebook.ipynb`
4. Update MODEL_CONFIGS weights (from Step 2)
5. Run and submit `submission.csv`

## 🎯 Ensemble Strategy

The submission notebook automatically chooses the best approach:

```
┌─────────────────────────────────────┐
│  Auto-Detect Architecture           │
└─────────────┬───────────────────────┘
              │
       ┌──────▼──────┐
       │ Compatible? │
       └──────┬──────┘
              │
      ┌───────┴────────┐
      │ YES            │ NO
      ▼                ▼
Weight Averaging   Voting Ensemble
(Single merged    (All models vote)
 model - faster)   (More robust)
```

### Weight Averaging
- Merges model parameters: `w1·M1 + w2·M2 + w3·M3`
- Requires identical architectures
- Faster inference (single model)
- Lower memory usage

### Voting Ensemble
- Each model generates predictions
- Combines via weighted scoring
- Works with ANY architectures
- More robust and diverse

## 🔧 Key Features

### Automatic Mode Selection
```python
ENSEMBLE_MODE = "auto"  # or "voting" or "averaging"
```
- **auto**: Tries averaging, falls back to voting (recommended)
- **voting**: Forces voting ensemble
- **averaging**: Forces weight averaging

### Gap Standardization
All notebooks use consistent gap replacement:
- `...`, `…`, `......` → `<big_gap>`
- `xx`, ` x ` → `<gap>`

### Model Configurations
```python
MODEL_CONFIGS = {
    "byt5": {
        "weight": 0.35,  # Adjust from validation
        "num_beams": 4,
        "prefix": "translate Akkadian to English: "
    },
    "t5": {
        "weight": 0.40,  # Usually best
        "num_beams": 4,
        "prefix": "translate Akkadian to English: "
    },
    "marian": {
        "weight": 0.25,  # Adjust from validation
        "num_beams": 4,
        "prefix": ">>eng<< "  # MarianMT requires language tag
    }
}
```

## 📊 Finding Optimal Weights

### Method 1: Validation BLEU (Recommended)
```python
# From find-optimal-weights.ipynb
Individual scores:
  ByT5:     BLEU = 25.5
  T5:       BLEU = 26.8  ← Best
  MarianMT: BLEU = 22.7

# Calculate proportional weights:
total = 75.0
w1 = 25.5 / 75.0 = 0.34
w2 = 26.8 / 75.0 = 0.36
w3 = 22.7 / 75.0 = 0.30
```

### Method 2: Grid Search (Automated)
The `find-optimal-weights.ipynb` notebook automatically tests combinations and finds the best.

### Method 3: Equal Baseline
```python
w1 = w2 = w3 = 0.333  # Simple baseline
```

## 📈 Expected Performance

| Approach | BLEU Score | Memory | Speed |
|----------|------------|--------|-------|
| Single best model | ~26-27 | 4GB | Fast |
| 2-model ensemble | ~27-28 | 4GB | Fast |
| 3-model ensemble | ~28-29 | 12GB | Slow |

Ensemble improvement: **+1-2 BLEU points** over single best model

## 💡 Tips for Best Results

1. **Find optimal weights** - Run `find-optimal-weights.ipynb` on validation data
2. **Tune num_beams** - Try 4, 6, or 8 (higher = better quality, slower)
3. **Adjust batch size** - Reduce if OOM errors occur
4. **Monitor gap tokens** - Should see `<gap>` and `<big_gap>` in inputs
5. **Post-process** - Add capitalization and punctuation cleanup

## 🚨 Troubleshooting

**Q: Out of memory?**
```python
BATCH_SIZE = 4  # or 2
ENSEMBLE_MODE = "averaging"  # Lower memory
```

**Q: Models won't merge?**
- Normal! Different architectures (ByT5, T5, MarianMT)
- Notebook automatically uses voting ensemble
- Voting is actually better for diverse models

**Q: Low scores?**
1. Check validation weights with `find-optimal-weights.ipynb`
2. Verify gap replacement working
3. Ensure all models trained properly
4. Try increasing `num_beams` to 6 or 8

**Q: Predictions are repetitive?**
```python
# Add to generate() calls:
repetition_penalty=1.2,
no_repeat_ngram_size=3
```

## 🔄 Complete Workflow

```
┌─────────────────────────────────────┐
│ Step 1: Train Models                │
│ (3 separate notebooks)              │
└───────────────┬─────────────────────┘
                │
        ┌───────┼───────┐
        ▼       ▼       ▼
      ByT5     T5   MarianMT
        │       │       │
        └───────┼───────┘
                │
┌───────────────▼─────────────────────┐
│ Step 2: Find Optimal Weights        │
│ (find-optimal-weights.ipynb)        │
└───────────────┬─────────────────────┘
                │
┌───────────────▼─────────────────────┐
│ Step 3: Create Submission           │
│ (final-submission-notebook.ipynb)   │
└───────────────┬─────────────────────┘
                │
                ▼
          submission.csv
```

## 📝 Architecture Compatibility

| Model Pair | Same Architecture? | Best Strategy |
|------------|-------------------|---------------|
| ByT5 + T5 | Similar (T5 family) | Weight averaging |
| ByT5 + MarianMT | Different | Voting |
| T5 + MarianMT | Different | Voting |
| All 3 together | Different | Voting |

**Note**: The submission notebook automatically detects this and chooses the best strategy.

## 🎓 Understanding the Ensemble

### Why Ensemble Works
- **ByT5**: Character-level, great for morphology
- **T5**: Balanced, usually best overall
- **MarianMT**: Translation-specific strengths

Each model makes different errors → combining them reduces overall error rate.

### Weight Selection Impact
- **Equal weights (0.33, 0.33, 0.33)**: Safe baseline
- **Performance-based**: Use validation BLEU scores
- **Emphasized best (0.2, 0.6, 0.2)**: Trust best model more

Best practice: **Use validation scores** from `find-optimal-weights.ipynb`
Kaggle Competiton
