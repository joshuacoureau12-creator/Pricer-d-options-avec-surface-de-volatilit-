# Pricer d'Options Européennes & Américaines avec Surface de Volatilité

> Projet de finance quantitative implémentant plusieurs méthodes de pricing d'options et une construction en temps réel de la surface de volatilité implicite, appliquées à des données de marché live via `yfinance`.

---

## Présentation

Ce projet propose un moteur de pricing modulaire en Python pour des **options vanilles européennes et américaines**, combinant trois méthodes numériques standards avec un pipeline complet d'**extraction de volatilité implicite (VI) et de visualisation 3D**. L'actif sous-jacent est Apple Inc. (AAPL), avec des données de marché récupérées en direct depuis Yahoo Finance.

Le notebook est structuré en quatre modules indépendants et réutilisables :

| Module | Description |
|--------|-------------|
| **Black-Scholes** | Pricing en forme fermée + Grecques (Δ, Γ, ν) |
| **Monte Carlo** | Simulation GBM avec réduction de variance par variables antithétiques |
| **Arbre Binomial CRR** | Modèle en treillis avec exercice anticipé (options américaines) |
| **Volatilité Implicite** | Inversion numérique de Brent + interpolation et surface 3D |

---

## Méthodes de Pricing

### Module 1 — Black-Scholes (Forme fermée)

Implémente la formule analytique de Black-Scholes pour les **calls et puts européens**, ainsi que trois Grecques clés :

- **Delta (Δ)** — sensibilité du prix de l'option au sous-jacent
- **Gamma (Γ)** — convexité du delta
- **Vega (ν)** — sensibilité à un mouvement de 1% de la volatilité

```python
black_scholes(S=150, K=155, T=0.5, r=0.05, sigma=0.20, option_type='call')
```

### Module 2 — Simulation Monte Carlo

Simule le sous-jacent via un **Mouvement Brownien Géométrique** sous la mesure risque-neutre :

$$dS = rS\,dt + \sigma S\,dW$$

Utilise des **variables antithétiques** pour réduire la variance de moitié sans augmenter le nombre de simulations. Retourne le prix estimé ainsi qu'un **intervalle de confiance** via l'erreur standard.

```python
price, std_err = monte_carlo_pricer(S=150, K=155, T=0.5, r=0.05, sigma=0.20, n_sims=100_000)
```

### Module 3 — Arbre Binomial CRR

Treillis binomial de Cox-Ross-Rubinstein avec **induction rétrograde**, supportant :

- **Options européennes** — sans exercice anticipé
- **Options américaines** — vérification de l'exercice anticipé à chaque nœud par comparaison avec la valeur intrinsèque

```python
crr_binomial(S=150, K=155, T=0.5, r=0.05, sigma=0.20, N=200, option_type='put', american=True)
```

### Module 4 — Volatilité Implicite & Surface

Inverse numériquement la formule de Black-Scholes via la **méthode de Brent** pour extraire la volatilité implicite à partir des prix de marché observés. La surface de VI est ensuite construite par **interpolation cubique** sur une grille (Strike × Maturité) et rendue sous forme de graphique 3D.

Pipeline de données :
1. Récupération du carnet d'options AAPL en direct (5 échéances) via `yfinance`
2. Calcul de la volatilité implicite pour chaque paire (K, T)
3. Interpolation sur une grille régulière avec `scipy.interpolate.griddata`
4. Visualisation avec une surface 3D `matplotlib` (colormap `viridis`)

---

## Installation

```bash
pip install numpy scipy matplotlib yfinance pandas
```

**Version Python requise :** 3.10+

---

## Utilisation

Ouvrir et exécuter le notebook cellule par cellule dans l'ordre. Chaque module est autonome et peut être testé indépendamment.

```bash
jupyter notebook "Pricer_d_options_européennes_et_américaines_avec_surface_de_volatilité.ipynb"
```

**Paramètres principaux** (modifiables en tête de chaque module) :

| Paramètre | Symbole | Description |
|-----------|---------|-------------|
| `S` | Prix spot | Prix courant du sous-jacent (récupéré en direct) |
| `K` | Strike | Prix d'exercice de l'option |
| `T` | Maturité | Temps avant expiration, en années |
| `r` | Taux sans risque | Taux continu composé (défaut : 5%) |
| `sigma` | Volatilité | Vol implicite ou historique annualisée |
| `N` | Pas | Nombre de pas de l'arbre binomial (défaut : 200) |
| `n_sims` | Simulations | Nombre de trajectoires Monte Carlo (défaut : 100 000) |

---

## Structure du projet

```
.
├── Pricer_d_options_européennes_et_américaines_avec_surface_de_volatilité.ipynb
│   ├── Module 1 — Black-Scholes & Grecques
│   ├── Module 2 — Monte Carlo (GBM + variables antithétiques)
│   ├── Module 3 — Arbre Binomial CRR (Européen & Américain)
│   └── Module 4 — Surface de Volatilité Implicite (données AAPL live)
└── README.md
```

---

## Choix techniques

- **Variables antithétiques** en Monte Carlo : réduction de variance d'environ 50% sans coût de calcul supplémentaire, en associant chaque tirage `Z` à son symétrique `-Z`.
- **Méthode de Brent** pour l'inversion de la VI : recherche de racine par encadrement dans `[1e-6, 10.0]`, avec retour `NaN` pour les prix de marché violant les bornes d'arbitrage.
- **Interpolation cubique** pour la surface de VI : rendu plus lisse que l'interpolation linéaire, évitant les artefacts anguleux typiques des données brutes de carnets d'options.
- **Accesseur `.values`** plutôt que l'attribut `.T` sur les DataFrames : évite l'ambiguïté classique de pandas entre la propriété de transposition et une colonne nommée `T`.

---

## Résultat illustratif

La surface de volatilité implicite pour AAPL met en évidence le **smile de volatilité** caractéristique selon les strikes, ainsi que l'effet de **structure par terme** selon les maturités — cohérent avec les dynamiques de marché observées (VI plus élevée pour les options OTM, structure décroissante en régime normal).

---

## Compétences illustrées

- Pricing de produits dérivés vanille (Black-Scholes, Monte Carlo, Arbres Binomiaux)
- Méthodes numériques : recherche de racine (Brent), interpolation (scipy)
- Techniques de réduction de variance en simulation
- Traitement de données financières en temps réel (`yfinance`, `pandas`)
- Visualisation 3D de données (`matplotlib`, `mpl_toolkits`)
- Code Python modulaire et documenté

---

## Auteur

**Joshua Coureau**  
Master de Mathématiques Fondamentales — Université de Bordeaux  
[GitHub](https://github.com/joshuacoureau12-creator)

---

## Licence

Ce projet est réalisé à des fins pédagogiques et de constitution de portfolio.
