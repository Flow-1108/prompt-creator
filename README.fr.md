# Prompt Creator — Skill Claude

Un skill Claude Code qui guide l'utilisateur pas à pas pour créer des prompts optimisés, quelle que soit la tâche.

---

## Ce que fait ce skill

Quand il se déclenche, le skill engage une **conversation naturelle** pour collecter le contexte nécessaire, puis génère **un seul prompt final** — sans explication, sans commentaire, prêt à copier-coller.

Le prompt généré s'adapte automatiquement selon la destination choisie :
- **Conversation claude.ai** → style naturel, conversationnel, longueur modérée
- **System prompt API** → structure exhaustive, balises XML, couverture des cas limites

---

## Phrases qui déclenchent le skill

- "crée-moi un prompt"
- "écris un system prompt"
- "améliore mon prompt"
- "génère des instructions pour Claude"
- "aide-moi à écrire un prompt pour…"
- "j'ai besoin d'un bon prompt pour…"
- *(et équivalents en anglais)*

---

## Structure du dossier

```
prompt-creator/
├── SKILL.md                        # Fichier principal du skill
├── README.md                       # Documentation en anglais
├── README.fr.md                    # Cette documentation (français)
└── references/
    └── prompt-patterns.md          # 5 structures de prompts réutilisables
```

---

## Comment ça marche

### 1. Phase de collecte
Le skill pose des questions **une par une**, dans un ordre logique, sans jargon technique. Il couvre au minimum :

| Question | Pourquoi |
|----------|----------|
| Objectif principal | Savoir ce que le prompt doit accomplir |
| Rôle / persona de Claude | Calibrer le ton et l'expertise |
| Audience cible | Adapter le niveau et le registre |
| Ton souhaité | Formel, pédagogique, conversationnel… |
| Destination | claude.ai ou system prompt API |
| Contraintes | Ce qu'il faut éviter absolument |
| Exemples d'entrée/sortie | Pour les tâches avec format précis |
| Format de sortie | Liste, JSON, texte libre, tableau… |

### 2. Phase de clarification
Le skill continue de poser des questions tant qu'il détecte des **ambiguïtés ou des informations manquantes**. Il décide lui-même quand il a assez de contexte — l'utilisateur n'a pas besoin de dire "c'est bon".

### 3. Génération du prompt
Une fois le contexte suffisant, le skill produit le prompt final en appliquant les **best practices Anthropic** :

- Balises XML sémantiques : `<role>`, `<context>`, `<instructions>`, `<examples>`, `<output_format>`
- Exemples few-shot (3 à 5) pour les tâches avec format ou ton précis
- Invitation au Chain of Thought (`<thinking>` / `<answer>`) pour les tâches analytiques
- Documents longs placés en haut du prompt
- Ton calme et direct — jamais de `MAJUSCULES ABUSIVES`, jamais de "NE FAIS JAMAIS"

---

## Principes d'ingénierie de prompts appliqués

Ce skill s'appuie sur les recommandations officielles d'Anthropic :

**Structure en composants**
Chaque prompt généré est découpé en sections claires avec des balises XML, ce qui améliore la compréhension de Claude et la cohérence des réponses.

**Few-shot examples**
Pour les tâches avec un format attendu (résumés, emails, fiches produit…), 3 à 5 exemples entrée/sortie sont intégrés directement dans le prompt.

**Chain of Thought**
Pour les tâches analytiques, multi-étapes ou impliquant une décision, le prompt invite Claude à raisonner dans une balise `<thinking>` avant de répondre dans `<answer>`.

**Ton calibré**
Langage direct et bienveillant. Aucune majuscule agressive, aucune formulation alarmiste. Les contraintes sont exprimées positivement.

---

## Installation

Place le dossier du skill dans ton répertoire de plugins Claude Code :

```
~/.claude/plugins/<ton-plugin>/skills/prompt-creator/
```

Ou directement dans :

```
~/.claude/skills/prompt-creator/
```

---

## Compatibilité

| Environnement | Supporté |
|---------------|----------|
| claude.ai | Oui |
| Claude Code | Oui |
| API (system prompt) | Oui — le skill peut générer des prompts pour cette cible |

---

## Fichiers de référence

Le fichier [`references/prompt-patterns.md`](./references/prompt-patterns.md) contient 5 structures de prompts réutilisables :

1. **Tâche analytique** — avec Chain of Thought
2. **Transformation de contenu** — résumé, réécriture, traduction
3. **Génération créative** — avec exemples few-shot
4. **Classification / extraction** — sorties structurées JSON
5. **Assistant conversationnel** — system prompt API complet

---

## Auteur

Créé avec [Claude Code](https://claude.ai/code) · [Flow-1108](https://github.com/Flow-1108)
