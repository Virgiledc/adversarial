---
name: adversarial
description: "Use when the user says '/adversarial', 'mode adversarial', 'dual solve', 'deux agents', or for any complex task where quality matters more than speed. Spawns two competing agents with different strategies, then a judge agent picks the best solution."
---

# Adversarial Mode - Dual Solve with Judge

Deux agents rivaux. Un juge. La meilleure solution gagne.

## INTERDIT AVANT PHASE 2

NE PAS lire du code, explorer le codebase, ou appeler Glob/Grep/Read avant d'avoir lance les agents en Phase 2. C'est le travail des agents, pas le tien.

## Exemple complet de workflow

Voici EXACTEMENT ce que l'utilisateur doit voir. Reproduis cette structure :

```
[MON OUTPUT - Phase 1]
**Objectif** : Diagnostiquer pourquoi le scraper retourne des posts vides et corriger.
**Contraintes** : scraper.py, service.py — ne pas casser la rotation de comptes.
**Succes** : Posts extraits correctement ou erreur loguee explicitement.

Validation ?

[UTILISATEUR REPOND "oui"]

[MON OUTPUT - Phase 2]
→ Agent(description="Alpha - Pragmatique", prompt="Tu es un developpeur senior pragmatique. OBLIGATOIRE : lis tout le code concerne AVANT... [probleme + fichiers]", isolation="worktree")
→ Agent(description="Beta - Sceptique", prompt="Tu es un architecte sceptique. OBLIGATOIRE : lis tout le code concerne AVANT... [probleme + fichiers]", isolation="worktree")
[Les 2 appels Agent dans le MEME message]

[AGENTS TERMINES]

[MON OUTPUT - Phase 3]
→ Agent(description="Juge", prompt="... --- SOLUTION A --- [output agent 1] --- SOLUTION B --- [output agent 2] ... Verdict : A, B, ou MERGE")
[AUCUNE mention d'Alpha/Beta/Pragmatique/Sceptique dans ce prompt]

[JUGE TERMINE]

[MON OUTPUT - Phase 4]
**Verdict** : MERGE — la solution A corrige le filtre, la B ajoute un cas limite critique.
[Solution retenue avec code]
```

## Quand utiliser

- Taches complexes multi-fichiers
- Bugs difficiles a diagnostiquer
- Decisions d'architecture avec tradeoffs
- Tout cas ou l'utilisateur dit "adversarial" ou "je veux le meilleur resultat"

## Workflow

### Phase 1 : Cadrer le probleme

Avant de lancer les agents, formuler clairement :
- **L'objectif** : qu'est-ce qu'on resout ?
- **Les contraintes** : fichiers concernes, conventions du projet, limites
- **Le critere de succes** : comment on juge que c'est bon ?

OBLIGATOIRE — Presenter ce cadrage a l'utilisateur en EXACTEMENT 3 lignes, une par section. C'est la PREMIERE chose a afficher, avant tout autre output :
- **Objectif** : 1 phrase
- **Contraintes** : 1 phrase (fichiers + limites, separes par virgules — JAMAIS de bullets)
- **Succes** : 1 phrase

Exemple :
> **Objectif** : Diagnostiquer pourquoi le scraper retourne des posts vides et corriger.
> **Contraintes** : `scraper.py`, `service.py` — ne pas casser la rotation de comptes.
> **Succes** : Posts extraits correctement ou erreur loguee explicitement.

MAXIMUM 3 lignes. Pas plus. Attendre validation avant de lancer les agents.

### Phase 2 : Dual Solve

Lancer **2 agents en parallele** (dans le MEME message, MEME bloc d'appels) avec des prompts differents :

**Agent Alpha** — Le Pragmatique
```
Prompt : Tu es un developpeur senior pragmatique. Resous ce probleme de la facon la plus directe et simple possible. Pas d'over-engineering. Le code le plus court qui marche correctement. OBLIGATOIRE : lis tout le code concerne AVANT de proposer une solution.

[description du probleme + fichiers + contraintes]

Retourne :
1. Ce que tu as lu et compris du code existant
2. Ta solution (code complet, pret a copier)
3. Les risques/limites de ta solution
```

**Agent Beta** — Le Sceptique
```
Prompt : Tu es un architecte sceptique et rigoureux. Resous ce probleme en challengeant chaque hypothese. Cherche les cas limites, les regressions possibles, les N+1 queries. OBLIGATOIRE : lis tout le code concerne AVANT de proposer une solution. Verifie que ta solution ne casse rien d'autre.

[description du probleme + fichiers + contraintes]

Retourne :
1. Ce que tu as lu et compris du code existant
2. Les hypotheses que tu as challengees
3. Ta solution (code complet, pret a copier)
4. Les cas limites que tu as verifies
```

Les deux agents doivent utiliser `isolation: "worktree"` pour les taches qui modifient du code.

### Phase 3 : Le Juge

Quand les deux agents ont fini, lancer un **3eme agent** :

**Agent Juge** — L'Arbitre
```
Prompt : Tu es un tech lead senior qui fait une code review. Voici deux solutions au meme probleme. Compare-les objectivement.

Probleme : [description]
Contraintes projet : [conventions CLAUDE.md pertinentes]

--- SOLUTION A ---
[coller ici l'output complet du premier agent, tel quel]

--- SOLUTION B ---
[coller ici l'output complet du second agent, tel quel]

Analyse :
1. Correction : laquelle est correcte ? Les deux ? Aucune ?
2. Completude : laquelle couvre plus de cas ?
3. Simplicite : laquelle est plus maintenable ?
4. Risques : laquelle a moins de risques de regression ?

Verdict : A, B, ou MERGE (prendre le meilleur des deux).
Si MERGE, fournis la solution fusionnee.
```

**ANONYMISATION OBLIGATOIRE** — mots INTERDITS dans le prompt Juge :
- "Alpha", "Beta", "Pragmatique", "Sceptique" ou tout qualificatif de strategie
- Toute indication permettant au juge de deviner quel agent a ecrit quelle solution
- Le prompt Juge ne contient QUE "Solution A" et "Solution B" comme etiquettes
- Le verdict est TOUJOURS "A", "B", ou "MERGE" — rien d'autre
- Meme les placeholders et descriptions internes doivent etre neutres : ecrire "[output premier agent]" — JAMAIS "[output Alpha]" ou "[output agent pragmatique]"

### Phase 4 : Presenter le resultat

Montrer a l'utilisateur :
- Le verdict du juge (1-2 lignes MAX — pas un paragraphe, pas un dump)
- La solution retenue
- Les points forts de la solution rejetee qui ont ete integres (si MERGE)

NE PAS montrer les solutions brutes des deux agents sauf si l'utilisateur le demande.

**ANONYMISATION PHASE 4** — les memes mots sont INTERDITS ici aussi : ne JAMAIS dire "Alpha", "Beta", "le pragmatique", "le sceptique" en presentant le resultat a l'utilisateur. Utiliser uniquement "Solution A"/"Solution B" ou decrire directement le contenu technique.

## Regles

- **JAMAIS lancer les agents sans cadrage valide par l'utilisateur**
- **TOUJOURS utiliser des worktrees** pour les taches qui modifient du code
- **Le juge ne voit PAS quel agent est Alpha ou Beta** — presenter comme "Solution A" et "Solution B" pour eviter le biais
- **Si le juge dit "aucune des deux"** — le signaler a l'utilisateur et proposer une 3eme approche
