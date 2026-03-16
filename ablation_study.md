## Étude d'ablation

Nous évaluons l'impact de plusieurs composants de l'architecture et de l'entraînement en les retirant ou en les modifiant individuellement, tout en conservant le reste du modèle identique.  
La configuration de base (baseline) inclut le whitening en entrée, l'activation GELU, les blocs SE, un max pooling global, l'augmentation par retournement horizontal (flip) et l'entraînement en précision mixte (FP16).

### Résultats

| Configuration | Accuracy |
|---|---|
| Baseline | **0.9420** |
| Sans SE | 0.9436 |
| ReLU (au lieu de GELU) | 0.9379 |
| Sans whitening | 0.9341 |
| Avg Pool (au lieu de Max Pool) | 0.9420 |
| Sans flip (augmentation) | 0.9338 |
| FP32 (au lieu de FP16) | 0.9406 |

### Observations

- Le **whitening** améliore les performances : sa suppression entraîne une baisse d’environ **0.8%** d’accuracy.
- L’**augmentation de données (flip horizontal)** est importante pour la généralisation : sa suppression réduit l’accuracy d’environ **0.8%**.
- L’activation **GELU** donne de meilleurs résultats que **ReLU** dans ce modèle.
- Les **blocs SE** n’apportent pas d’amélioration notable dans cette architecture.
- Le type de **pooling global** (max ou moyenne) a un impact négligeable.
- L'entraînement en **précision mixte (FP16)** obtient des performances similaires au **FP32**, tout en étant plus efficace en calcul.

### Conclusion

Les éléments ayant le plus d’impact sur les performances sont **l’augmentation de données** et le **whitening en entrée**.  
Les choix architecturaux comme les blocs SE ou le type de pooling influencent peu les performances dans cette configuration.
