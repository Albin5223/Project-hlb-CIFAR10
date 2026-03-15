## Ablation study

### se=False

- En changeant `se=True` à `False`, on obtient un train plus rapide:  78.9684 sec à 63.1623 sec
- Précision identique: 0.9444 à 0.9425

### short=True

- En changeant dans `residual1`, `short=False` à `True`, le train est plus rapide:  46.6951 sec
- Précision réduite: 0.9349
