# adversarial

Plugin Claude Code — workflow de résolution de problème par deux agents rivaux + un juge.

Deux agents en parallèle (Alpha pragmatique, Beta sceptique) attaquent le même problème dans des worktrees git isolés. Un troisième agent fait office de juge sur les solutions anonymisées et rend un verdict : `ALPHA`, `BETA`, ou `MERGE`.

## Installation

Dans Claude Code :

```
/plugin install Virgiledc/adversarial@github
```

## Usage

Le skill se déclenche automatiquement quand tu dis :

- `/adversarial`
- `mode adversarial`
- `dual solve`
- `deux agents`
- ou implicitement sur toute tâche complexe où la qualité prime sur la vitesse

## Workflow

1. **Cadrage** — objectif, contraintes, critère de succès présentés en 3-5 lignes, validés par l'utilisateur.
2. **Dual solve** — Alpha (pragmatique, code direct) et Beta (sceptique, challenge des hypothèses, cas limites) lancés en parallèle dans des worktrees git isolés.
3. **Juge** — un 3ème agent compare les solutions A/B anonymisées sur 4 axes (correction, complétude, simplicité, risques de régression).
4. **Verdict** — `ALPHA`, `BETA`, ou `MERGE` (fusion des meilleurs éléments des deux).

## Quand l'utiliser

- Bugs difficiles à diagnostiquer
- Refactor multi-fichiers
- Décisions d'architecture avec tradeoffs
- Toute tâche où une mauvaise solution coûterait plus cher que le double de temps de calcul

## Quand ne pas l'utiliser

- Tâche triviale (1-2 lignes)
- Question conversationnelle / méta
- Modification purement documentaire
