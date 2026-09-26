# Prompt Creator — Skill Claude (v4)

Un skill Claude Code qui guide l'utilisateur pas à pas pour créer des prompts optimisés pour **Claude**, **Gemini** ou **GPT**, quelle que soit la tâche.

> **Contenu vérifié en septembre 2026.** Les noms de modèles et paramètres d'API évoluent vite chez les trois fournisseurs. Les fichiers de référence portent chacun une date de vérification — si elle date de plus de quelques mois, revérifie avant de te fier à un nom de modèle précis.

---

## Ce que fait ce skill

Quand il se déclenche, le skill **demande d'abord pour quelle IA le prompt est destiné** (Claude, Gemini ou GPT), puis engage une conversation naturelle pour collecter le contexte nécessaire, et génère **un seul prompt final** — sans explication, sans commentaire, prêt à copier-coller.

Le prompt généré s'adapte selon :
- **L'IA cible** : Claude (best practices Anthropic), Gemini (framework PTCF de Google) ou GPT (framework CTCO)
- **La destination** : conversation UI (claude.ai / AI Studio / ChatGPT) ou system prompt API

---

## Le principe qui structure la v4

Les trois fournisseurs ont convergé sur la même architecture : **la profondeur de raisonnement est devenue un paramètre, plus du texte de prompt.**

| Fournisseur | Paramètre de profondeur |
|---|---|
| Anthropic | `thinking: {type: "adaptive"}` + `output_config.effort` (`low` → `max`) |
| OpenAI | `reasoning_effort` (`none` → `max`) |
| Google | `thinking_level` (`low` / `medium` / `high`) |

Conséquences concrètes sur la rédaction des prompts :

- **Plus de scaffold Chain of Thought.** « Think step by step », balises `<thinking>`, chorégraphies de planification : sur les modèles actuels ces consignes doublonnent un raisonnement déjà interne et peuvent **dégrader** la sortie.
- **Plus de paramètres d'échantillonnage.** Les modèles frontier de Claude rejettent `temperature`/`top_p`/`top_k` ; Gemini 3+ demande de les retirer des configs.
- **Les sorties structurées sont une fonctionnalité d'API** chez les trois. Le bricolage au niveau du prompt (prefill, stop sequences, « produis UNIQUEMENT du JSON valide », retry sur parse) est obsolète.
- **Le suivi d'instructions est devenu littéral.** L'emphase gonflée sur-déclenche, et les précautions résiduelles (« essaie de », « si possible ») sont lues comme une permission d'en faire moins.

Le rôle du prompt est désormais de porter le **contexte et l'intention** — audience, produit, exigence de qualité, contraintes et leurs raisons. C'est ce que seul l'auteur connaît.

---

## Modèles actuels (vérifié septembre 2026)

| Fournisseur | Modèles |
|---|---|
| **Anthropic** | Claude Fable 5.1 (`claude-fable-5-1`), Opus 5 (`claude-opus-5`), Sonnet 5 (`claude-sonnet-5`), Haiku 4.5 (`claude-haiku-4-5`) |
| **OpenAI** | GPT-6 Astra (`gpt-6-astra`), GPT-6 Sol (`gpt-6-sol`), GPT-6 Luna (`gpt-6-luna`), GPT-5.6 Terra (`gpt-5.6-terra` — pas de GPT-6 Terra) |
| **Google** | Gemini 3.8 / 3.7 / 3.6 / 3.5 Flash, 3.5 Flash-Lite, 3.1 Pro (preview) |

`o3` ne figure plus au catalogue OpenAI. La distinction pertinente n'est plus *quel modèle raisonne* mais *à quel niveau d'effort*.

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
├── SKILL.md                        # Fichier principal du skill (v4)
├── README.md                       # Documentation en anglais
├── README.fr.md                    # Cette documentation (français)
└── references/
    ├── prompt-patterns.md          # Guide Claude : modèles, patterns, patterns datés
    ├── gemini-guide.md             # Guide Gemini : PTCF, thinking_level, patterns
    └── gpt-guide.md                # Guide GPT : CTCO, reasoning_effort, agents
```

---

## Comment ça marche

### 0. Choix de l'IA cible
Toute première question, toujours : **"Ce prompt est pour Claude, Gemini ou GPT ?"** Ce choix détermine la structure, les patterns et les conventions utilisées pour la génération.

### 0.5. Recommandation de skills *(Claude + Claude Code uniquement)*
Après avoir compris l'objectif, le skill vérifie si un skill Claude Code disponible couvre la tâche. **Il lit la liste de skills de la session en cours** plutôt qu'un catalogue figé — l'ensemble installé diffère selon la machine, les marketplaces et l'utilisateur, et s'enrichit à chaque version. S'il trouve une correspondance, il la **signale** et continue la création du prompt en l'**optimisant pour ce skill** — les deux sont complémentaires, pas alternatifs. Sinon, cette phase est silencieuse.

> Exemple : l'utilisateur veut traiter des PDFs → le skill signale le skill `pdf` s'il est disponible, et génère un prompt calibré pour l'utiliser efficacement.

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
| Comportement agent | Agent autonome avec outils, ou assistant standard ? |
| Profondeur de raisonnement | Analyse complexe ou exécution rapide ? Se traduit en paramètre chez les trois fournisseurs |
| Modèle précis *(API, si ça change quelque chose)* | Un tier économique peut nécessiter plus d'explicite qu'un flagship |

### 2. Phase de clarification
Le skill continue de poser des questions tant qu'il détecte des **ambiguïtés ou des informations manquantes**. Il décide lui-même quand il a assez de contexte.

### 3. Génération du prompt
Une fois le contexte suffisant, le skill produit le prompt final en appliquant les **best practices de l'IA cible** :

**Pour Claude (Anthropic) :**
- Balises XML sémantiques : `<role>`, `<context>`, `<instructions>`, `<examples>`, `<output_format>`
- Exemples few-shot (3 à 5) variés délibérément — Claude calque leur longueur, leur ton et leur structure
- **Pas de scaffold de raisonnement** : la profondeur passe par `effort`, pas par des balises `<thinking>`. Sur Claude Fable 5.1, demander au modèle de reproduire son raisonnement peut déclencher un refus
- Documents longs en haut du prompt
- Ton calme et direct — jamais de `MAJUSCULES ABUSIVES`
- Pas de prefill ni de bricolage JSON au niveau du prompt : `output_config.format` s'en charge

**Pour Gemini (Google) :**
- Framework PTCF : Persona · Task · Context · Format
- Few-shot examples en priorité (recommandation forte de Google)
- **`thinking_level`** (`low` / `medium` / `high`) plutôt que des instructions de planification. `minimal` renvoie une erreur
- **Aucun paramètre d'échantillonnage** : `temperature`, `top_p`, `top_k` et `candidate_count` sont à retirer sur Gemini 3+
- Contraintes positives (éviter les négations larges qui perturbent Gemini)
- Délimiteurs cohérents : XML *ou* Markdown, jamais les deux mélangés
- Support natif multimodal, long context jusqu'à 1M tokens

**Pour GPT (OpenAI) :**
- Framework CTCO : Context → Task → Constraints → Output
- Contraintes séparées de la tâche (réduit la dérive d'instructions)
- 3 instructions agentiques : Persistance + Périmètre/complétion + Outils
- Instructions négatives toujours accompagnées d'une alternative positive
- `reasoning_effort` : `none`/`low` pour extraction et formatage, `medium` par défaut, `high`/`xhigh`/`max` pour les problèmes difficiles
- Structured Outputs : contrainte JSON au niveau token via `response_format`
- Prompt caching : statique en haut, dynamique en bas
- Spécificités GPT-6 (Astra, Sol, Luna) : pousser à l'action quand l'intention est claire, demander explicitement de la prose (il formate en listes par défaut), cadrer l'ampleur des tests sur les tâches de code

---

## Principes d'ingénierie de prompts

### Commun aux trois IAs
- **Structure en composants** — sections claires avec balises XML
- **Few-shot examples** — 3 à 5 exemples pour les tâches avec format attendu
- **Ton calibré** — langage direct, contraintes positives, emphase méritée et non systématique
- **Profondeur en configuration** — jamais de « think step by step » dans le texte du prompt
- **Dire les choses une fois** — les prompts allégés surpassent mesurablement les prompts rembourrés

### Spécifique à Claude
- **Adaptive thinking + `effort`** à la place des balises de raisonnement
- Contexte riche et comportements explicites ; pas de chorégraphie en étapes pour les tâches de jugement

### Spécifique à Gemini
- **Framework PTCF** (structure recommandée par Google)
- **`thinking_level`** à la place du planning explicite
- **Concision** — Gemini 3 suit bien les instructions sans sur-spécification
- **Aucun paramètre d'échantillonnage** — `temperature` doit être absent, pas réglé
- **Multimodal natif** — questions spécifiques sur les médias, pas "analyse ceci"

### Spécifique à GPT
- **Framework CTCO** — convention fiable, alignée sur la doc OpenAI (voir la nuance ci-dessous)
- **Contraintes isolées** — section dédiée, séparée de la tâche
- **3 instructions agentiques** — persistance, périmètre/complétion, usage d'outils
- **Pas de CoT explicite** — tous les flagships actuels raisonnent en interne
- **Caching** — instructions statiques en haut, contenu variable en bas
- **Double placement** — instructions avant ET après les documents longs

---

## Points non vérifiés

Deux affirmations de la documentation sont explicitement signalées comme non confirmées dans les fichiers de référence :

- **CTCO** est une convention communautaire largement utilisée et cohérente avec les recommandations d'OpenAI, mais elle n'a pas été retrouvée comme framework nommé dans la documentation officielle d'OpenAI (vérifié le 2026-09-12). Les versions antérieures de ce skill la présentaient comme « structure officielle OpenAI ».
- **L'étendue du `reasoning_effort` de GPT-6 Astra** diffère entre deux pages officielles OpenAI (`low`/`medium`/`high` d'un côté, jusqu'à `max` de l'autre). Les deux s'accordent sur le fait que `none` n'est pas supporté.

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

**[`references/prompt-patterns.md`](./references/prompt-patterns.md)** — Guide Claude :
1. Panorama des modèles et comportements d'API qui changent la rédaction
2. Patterns datés à ne plus générer
3. 5 structures de prompts (tâche standard, analytique, documents, conversationnel, classification)
4. Conventions de balises XML

**[`references/gemini-guide.md`](./references/gemini-guide.md)** — Guide Gemini :
1. Panorama des modèles et config de génération (Gemini 3+)
2. Framework PTCF et template
3. 6 patterns de prompts (standard, analytique, documents, conversationnel, multimodal, JSON)
4. Best practices, pièges à éviter, conventions de tags XML

**[`references/gpt-guide.md`](./references/gpt-guide.md)** — Guide GPT/OpenAI :
1. Panorama des modèles et niveaux de `reasoning_effort`
2. Spécificités comportementales de la famille GPT-6
3. Framework CTCO et template
4. 6 patterns de prompts (standard, reasoning, agent, documents longs, conversationnel, JSON)
5. Best practices, pièges à éviter, conventions XML

---

## Auteur

Créé avec [Claude Code](https://claude.ai/code) · [Flow-1108](https://github.com/Flow-1108)
