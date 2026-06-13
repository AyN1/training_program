# Training Plan — Brainstorm & Roadmap

Récapitulatif du brainstorm produit/UX et de l'avancement.
Légende : ✅ fait · 🟡 partiel · ⏸️ reporté volontairement · ⬜ à faire

---

## État des versions livrées

| Version | Contenu | Tag |
|---|---|---|
| **v0** | PWA installable (manifest + icône + service worker) + rappel de sauvegarde + `storage.persist()` | `v0` |
| **v1** | Notes contextuelles (exercice + séance) · flux save/reset sécurisé · fix prefill | `v1` |
| **v2** | Chips de notes rapides (« machine prise » · « autre salle » · « élastiques ») | `v2` |
| **v3** | ST3 — substitution rapide (7 exercices machines/poulies) | `v3` |
| **v4** | QW6 (lisibilité, cibles tactiles, wake lock, zoom réactivé) + QW4 (steppers de poids ±2,5 kg) | `v4` |

---

## Axe 1 — Design / UI / UX

### Problèmes observés (constat initial)
- Cartes toutes dépliées en permanence → scroll important pendant la séance.
- Le timer de repos disparaît de l'écran quand on scrolle ; aucun indicateur global.
- Saisie de poids = clavier à chaque fois ; pas de +/- pour les poids.
- Cibles tactiles < 44px (+/-, timer) ; textes 11-12px et placeholders très peu contrastés (`--faint`).
- ✅ « Réinitialiser » collé à « Sauvegarder », sans garde → corrigé en v1.
- ✅ Brouillon non vidé après sauvegarde, doublons possibles → corrigé en v1.
- 7 onglets scrollables sans indice de débordement ; ouverture toujours sur S1.

### Idées d'amélioration (priorisées impact / effort)

**Impact fort / effort faible**
- ✅ **QW1 — Sécuriser save/reset** (confirmation, reset discret, pastille brouillon, anti-double-save) — *v1*
- ✅ **QW2 — Fix prefill** (un poids suggéré validé est bien sauvegardé) — *v1*
- ✅ **QW4 — Steppers poids ±2,5 kg** autour de chaque champ poids (moins de clavier ; +/- confirme le prefill gris) — *v4*
- ⬜ **QW5 — Chip « séance suggérée du jour »** (rotation S1→S2→S3→S4 d'après le dernier log)
- ✅ **QW6 — Lisibilité & cibles tactiles** (`--faint` éclairci, +/- agrandis, inputs 16px → fin de l'auto-zoom iOS) **+ wake lock** (écran allumé en séance) **+ zoom réactivé** (`maximum-scale` retiré) — *v4*

**Impact fort / effort moyen**
- ⏸️ **INT2 — Mode focus** (cartes repliables, seul l'exercice en cours déplié) — *reporté à la demande*
- ⬜ **INT1 — Timer de repos sticky** (barre fixe bas d'écran) + démarrage auto à l'incrément de série ; passer le timer en horodatage (`Date.now`) pour corriger la dérive iOS en arrière-plan
- ⬜ **INT5 — Bottom nav fixe** (Séances / Cardio / Corps / Carnet) à portée de pouce
- ⬜ **INT3 — Historique par exercice** (tap sur le nom → 5 dernières perfs)
- ⬜ **Écran récap post-sauvegarde** (volume, exercices validés, PR éventuels)

**Nice to have**
- ⬜ **INT4 — Dates ISO dans les entrées** (prérequis aux graphiques) + corriger `fmtISO` (bug UTC entre minuit et ~2h)
- ⬜ Graphiques de progression (poids, BF%)
- ⬜ Streak / régularité hebdo
- ⬜ Swipe horizontal entre séances ; réordonner les exercices ; célébration PR

---

## Axe 2 — Logging en salle différente / voyage

Principe directeur : le **créneau de mouvement** est l'unité stable du programme ; l'exercice n'en est qu'une incarnation selon le matériel disponible. Adaptations **session-only**, sans réécrire le programme.

- ✅ **Notes contextuelles** (note par exercice + note de séance, échappées au carnet) — *v1* (c'était le MVP voyage)
- ✅ **Chips de notes rapides** (raccourcis 1-tap) — *v2*
- ✅ **ST3 — Substitution rapide** (bouton ⇄, 3 équivalents + retour, badge « remplacé », carnet « prévu → équivalent », pas d'héritage de charge, pas de mélange d'historique) — *v3, Batch 1 : machines/poulies*
- ⬜ **ST3 Batch 2** — équivalents pour les poids libres composés (`s1e3` Développé couché, `s2e1a` Développé incliné, `s2e2a` Squat/Hack)
- ⬜ **ST3 v2** — mémoire de poids par équivalent (si on subs souvent le même)
- ⬜ **ST2 — Chips d'équipement** généralisées (Machine/Haltères/Barre/Poulie/PDC) avec mémoire de poids par variante (generalisation du pattern B2 RDL/Deadlift existant)
- ⬜ **ST4 — Profils de salle** (poids libres globaux, poids machine par salle ; badge salle dans la date-bar)

> Note : la table `SUBS` (7 exercices, dans `index.html`) liste des équivalents proposés — à ajuster selon la salle réelle.

---

## Changements structurels

- ⬜ **ST1 — Catalogue d'exercices data-driven** : remplacer ~300 lignes de HTML dupliqué + 3 maps JS synchronisées à la main par un tableau `EXERCISES` unique, cartes générées. Débloque ST2/ST4 proprement. (Gros diff, à conserver les `exId` actuels pour zéro migration localStorage.)

---

## Reliquats de l'audit code initial (qualité / accessibilité)

- 🟡 **Échappement HTML / import** : `esc()` appliqué aux notes, à la date et au `cardio_type` du carnet (*v1*) ; reste : import JSON sans validation ni confirmation, écrasement silencieux.
- ⬜ **Timer audio** : `AudioContext` recréé à chaque fois (fuite + plafond navigateur) et bip de fin probablement bloqué sur iOS (hors geste utilisateur). Réutiliser un contexte unique, le débloquer au tap de démarrage.
- ⬜ **`--meas:#A78BFA;` parasite** (≈ ligne 185) hors de tout sélecteur → avale la règle `.meas-accent` suivante.
- ⬜ **`buildWeightHistory()`** défini mais jamais appelé (code mort).
- 🟡 **Accessibilité** : zoom réactivé + contraste `--faint` amélioré + cibles tactiles agrandies (*v4*) ; reste : aucun `aria-*`/`role`, `<div onclick>` non focusables.
- ⬜ **Sémantique** : le statut « skip » compte comme progression dans la barre (choix à trancher).
- ⬜ **Pas de `max-width`** : sur grand écran la colonne s'étire (app pensée mobile).

---

## Prochaines étapes recommandées

1. **QW5** — chip « séance suggérée ».
2. **INT1 → INT5/INT2** — timer sticky puis bottom nav puis mode focus (à penser ensemble, plus gros chantier visuel).
3. **INT4** — dates ISO, prérequis avant tout graphique de progression.
4. **ST3 Batch 2** / ajustement de la table `SUBS`.
5. **ST1** (data-driven) si on veut industrialiser ST2/ST4.
