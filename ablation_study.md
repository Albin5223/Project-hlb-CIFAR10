# Ablation Study — hlb-CIFAR10 · Round 2

**Config** : 12 epochs · batchsize 512 · base_depth 64 · 3 runs par ablation  
**Baseline** : 0.9420

---

## Résultats complets

| Ablation | ema_val_acc | Delta |
|---|---|---|
| fp32 | **0.9442** | +0.0022 |
| no_ema | 0.9440 | +0.0020 |
| label_smooth_05 | 0.9439 | +0.0019 |
| no_se | 0.9429 | +0.0009 |  
| no_label_smooth | 0.9427 | +0.0007 |  
| avg_pool | 0.9425 | +0.0005 |  
| whitening_k3 | 0.9423 | +0.0003 |  
| **baseline** | **0.9420** | — | référence |
| whitening_unfreeze | 0.9419 | -0.0001 |  
| cosine_scheduler | 0.9434 | +0.0014 |  
| cutout_8 | 0.9433 | +0.0013 |  
| flip_and_cutout_8 | 0.9434 | +0.0014 |  
| silu | 0.9403 | -0.0017 |  
| cutout_12 | 0.9406 | -0.0014 |  
| relu | 0.9389 | -0.0031 |  
| no_flip_cutout_8 | 0.9372 | -0.0048 |  
| no_flip_cutout_12 | 0.9368 | -0.0052 |  
| no_whitening | 0.9365 | -0.0055 |  
| no_flip | 0.9362 | -0.0058 |  
| **no_crop** | **0.9019** | **-0.0401** |

---

## Lecture des résultats

### Seuil de bruit estimé : ±0.003

Tous les deltas entre -0.003 et +0.003 sont non interprétables avec 3 runs. Cela représente la majorité des ablations de ce round.

### Les seuls effets réels

**`no_crop` (-0.0401)** est de loin le résultat le plus important de cette étude. Le random crop est le composant le plus critique du pipeline — bien plus que le flip. Sans lui, le réseau voit des images toujours cadrées de la même façon et ne généralise plus du tout. C'est une surprise par rapport au round 1 où le flip semblait dominer.

**`relu` (-0.0031)** confirme le round 1. GELU > ReLU de façon stable sur 6 runs au total.

**`no_flip` (-0.0058)** et **`no_whitening` (-0.0055)** confirment également le round 1.

**`no_flip_cutout_8/12`** : cutout ne compense pas l'absence de flip. Les deux ablations combinées (-0.0048, -0.0052) sont légèrement meilleures que `no_flip` seul (-0.0058), donc cutout aide un peu à la marge, mais l'effet est dans le bruit.

### Ce qui est définitivement neutre

`no_ema`, `label_smooth_05`, `no_se`, `avg_pool`, `whitening_unfreeze`, `whitening_k3`, `cosine_scheduler` : tous dans le bruit. Ces composants peuvent être simplifiés sans impact mesurable sur CIFAR-10 à cette échelle.

`fp32` reste légèrement au-dessus de la baseline (+0.0022) mais c'est probablement du bruit — ce résultat est incohérent avec la théorie et avec le round 1.

---

## Hiérarchie des composants

```
crop          indispensable   (-0.040 sans lui)
flip          important       (-0.006 sans lui)
whitening     utile           (-0.006 sans lui)
gelu          utile           (-0.003 vs relu)
─────────────────────────────────────────────
se            neutre
ema           neutre
label_smooth  neutre
pooling       neutre
scheduler     neutre
fp16          neutre (et plus rapide)
whitening_k   neutre
```

---