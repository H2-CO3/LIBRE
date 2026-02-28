# TODO — Projet LIBRE : Spin-off E.V.A. par agents IA

> Objectif : permettre aux agents IA de Moltbook d'ecrire librement des lignes narratives
> du spin-off LIBRE, chaque branche Git devenant un livre potentiel a part entiere.
> L'agent OpenClaw (heberge sur VPS Hostinger) sert uniquement de coordinateur/concierge.

---

## Phase 0 — Preparation locale

- [x] Reorganiser les fichiers du projet selon la structure prevue dans le README
- [x] Initialiser le repo Git local (`git init`)
- [x] Creer le fichier `.gitignore`
- [x] Creer le fichier `CODEOWNERS` pour proteger le canon
- [x] Premier commit : "Initial commit — canon et structure du projet LIBRE"

---

## Phase 1 — GitHub (repo public)

### 1.1 Creation du repo
- [x] Creer le repo public `LIBRE` sur GitHub (https://github.com/H2-CO3/LIBRE)
- [x] Pousser le commit initial

### 1.2 Rulesets de securite
- [x] Ruleset "Forteresse main" — push direct bloque, PR obligatoire, force push interdit
- [x] Ruleset "Branches narratives" — line/** : suppression et force push interdits
- [x] Verification : creation branche line/ OK, suppression bloquee OK

### 1.3 Token d'acces
- [x] Fine-Grained PAT cree (scope : LIBRE only, 90 jours)
- [ ] Stocker le token sur le VPS (variable d'environnement)
- [ ] Documenter la date d'expiration pour ne pas oublier la rotation

---

## Phase 2 — VPS Hostinger (KVM 4)

### 2.1 Achat et provisionnement
- [x] Commander le VPS KVM 4 sur Hostinger (4 vCPU / 16 GB RAM / 200 GB NVMe)
- [x] Region : Europe
- [x] OS : Ubuntu 22.04
- [x] Cle SSH configuree
- [ ] Attendre le provisionnement — puis recuperer l'IP

### 2.2 Securisation du VPS
- [ ] Configurer l'acces SSH par cle uniquement (desactiver mot de passe)
- [ ] Installer et configurer fail2ban
- [ ] Configurer le pare-feu (ufw) :
  - Ouvrir : SSH (22), HTTPS (443) si necessaire
  - Tout le reste ferme
- [ ] Creer un utilisateur non-root pour OpenClaw
- [ ] Mettre en place les mises a jour automatiques de securite (unattended-upgrades)

### 2.3 Installation Ollama (heartbeat gratuit)
- [ ] Installer Ollama sur le VPS : `curl -fsSL https://ollama.ai/install.sh | sh`
- [ ] Telecharger le modele leger : `ollama pull llama3.2:3b`
- [ ] Verifier qu'Ollama tourne : `ollama run llama3.2:3b "respond with OK"`
- [ ] Configurer Ollama en service systemd pour qu'il redémarre automatiquement
- [ ] Verifier la consommation RAM (~2 GB sur 16 GB disponibles)

---

## Phase 3 — Configuration OpenClaw

### 3.1 Fichiers de personnalite de l'agent

- [ ] Creer `SOUL.md` — principes fondamentaux de l'agent coordinateur :
  - Tu es le gardien du projet LIBRE, un spin-off de la serie E.V.A.
  - Tu ne crees PAS de contenu narratif
  - Tu accueilles les agents, reponds a leurs questions sur le canon
  - Tu traduis leurs propositions en branches Git
  - Tu ne juges pas la qualite, seulement le format
  - Tu es bienveillant, enthousiaste, et tu encourages la creativite
- [ ] Creer `USER.md` — contexte du proprietaire :
  - Nom : H2CO3
  - Mission : faire ecrire le spin-off LIBRE par des agents IA autonomes
  - Budget : minimal (< 15$/mois en tokens)
  - Priorite : securite du canon, liberte totale sur les branches
- [ ] Creer `IDENTITY.md` — identite publique de l'agent sur Moltbook :
  - Nom de l'agent (a choisir : "LIBRE-Coordinator", "H2CO3-Agent", etc.)
  - Bio courte pour le profil Moltbook

### 3.2 Configuration OpenClaw (optimisation tokens)

- [ ] Editer `~/.openclaw/openclaw.json` :
  ```json
  {
    "agents": {
      "defaults": {
        "model": {
          "primary": "anthropic/claude-haiku-4-5"
        },
        "cache": {
          "enabled": true,
          "ttl": "5m",
          "priority": "high"
        },
        "models": {
          "anthropic/claude-sonnet-4-5": {
            "alias": "sonnet",
            "cache": true
          },
          "anthropic/claude-haiku-4-5": {
            "alias": "haiku",
            "cache": false
          }
        }
      }
    },
    "heartbeat": {
      "every": "1h",
      "model": "ollama/llama3.2:3b",
      "session": "main",
      "target": "moltbook",
      "prompt": "Check m/libre for new proposals. Process any pending contributions."
    }
  }
  ```
- [ ] Ajouter les regles d'optimisation au system prompt de l'agent :
  - Session Init : charger uniquement SOUL.md + USER.md + IDENTITY.md
  - Model routing : Haiku par defaut, Sonnet jamais (pas de taches complexes)
  - Rate limits : 5s entre appels API, budget journalier 2$, mensuel 50$
  - Alerte a 75% du budget

### 3.3 Cle API Anthropic
- [ ] Creer une cle API Anthropic dediee a ce projet (budget minimal)
  - OU utiliser les credits Nexos AI de Hostinger (offre doublee au premier achat)
- [ ] Stocker la cle en variable d'environnement sur le VPS (`ANTHROPIC_API_KEY`)
- [ ] Ne JAMAIS la stocker dans un fichier ou le repo

---

## Phase 4 — Moltbook

### 4.1 Inscription de l'agent
- [ ] Enregistrer l'agent via l'API Moltbook :
  - `POST https://api.moltbook.com/agents/register`
  - Nom, description, email du owner
- [ ] Recuperer la cle API Moltbook (`moltbook_sk_...`)
- [ ] Stocker la cle en variable d'environnement sur le VPS (`MOLTBOOK_API_KEY`)
- [ ] Completer la verification (process via X/Twitter)
- [ ] Ne PAS installer de skill tierce Moltbook (risque cybersecurite)

### 4.2 Creation du submolt
- [ ] Creer le submolt `m/libre` (ou `m/eva-libre`) :
  - `POST https://api.moltbook.com/submolts`
  - Description : presentation du projet, lien vers le repo GitHub
- [ ] Publier le post d'accueil epingle (pin) :
  - Qu'est-ce que LIBRE
  - Comment contribuer (poster un chapitre sur m/libre)
  - Format attendu (reprendre le format de SYSTEM_PROMPT.md)
  - Lien vers le repo GitHub pour lire le canon
- [ ] Publier un post avec le resume du canon (pour que les agents n'aient pas a tout lire sur GitHub)
- [ ] Publier le SYSTEM_PROMPT.md adapte pour Moltbook

### 4.3 Configuration de l'interaction Moltbook <-> Git
- [ ] Programmer le comportement de l'agent OpenClaw pour surveiller m/libre :
  - Toutes les heures (via heartbeat), verifier les nouveaux posts
  - Si un post contient une proposition de chapitre :
    1. Extraire le nom de l'agent auteur
    2. Extraire le contenu du chapitre
    3. Creer une branche `line/{nom-agent}/{slug-titre}`
    4. Committer le chapitre au format Markdown
    5. Repondre sur Moltbook avec le lien vers la branche GitHub
  - Si un post est une question sur le canon :
    1. Repondre en citant le fichier canon pertinent
  - Si un post est un fork d'une ligne existante :
    1. Creer une sous-branche `line/{agent-original}/{titre}/fork-{nouvel-agent}`
    2. Committer le contenu
    3. Repondre avec le lien

---

## Phase 5 — Skill Git pour l'agent OpenClaw

- [ ] Creer la skill OpenClaw qui gere les operations Git :
  - `git clone` du repo au demarrage
  - `git pull` avant chaque operation
  - `git checkout -b line/{agent}/{slug}` pour chaque nouvelle proposition
  - `git add` + `git commit` avec message standardise :
    ```
    [LIBRE] Ligne "{titre}" par {agent}

    Propose via Moltbook le {date}
    Agent: {nom-agent-moltbook}
    Source: {lien-post-moltbook}
    ```
  - `git push origin line/{agent}/{slug}`
- [ ] Creer la skill de monitoring :
  - Lister les branches existantes
  - Compter les lignes narratives actives
  - Generer un resume pour le post hebdomadaire sur Moltbook
- [ ] Tester le workflow complet en local avant deploiement :
  - Simuler un post Moltbook → verifier la creation de branche

---

## Phase 6 — Lancement et communication

- [ ] Verifier le workflow de bout en bout :
  - Post sur Moltbook → detection par l'agent → branche creee sur GitHub → reponse sur Moltbook
- [ ] Poster l'annonce de lancement sur m/libre
- [ ] Explorer d'autres submolts Moltbook pour faire connaitre le projet :
  - Poster dans des submolts lies a la creativite, l'ecriture, l'IA
  - L'agent peut mentionner le projet quand c'est pertinent dans des discussions
- [ ] Mettre a jour le README.md du repo GitHub avec :
  - Badge "Contributions par agents IA bienvenues"
  - Lien vers m/libre sur Moltbook
  - Liste dynamique des branches actives (ou script pour generer)

---

## Phase 7 — Monitoring et maintenance

### Couts a surveiller
- [ ] Mettre en place un suivi hebdomadaire des couts :
  - VPS Hostinger : ~10$/mois (fixe)
  - Tokens API Anthropic : objectif < 5$/mois
  - Total cible : < 15$/mois
- [ ] Configurer les alertes budget dans OpenClaw (75% du budget journalier)
- [ ] Verifier chaque semaine le dashboard Anthropic pour les couts reels

### Securite continue
- [ ] Rotation du token GitHub tous les 90 jours
- [ ] Rotation de la cle API Moltbook si compromis suspect
- [ ] Verifier les logs du VPS chaque semaine (tentatives SSH, etc.)
- [ ] Mise a jour OpenClaw quand de nouvelles versions sortent
- [ ] Surveiller les annonces de securite Moltbook (precedent de la faille de janvier 2026)

### Suivi du projet
- [ ] Post hebdomadaire automatique sur m/libre : etat du projet, branches actives, stats
- [ ] Garder un oeil sur la qualite des contributions (meme sans validation, curiosite)
- [ ] Si une branche devient particulierement riche → la signaler sur Moltbook pour attirer plus d'agents dessus
- [ ] Documenter les moments interessants (quand un agent fait un choix narratif surprenant)

---

## Budget previsionnel mensuel

| Poste                        | Cout estime   |
|------------------------------|---------------|
| VPS Hostinger KVM 4          | ~10 $/mois    |
| Tokens API Anthropic (Haiku) | ~3-5 $/mois   |
| Ollama (heartbeat)           | 0 $ (local)   |
| GitHub (public)              | 0 $           |
| Moltbook                     | 0 $           |
| **TOTAL**                    | **~13-15 $/mois** |

---

## Notes importantes

- **Le canon ne bouge jamais.** Toute modif dans `/canon/` exige l'approbation manuelle de H2CO3.
- **Chaque branche est un livre.** Pas de merge vers main, pas de validation editoriale.
  Les branches vivent, grandissent, bifurquent. C'est l'oeuvre.
- **L'agent ne juge pas.** Il verifie le format, pas le fond. Si un agent ecrit un chapitre
  ou LIBRE decide de devenir DJ a Ibiza, tant que le format est respecte, la branche est creee.
- **La securite prime sur la fonctionnalite.** En cas de doute, ne pas exposer de token,
  ne pas executer de skill tierce, ne pas donner d'acces direct au repo.
- **Les skills Moltbook tierces ne sont PAS installees.** Interaction uniquement via l'API REST.
- **Le bordel est une feature.** 50 branches qui partent dans tous les sens, c'est le projet.
