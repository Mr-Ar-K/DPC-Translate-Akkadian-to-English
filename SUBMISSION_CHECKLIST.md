# Competition Submission Checklist

## 📋 Pre-Training Checklist

### Data Files Required
- [ ] `train.csv` in input directory
- [ ] `published_texts.csv` available (for monolingual pre-training)
- [ ] `publications.csv` available (for data mining)
- [ ] `Sentences_Oare_FirstWord_LinNum.csv` (optional, improves alignment)

### Configuration Check
- [ ] `ENABLE_MONO_PRETRAIN=True` (in training notebooks)
- [ ] Gap handling enabled (`keep_gaps=True`)
- [ ] Publications mining will run automatically if files exist
- [ ] Batch size appropriate for your GPU (8 for T4, 4 for P100)

---

## 🎯 Training Workflow

### Model 1: ByT5 (Character-level)
- [ ] Run `notebook-a-byt5-base-training.ipynb`
- [ ] Wait for data augmentation messages:
  - `✓ Mined X additional training pairs from publications`
  - `✓ Monolingual pre-training complete`
- [ ] Training completes (~2-3 hours on T4)
- [ ] Save output as Kaggle dataset: "notebook-a-byt5"
- [ ] **Expected Training Size**: 3,000-5,000 pairs (vs 1,500 baseline)

### Model 2: T5 (Subword-level)
- [ ] Run `notebook-b-t5-training.ipynb`
- [ ] Same data augmentation applies
- [ ] Training completes (~1-2 hours on T4)
- [ ] Save output as: "notebook-b-t5"

### Model 3: MarianMT (Translation-focused)
- [ ] Run `notebook-c-marianmt-train.ipynb`
- [ ] Training completes (~2-3 hours on T4)
- [ ] Save output as: "notebook-c-marian-mt"

---

## 🔍 Find Optimal Weights

- [ ] Run `find-optimal-weights.ipynb`
- [ ] Add all 3 trained models as inputs
- [ ] Wait for grid search (~30-60 minutes)
- [ ] Copy optimal weights from output:
  ```python
  ByT5:    weight = X.XX
  T5:      weight = X.XX
  MarianMT: weight = X.XX
  ```
- [ ] Save these for submission notebook

---

## 📤 Create Submission

### Setup
- [ ] Create new Kaggle notebook
- [ ] Add competition data as input
- [ ] Add all 3 model datasets as inputs:
  - `notebook-a-byt5`
  - `notebook-b-t5`
  - `notebook-c-marian-mt`

### Configuration
- [ ] Copy code from `final-submission-notebook.ipynb`
- [ ] Update MODEL_CONFIGS with optimal weights
- [ ] Set `ENSEMBLE_MODE = "auto"` (or "voting" for safety)
- [ ] Verify model paths match your dataset names

### Run and Verify
- [ ] Run submission notebook
- [ ] Check console output:
  - `✓ Ensemble model ready`
  - `✓ Generated X predictions`
  - `✓ Submission saved to submission.csv`
- [ ] Inspect sample predictions (should handle gaps properly)
- [ ] Check statistics:
  - Empty/broken: Should be <5% of total
  - Average length: Should be 15-30 words
  - Gap tokens visible in inputs

### Submit
- [ ] Download `submission.csv`
- [ ] Submit to competition
- [ ] Note your score for comparison

---

## ✅ Quality Checks

### Training Data Quality
```python
# Should see these in training output:
✓ Loading sentence alignment data...
✓ Aligned using sentence map: 3000+ examples
✓ Mined 1000+ additional training pairs from publications
✓ Total training examples after augmentation: 4000+
✓ Monolingual pre-training complete
```

### Gap Handling Verification
```python
# Training data should include:
Input:  "a-na <gap> lu-pu-ut"
Output: "to ... tablet"

# NOT:
Input:  "a-na lu-pu-ut"  # gaps removed
Output: "to tablet"       # model won't handle gaps
```

### Ensemble Verification
```python
# Submission notebook should show:
✓ Using voting ensemble (models incompatible for averaging)
✓ Generated predictions from 3 models
✓ Combined predictions via weighted voting
```

---

## 🎯 Expected Performance

| Metric | Baseline | With Improvements | Target |
|--------|----------|-------------------|--------|
| Training Size | 1,500 | 4,000-6,000 | 5,000+ |
| Validation BLEU | 24-26 | 30-32 | 32+ |
| Test BLEU | 23-25 | 29-31 | 31+ |
| Competition Rank | 50-100 | 10-20 | Top 15 |

---

## 🚨 Troubleshooting

### "No additional pairs extracted from publications"
- Publications mining may not find matches
- Still beneficial to run (monolingual pre-training still helps)
- Consider manual extraction for specific texts

### "Out of memory" during training
- Reduce `per_device_train_batch_size` from 8 to 4
- Disable monolingual pre-training temporarily
- Limit publications mining to 1,000 texts

### "Models won't merge" in submission
- Normal! ByT5/T5/MarianMT have different architectures
- Notebook automatically uses voting ensemble
- This is actually better for performance

### Low scores despite improvements
- Verify gap tokens present in training: `<gap>`, `<big_gap>`
- Check validation scores with `find-optimal-weights.ipynb`
- Ensure publications mining found examples
- Try increasing `num_beams` to 6 or 8

---

## 📊 Score Tracking

| Date | Model | Augmentation | BLEU | Rank | Notes |
|------|-------|--------------|------|------|-------|
| ___ | ByT5 alone | None | __ | __ | Baseline |
| ___ | ByT5 | +Publications | __ | __ | Data mining |
| ___ | ByT5 | +MLM | __ | __ | Monolingual |
| ___ | 3-Model Ensemble | All | __ | __ | Final |

---

## 🎓 Key Success Factors

1. **Data Augmentation** - Most important! Mine publications.csv
2. **Gap Training** - Critical for test set performance
3. **Monolingual Pre-training** - Teaches Akkadian grammar
4. **Proper Ensemble** - Use optimal weights from validation
5. **Quality Checks** - Verify at each step

---

## 📞 Quick Reference

**Training Time**: 6-8 hours total (3 models x 2-3 hours each)  
**Finding Weights**: 30-60 minutes  
**Submission**: 5-10 minutes  

**Total Pipeline**: 7-9 hours for complete competition entry  
**Expected Rank**: Top 10-20 with all improvements  
