# Prompt Creator — Skill Claude (v3)

Un skill Claude Code qui guide l'utilisateur pas à pas pour créer des prompts optimisés pour **Claude**, **Gemini** ou **GPT**, quelle que soit la tâche.

---

## Ce que fait ce skill

Quand il se déclenche, le skill **demande d'abord pour quelle IA le prompt est destiné** (Claude, Gemini ou GPT), puis engage une conversation naturelle pour collecter le contexte nécessaire, et génère **un seul prompt final** — sans explication, sans commentaire, prêt à copier-coller.

Le prompt généré s'adapte selon :
- **L'IA cible** : Claude (best practices Anthropic), Gemini (framework PTCF de Google) ou GPT (framework CTCO d'OpenAI)
- **La destination** : conversation UI (claude.ai / AI Studio / ChatGPT) ou system prompt API

---

## Phrases qui déclenchent le skill

- "crée-moi un prompt"
- "écris un system prompt"
- "améliore mon prompt"
- "génère des instructions pour Claude / Gemini / GPT"
- "aide-moi à écrire un prompt pour…"
- "j'ai besoin d'un bon prompt pour…"
- "prompt pour mon app"
- "system prompt pour Gemini"
- "prompt pour ChatGPT"
- "system prompt pour GPT"
- *(et équivalents en anglais)*

---

## Structure du dossier

```
prompt-creator/
├── SKILL.md                        # Fichier principal du skill (v3)
├── README.md                       # Documentation en anglais
├── README.fr.md                    # Cette documentation (français)
└── references/
    ├── prompt-patterns.md          # 5 structures de prompts Claude
    ├── gemini-guide.md             # Guide Gemini : PTCF, patterns, best practices
    └── gpt-guide.md               # Guide GPT : CTCO, agents, reasoning
```

---

## Comment ça marche

### 0. Choix de l'IA cible
Toute première question, toujours : **"Ce prompt est pour Claude, Gemini ou GPT ?"** Ce choix détermine la structure, les patterns et les conventions utilisées pour la génération.

### 1. Phase de collecte
Le skill pose des questions **une par une**, dans un ordre logique, sans jargon technique. Il couvre au minimum :

| Question | Pourquoi |
|----------|----------|
| IA cible | Choisir les bonnes best practices (Claude / Gemini / GPT) |
| Objectif principal | Savoir ce que le prompt doit accomplir |
| Rôle / persona | Calibrer le ton et l'expertise |
| Audience cible | Adapter le niveau et le registre |
| Ton souhaité | Formel, pédagogique, conversationnel… |
| Destination | UI conversation ou system prompt API |
| Contraintes | Ce qu'il faut éviter absolument |
| Exemples d'entrée/sortie | Pour les tâches avec format précis |
| Format de sortie | Liste, JSON, texte libre, tableau… |
| Inputs multimodaux *(Gemini/GPT)* | Images, vidéos, audio à traiter ? |
| Long context *(Gemini/GPT)* | Documents volumineux en contexte ? |
| Comportement agent *(GPT)* | Agent autonome avec outils, ou assistant standard ? |
| Profondeur de raisonnement *(GPT)* | Analyse complexe ou exécution rapide ? |

### 2. Phase de clarification
Le skill continue de poser des questions tant qu'il détecte des **ambiguïtés ou des informations manquantes**. Il décide lui-même quand il a assez de contexte.

### 3. Génération du prompt
Une fois le contexte suffisant, le skill produit le prompt final en appliquant les **best practices de l'IA cible** :

**Pour Claude (Anthropic) :**
- Balises XML sémantiques : `<role>`, `<context>`, `<instructions>`, `<examples>`, `<output_format>`
- Exemples few-shot (3 à 5) pour les tâches avec format ou ton précis
- Chain of Thought (`<thinking>` / `<answer>`) pour les tâches analytiques
- Documents longs en haut du prompt
- Ton calme et direct — jamais de `MAJUSCULES ABUSIVES`

**Pour Gemini (Google) :**
- Framework PTCF : Persona · Task · Context · Format
- Few-shot examples en priorité (recommandation forte de Google)
- Instructions de planification explicites au lieu de `<thinking>` tags
- Contraintes positives (éviter les négations larges qui perturbent Gemini)
- Prompts concis — Gemini 3 suit bien les instructions sans sur-spécification
- Support natif multimodal (images, vidéo, audio)
- Long context jusqu'à 1M+ tokens

**Pour GPT (OpenAI) :**
- Framework CTCO : Context → Task → Constraints → Output
- Contraintes séparées de la tâche (réduit la dérive d'instructions)
- 3 instructions agentiques : Persistance + Planification + Outils (+20% SWE-bench)
- Instructions négatives toujours accompagnées d'une alternative positive
- Raisonnement adaptatif : preamble pour GPT standard, rien pour modèles Thinking
- `reasoning_effort` en API : low / medium / high selon la complexité
- Structured Outputs : contrainte JSON au niveau token via `response_format`
- Prompt caching : statique en haut, dynamique en bas

---

## Principes d'ingénierie de prompts

### Commun aux trois IAs
- **Structure en composants** — sections claires avec balises XML
- **Few-shot examples** — 3 à 5 exemples pour les tâches avec format attendu
- **Ton calibré** — langage direct, bienveillant, contraintes positives

### Spécifique à Claude
- **Chain of Thought** avec balises `<thinking>` / `<answer>`
- Instructions détaillées, comportements explicites

### Spécifique à Gemini
- **Framework PTCF** (structure officielle Google)
- **Planning explicite** au lieu de thinking tags
- **Concision** — Gemini 3 préfère les prompts courts et directs
- **Température à 1.0** — ne pas modifier pour Gemini 3
- **Multimodal natif** — questions spécifiques sur les médias, pas "analyse ceci"

### Spécifique à GPT
- **Framework CTCO** (structure officielle OpenAI)
- **Contraintes isolées** — section dédiée, séparée de la tâche
- **3 instructions agentiques** — persistance, planification, usage d'outils
- **Pas de CoT explicite pour les modèles Thinking** — déjà intégré en interne
- **Caching** — instructions statiques en haut, contenu variable en bas
- **Double placement** — instructions avant ET après les documents longs

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

**[`references/prompt-patterns.md`](./references/prompt-patterns.md)** — 5 structures de prompts Claude :
1. Tâche standard (API system prompt)
2. Tâche analytique (Chain of Thought)
3. Traitement de documents
4. Assistant conversationnel (claude.ai)
5. Classification / routage

**[`references/gemini-guide.md`](./references/gemini-guide.md)** — Guide complet Gemini :
1. Framework PTCF et template
2. 6 patterns de prompts (standard, analytique, documents, conversationnel, multimodal, JSON)
3. Best practices et pièges à éviter
4. Conventions de tags XML pour Gemini

**[`references/gpt-guide.md`](./references/gpt-guide.md)** — Guide complet GPT/OpenAI :
1. Framework CTCO et template
2. 6 patterns de prompts (standard, reasoning, agent, documents longs, conversationnel, JSON)
3. 3 instructions agentiques (+20% SWE-bench)
4. Best practices, pièges à éviter, conventions XML

---

## Auteur

Créé avec [Claude Code](https://claude.ai/code) · [Flow-1108](https://github.com/Flow-1108)
