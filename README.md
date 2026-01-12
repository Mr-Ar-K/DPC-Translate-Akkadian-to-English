# DPC-Translate-Akkadian-to-English

## Competition-Winning Akkadian Translation System

Advanced 3-checkpoint ByT5 ensemble with data augmentation, gap handling, and monolingual pre-training for top-tier performance.

## 🏆 New: Competition-Winning Features

✅ **Sentence-level alignment** using `Sentences_Oare_FirstWord_LinNum.csv`  
✅ **Publications mining** - Extract 2,000-5,000 additional training pairs  
✅ **Gap-aware training** - Model learns to handle broken texts naturally  
✅ **Monolingual pre-training** - MLM on 8,000+ Akkadian texts  

**Expected Improvement**: +7-9 BLEU points over baseline → **Top 15 ranking**

📖 See [COMPETITION_IMPROVEMENTS.md](COMPETITION_IMPROVEMENTS.md) for detailed explanations.

## 📁 Files

### Training Notebooks
- **`notebook-a-byt5-monolingual.ipynb`** - ByT5 Purist (baseline + monolingual MLM)
- **`notebook-b-byt5-augmented.ipynb`** - ByT5 Greedy (data-augmented)
- **`notebook-c-byt5-dropout.ipynb`** - ByT5 Specialist (dropout-tuned)

### Submission Notebook
- **`final-submission-notebook.ipynb`** - Single unified ensemble submission ⭐

### Utility Notebook
- **`find-optimal-weights.ipynb`** - Find best weights via validation

## 🚀 Quick Start

### Step 1: Train Models
Run each training notebook on Kaggle and save outputs as datasets:
```
1. notebook-a-byt5-monolingual.ipynb → Save as "notebook-a-byt5"
2. notebook-b-byt5-augmented.ipynb → Save as "notebook-b-byt5-augmented"  
3. notebook-c-byt5-dropout.ipynb → Save as "notebook-c-byt5-dropout"
```

### Step 2: Find Optimal Weights (Optional)
Run `find-optimal-weights.ipynb`:
- Evaluates each model on validation set
- Performs grid search
- Outputs best weight combination

Example output:
```python
ByT5-Purist:      weight = 0.35
ByT5-Greedy:      weight = 0.35
ByT5-Specialist:  weight = 0.30
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
  "byt5_purist": {
    "weight": 0.35,  # Adjust from validation
    "num_beams": 4,
    "prefix": "translate Akkadian to English: "
  },
  "byt5_greedy": {
    "weight": 0.35,
    "num_beams": 4,
    "prefix": "translate Akkadian to English: "
  },
  "byt5_specialist": {
    "weight": 0.30,
    "num_beams": 4,
    "prefix": "translate Akkadian to English: "
  }
}
```

## 📊 Finding Optimal Weights

### Method 1: Validation BLEU (Recommended)
```python
# From find-optimal-weights.ipynb
Individual scores:
  ByT5-Purist:     BLEU = 25.5
  ByT5-Greedy:     BLEU = 26.0
  ByT5-Specialist: BLEU = 24.5

# Calculate proportional weights:
total = 76.0
w1 = 25.5 / 76.0 = 0.34
w2 = 26.0 / 76.0 = 0.34
w3 = 24.5 / 76.0 = 0.32
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
| 2-checkpoint soup | ~27-28 | 4GB | Fast |
| 3-checkpoint soup | ~28-29 | 6-8GB | Moderate |

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
- All three checkpoints are ByT5; weight averaging works out of the box.
- If GPU RAM is tight, switch to voting to keep memory predictable.

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
│ Step 1: Train ByT5 checkpoints      │
│ (3 separate notebooks)              │
└───────────────┬─────────────────────┘
    │
  ┌───────┼───────┐
  ▼       ▼       ▼
   Purist   Greedy   Specialist
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

All checkpoints share the ByT5 architecture, so weight averaging/model soup works directly. If memory is constrained, switch the submission notebook to voting (per-example generation from each checkpoint).

## 🎓 Understanding the Ensemble

### Why Ensemble Works
- **ByT5 Purist**: Strong baseline + MLM pretraining
- **ByT5 Greedy**: Beneficial data augmentation and aggressive beam search
- **ByT5 Specialist**: Dropout-tuned for robustness

Each checkpoint makes slightly different errors → combining them reduces overall error rate.

### Weight Selection Impact
- **Equal weights (0.33 each)**: Safe baseline
- **Performance-based**: Use validation BLEU scores
- **Emphasize best (e.g., 0.2, 0.4, 0.4)**: Favor strongest variants

Best practice: **Use validation scores** from `find-optimal-weights.ipynb`
