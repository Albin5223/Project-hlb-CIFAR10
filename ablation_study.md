# Ablation Study — hlb-CIFAR10

**Config** : 12 epochs · batchsize 512 · base_depth 64 · 3 runs par ablation  
**Baseline** : 0.9420

---

## Résultats complets (Accuracy)

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

### Les seuls effets réels

**`no_crop` (-0.0401)** : c'est le point le plus critique de cette analyse en accuracy. Sans le crop, le réseau voit des images toujours cadrées de la même façon et ne généralise plus du tout.

**`relu` (-0.0031)** : effet de "dying ReLU", ReLU va passer des gradients nuls lorsque les valeurs sont négatives, donc les neurones n'apprennent plus. Contrairement à GELU qui est une fonction lisse et continue, dérivable partout.  

**`no_flip` (-0.0058)** et **`no_whitening` (-0.0055)** : sans le flip, les données ne sont plus diversifiées. Ce qu'on avait auparavant était avec une proba 1/2 que les images le soient. En regardant le `train_acc`, on peut voir qu'à l'epoch 1, il est carrément de `1.00`, on est en plein dans l'overfitting: le réseau a dû voir des patterns selon l'orientation des images.

**`no_flip_cutout_8/12`** : cutout ne compense pas l'absence de flip. Les deux ablations combinées (-0.0048, -0.0052) sont légèrement meilleures que `no_flip` seul (-0.0058), donc cutout aide un peu à la marge, mais l'effet est dans le bruit.

### Ce qui est définitivement neutre

`no_ema`, `label_smooth_05`, `no_se`, `avg_pool`, `whitening_unfreeze`, `whitening_k3`, `cosine_scheduler` : tous dans le bruit. Ces composants peuvent être simplifiés sans impact mesurable sur CIFAR-10 à cette échelle.

`fp32` reste légèrement au-dessus de la baseline (+0.0022) mais c'est probablement du bruit.

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

## Résultats complets (Temps)

| Ablation | Run 1 | Run 2 | Run 3 | Moyenne | Delta (s) | Delta (%) | ema_val_acc |
|---|---|---|---|---|---|---|---|
| **no_crop** | 114.37 | 115.17 | 115.34 | **114.96** | **+40.13** | **+53.6%** | 0.9019 |
| no_whitening | 75.72 | 75.74 | 75.60 | 75.69 | +0.86 | +1.2% | 0.9365 |
| avg_pool | 74.84 | 75.02 | 75.03 | 74.96 | +0.14 | +0.2% | 0.9425 |
| **baseline** | 75.36 | 74.72 | 74.39 | **74.82** | — | — | 0.9420 |
| cutout_8 | 74.52 | 74.54 | 74.60 | 74.56 | -0.27 | -0.4% | 0.9433 |
| cutout_12 | 74.54 | 74.58 | 74.24 | 74.45 | -0.37 | -0.5% | 0.9406 |
| whitening_k3 | 74.50 | 74.36 | 74.43 | 74.43 | -0.39 | -0.5% | 0.9423 |
| fp32 | 74.41 | 74.52 | 74.40 | 74.44 | -0.38 | -0.5% | 0.9442 |
| no_label_smooth | 74.81 | 74.34 | 74.11 | 74.42 | -0.40 | -0.5% | 0.9427 |
| label_smooth_05 | 74.12 | 74.57 | 74.54 | 74.41 | -0.42 | -0.6% | 0.9439 |
| cosine_scheduler | 74.28 | 74.33 | 74.50 | 74.37 | -0.46 | -0.6% | 0.9434 |
| no_flip | 74.29 | 74.53 | 74.26 | 74.36 | -0.46 | -0.6% | 0.9362 |
| no_flip_cutout_8 | 74.37 | 74.25 | 74.46 | 74.36 | -0.47 | -0.6% | 0.9372 |
| no_flip_cutout_12 | 74.33 | 74.33 | 74.30 | 74.32 | -0.50 | -0.7% | 0.9368 |
| no_ema | 74.02 | 74.55 | 74.26 | 74.28 | -0.55 | -0.7% | 0.9440 |
| whitening_unfreeze | 74.29 | 74.68 | 73.83 | 74.27 | -0.56 | -0.7% | 0.9419 |
| no_se | 74.34 | 74.10 | 74.26 | 74.23 | -0.59 | -0.8% | 0.9429 |
| flip_and_cutout_8 | 74.10 | 74.05 | 74.19 | 74.11 | -0.71 | -0.9% | 0.9434 |
| relu | 71.38 | 71.23 | 71.36 | **71.32** | **-3.50** | **-4.7%** | 0.9389 |
| silu | 71.21 | 70.98 | 71.46 | **71.22** | **-3.61** | **-4.8%** | 0.9403 |

---

## Conclusion des résultats en temps

- `no_crop` en plus de désavantager l'accuracy sanctionne aussi la performance en temps: les images ne sont pas croppés à chaque epoch, et donc nécessite plus de temps pour converger.

- `relu` et `silu` sont plus rapides que `gelu`: c'est normal car on calcule un simple `max` et `sigmoid` au lieu de la CDF gaussienne.

- Le reste n'a pas l'air de grandement impacter les résultats: on est plus ou moins au même niveau de la baseline.