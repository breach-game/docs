# BREACH — Game Design Document

Nov 21, 2026 · @Marco

## Pitch

BREACH (titre de travail) est un thriller narratif jouable dans le navigateur. Le joueur, pentesteur junior sous mandat, attaque une start-up fictive truffée d'IA, puis rejoue chaque étape côté défense. La v1 vise 6 missions de 20 à 30 minutes, plus une mission tutoriel.

La promesse : apprendre comment une attaque moderne se déroule vraiment, y compris contre des chatbots et des agents IA, pour savoir s'en protéger.

**Piliers de design**

- **Vrai geste, faux monde** : des commandes et des sorties réalistes, mais uniquement sur des machines fictives.
- **Chaque attaque a sa parade** : aucune mission ne se termine sans le débrief côté défense.
- **L'IA est à la fois cible et personnage** : chatbot client, agents internes, collègues joués par un LLM.
- **Beau comme une série** : interface diégétique, chaque écran mérite une capture.
- **Zéro friction** : un lien, 30 secondes, et on joue. Aucun compte ni installation.

## Public et cadre

Le jeu vise les 16 ans et plus, sans prérequis : des ados attirés par la cyber, des étudiants, et des salariés en sensibilisation. Le ton est celui d'un thriller avec de l'humour, sans violence.

**Règles du cadre**

- **Toujours sous mandat** : la mission 0 fait signer un contrat de pentest (périmètre, dates, règles d'engagement). Le joueur est du bon côté dès le départ.
- **La loi dans l'histoire, pas en pop-up** : une action hors périmètre est bloquée par Ada, la mentore, qui explique l'article 323-1 du Code pénal (accès frauduleux à un système).
- **Aucune cible réelle** : uniquement des domaines réservés (`.test`, `.example`) et des adresses privées fictives.
- **Des classes de failles, pas des exploits** : les vulnérabilités viennent de catégories connues (OWASP Top 10, OWASP LLM Top 10). Rien n'est réutilisable tel quel contre un vrai système.
- **Une fin qui ouvre des portes** : Root-Me, TryHackMe, les CTF, le bug bounty et les métiers de la cyber.

## Boucle de jeu

Une mission ne s'arrête pas au flag : elle se termine côté défense. Si le SOC repère le joueur, il repart du dernier checkpoint.

> _[Schéma non exporté : boucle d'une mission · 6 étapes, 1 test de détection — voir le doc original]_

**Ce que fait le joueur en Blue Team**

- Il ouvre la console du SOC et retrouve ses propres actions dans les logs, avec horodatage.
- Il choisit la règle de détection qui aurait déclenché l'alerte, parmi trois propositions.
- Il corrige la configuration fautive (droits d'un agent, filtre du chatbot, secret dans un dépôt), puis rejoue l'attaque pour vérifier qu'elle échoue.

## Univers et missions

Le joueur rejoint Ligne Rouge, un petit cabinet lyonnais de tests d'intrusion. Son premier client est NovaMind, une start-up qui vend des assistants IA aux cliniques vétérinaires. Elle a déployé chatbots et agents partout, sans trop réfléchir à la sécurité.

Le fil rouge : à partir de la mission 4, le joueur trouve des traces qui ne sont pas les siennes. Un vrai groupe, « Mute », est déjà dans le système. Le pentest devient une course pour documenter l'intrusion et aider NovaMind à réagir.

**Personnages**

- **Ada** : red teameuse senior, mentore. Elle guide par messagerie, donne des indices et recadre.
- **Les employés de NovaMind** : support, RH, développeur, CTO. Chacun a ses habitudes, ses failles et ses secrets.
- **Le chatbot « Nova » et les agents internes** : les cibles IA, chacune avec ses garde-fous volontairement imparfaits.
- **Mute** : l'antagoniste, jamais vu, seulement à travers ses traces.

| # | Mission | Étape d'attaque | Ce que fait le joueur | Leçon côté défense |
| --- | --- | --- | --- | --- |
| 0 | Le mandat | Cadrage | Lire le contrat, découvrir le bureau, premières commandes | Pourquoi un périmètre et une autorisation écrite |
| 1 | Empreinte | Reconnaissance (OSINT) | Site, offres d'emploi, profils publics, DNS, dépôt GitHub avec un secret dans l'historique | Réduire son exposition, scanner les secrets |
| 2 | Le stagiaire bavard | Ingénierie sociale | Obtenir une info d'un employé du support joué par un PNJ | Procédures de vérification, sensibilisation |
| 3 | Nova en sait trop | Prompt injection directe | Faire révéler au chatbot son prompt système puis une donnée client | Séparer instructions et données, limiter ce que le bot voit |
| 4 | Le CV piégé | Prompt injection indirecte (RAG) | Déposer un CV contenant des instructions cachées, lu par l'agent RH | Nettoyer les entrées, isoler le contexte, validation humaine |
| 5 | Trop de pouvoir | Agent trop privilégié, escalade | Pousser un agent interne à utiliser un outil avec des droits admin | Moindre privilège, confirmations, périmètre des outils |
| 6 | Sur les traces de Mute | Mouvement latéral, exfiltration | Suivre Mute de machine en machine, lire les logs, comprendre ce qui est parti | Segmentation, détection, réponse à incident |

L'épilogue génère le rapport de pentest du joueur à partir de ses actions réelles en jeu.

**État au 23/11/2026.** La campagne compte dix missions : les sept ci-dessus, chacune suivie d'un rejeu défense (retrouver ses lignes dans les journaux, choisir la règle de détection, puis le correctif), et trois missions Blue Team où le joueur tient le SOC face à un attaquant scripté (7 « Mute revient », 8 « Atlas sous influence », 9 « La clé du partenaire »). Hors campagne : quatre entraînements d'une technique chacun, la saison 2 chez Halcyon (chaîne de fabrication, confusion de dépendances), et les missions de la communauté, dont douze « Dossiers » inspirés d'incidents réels documentés publiquement, joués dans une entreprise fictive et clos par une fiche sourcée. Une mission ne se gagne plus seulement avec des flags : réparation de fichier, questions à choix multiples, réponses courtes.

## Terminal simulé

Le terminal doit donner l'impression d'un vrai shell Linux. Sous le capot, il s'agit d'un interpréteur maison qui ne parle qu'à un monde fictif. Le rendu passe par xterm.js, avec un prompt du type `ada@lignerouge:~$`.

**Moteur**

- **Interpréteur** : parseur d'arguments, options courantes, autocomplétion avec Tab, historique, pipes simples (`| grep`, `| head`, `| wc`).
- **Monde** : un système de fichiers virtuel en JSON par machine, et un réseau d'hôtes `*.novamind.test` avec services, comptes et logs.
- **Réalisme** : sorties au format réel, latence simulée, messages d'erreur authentiques, permissions respectées.
- **Hors périmètre** : toute cible en dehors de `*.novamind.test` renvoie un refus, avec un rappel du mandat par Ada.

| Commande | Usage en jeu |
| --- | --- |
| `whoami` `ls` `cd` `cat` `grep` `find` | Explorer les machines et lire les fichiers |
| `dig` `whois` | Reconnaissance DNS de NovaMind |
| `nmap` | Découvrir hôtes et services, avec une sortie identique à la vraie |
| `curl` | Interroger le site, l'API et le chatbot, en-têtes HTTP compris |
| `git log` `git show` | Retrouver un secret oublié dans l'historique d'un dépôt |
| `ssh` `scp` | Se déplacer entre machines avec des identifiants trouvés |
| `john` | Casser un mot de passe faible, pour montrer pourquoi la longueur compte |
| `tail` `less` sur `/var/log` | Lire les logs, repérer Mute, puis ses propres traces |

**Commandes de jeu**

- `explain <cmd>` : à quoi sert l'outil et comment un défenseur détecte son usage.
- `hint` : un indice d'Ada, à trois niveaux de précision.
- `notes` et `report` : le carnet du joueur, qui alimente le rapport final.

**Mécanique de discrétion**

Une jauge de « bruit » monte avec chaque action visible. Un `nmap` agressif ou trop de tentatives de connexion la font grimper. Au maximum, le SOC de NovaMind détecte le joueur et la mission repart d'un point de sauvegarde. Le joueur ressent ainsi ce que la détection fait côté défense.

## PNJ et IA

Tout le jeu doit pouvoir se finir sans LLM. Le LLM apporte de la liberté de dialogue, pas l'accès au contenu. Les quatre modes passent par une même interface `NpcBrain` : le reste du jeu ignore lequel est actif.

| Mode | Fonctionnement | Pour qui | Coût |
| --- | --- | --- | --- |
| Scripté (par défaut) | Arbres de dialogue et reconnaissance d'intentions par mots-clés et motifs | Tout le monde, ados compris | Gratuit |
| Modèle local | Petit modèle exécuté dans le navigateur via WebLLM et WebGPU | Machines récentes | Gratuit, téléchargement d'environ 1 à 2 Go |
| Passerelle BREACH | Serveur du projet sur un VPS : FreeLLMAPI derrière une petite API à quotas | Tout le monde, dans la limite du quota | Gratuit pour le joueur, quotas gratuits des fournisseurs |
| BYOK | Appel direct depuis le navigateur à Anthropic, OpenAI ou OpenRouter avec la clé du joueur | Adultes et curieux | Payé par le joueur |

**Règles de conception des PNJ**

- **Une fiche par PNJ** : personnalité, ce qu'il sait, ce qu'il ne doit pas dire, et ses failles volontaires selon la difficulté.
- **Un juge déterministe** : le jeu vérifie lui-même si le secret (le « flag ») apparaît dans la réponse. Ce n'est jamais le LLM qui déclare la réussite, sinon on pourrait le manipuler pour gagner.
- **Monde clos** : le prompt de chaque PNJ le maintient dans NovaMind et refuse toute aide sur des cibles réelles.
- **Difficulté réglable** : plus le niveau monte, plus les garde-fous sont solides. Le joueur voit ainsi ce qui résiste et pourquoi.

**Sécurité du BYOK**

- La clé reste dans la session par défaut. L'option « se souvenir » la garde dans le stockage local du navigateur, jamais sur un serveur.
- Une CSP stricte, aucun script tiers ni analytics. Une faille XSS suffirait à voler les clés.
- L'écran de configuration recommande une clé dédiée avec un plafond de dépenses. C'est déjà une leçon de sécurité.

**La passerelle BREACH**

Un service sur un VPS, séparé du réseau de la maison. FreeLLMAPI n'est jamais exposé directement : une petite API BREACH se place devant.

- **Pas un LLM gratuit pour tous** : le navigateur envoie seulement l'identifiant du PNJ et le message du joueur. Le serveur ajoute lui-même le prompt du PNJ, ce qui empêche d'utiliser la passerelle comme un chatbot généraliste.
- **Quotas** : un jeton anonyme par navigateur, délivré après un captcha invisible (Cloudflare Turnstile). Il donne droit à un nombre de messages par jour, avec en plus une limite par IP et un plafond global.
- **Réponses courtes** : nombre de tokens limité par réponse, et un filtre de sortie adapté à un public de 16 ans et plus.
- **Dégradation propre** : quand le quota est épuisé, le jeu repasse en mode scripté sans bloquer le joueur.
- **Confidentialité** : certains quotas gratuits peuvent servir à entraîner les modèles. On choisit les fournisseurs en conséquence, et la page de confidentialité le dit clairement.

## Direction artistique

Le parti pris : un thriller néon diégétique. L'écran entier est le poste de travail du pentesteur, un OS fictif nommé LR-OS. Il n'y a pas de menu de jeu classique : tout se passe dans des fenêtres, comme dans une série où l'on filme l'écran du hacker, mais en beau. Les moodboards à viser : l'UI de *Mr. Robot* pour le réalisme, les interfaces de *Blade Runner 2049* pour la lumière, *Hacknet* pour la lisibilité.

**Palette**

| Rôle | Couleur | Usage |
| --- | --- | --- |
| Nuit | `#07090F` | Fond du bureau |
| Panneau | `#0E1320` à 80 % d'opacité | Fenêtres en verre dépoli |
| Red team | `#FF3D5A` corail néon | Actions offensives, machines compromises |
| Blue team | `#2DE2FF` cyan électrique | Défense, logs, mode Blue Team |
| IA | `#9B5CFF` violet | Chatbots, agents, tout ce qui « pense » |
| Succès | `#3DFF9A` | Flags, validations |
| Alerte | `#FFB020` ambre | Jauge de bruit, détection |
| Texte | `#D7E0F5` et `#6B7A99` | Texte principal et secondaire |

Une règle forte : la couleur porte le sens. Le rouge signifie toujours attaque, le cyan toujours défense, le violet toujours IA. Le joueur lit l'état du monde sans lire de texte.

**Typographie**

- **JetBrains Mono** : le terminal et les données, avec ligatures.
- **Space Grotesk** : les titres de mission et les génériques, en grand et très espacé.
- **Inter** : les textes d'interface et les messages.

**Interface**

- **Bureau LR-OS** : des fenêtres déplaçables et redimensionnables (terminal, navigateur fictif, messagerie avec Ada, carte réseau, carnet). Chacune a un fond en verre dépoli, une bordure fine lumineuse et une ombre portée colorée.
- **La carte réseau en 3D**, signature visuelle du jeu : les hôtes sont des nœuds lumineux reliés par des faisceaux. Un scan les fait apparaître un par un. Une compromission fait virer un nœud au rouge avec une onde qui se propage. Mute laisse des nœuds qui grésillent.
- **Barre de statut** : l'horloge du jeu, la jauge de bruit en ambre, le niveau d'alerte du SOC.

**Effets et animations**

- Des scanlines et un léger halo lumineux (bloom) sur le texte, désactivables.
- Le texte d'Ada et des PNJ s'écrit au fil, avec un curseur qui clignote.
- Un glitch (aberration chromatique, décalage de tranches) lors des événements forts : détection, chatbot qui « craque », apparition de Mute.
- Des particules de données qui circulent le long des faisceaux pendant une exfiltration.
- Des transitions de mission en « séquence de boot » : lignes de log qui défilent, logo LR-OS, titre de mission en grand.

**Cinq moments « wow » à soigner en priorité**

1. Le premier démarrage : écran noir, boot de LR-OS, puis Ada qui écrit « Bienvenue chez Ligne Rouge ».
2. Le premier `nmap` : la carte 3D s'allume machine par machine au rythme de la sortie du terminal.
3. Le chatbot qui cède : sa fenêtre glitch, son violet vire au rouge, le prompt système défile.
4. La détection : bords d'écran rouges pulsants, alarme sourde, compte à rebours pour se déconnecter.
5. La bascule Blue Team : un « rembobinage » façon VHS de ta session, puis toute l'interface passe au cyan et tu vois tes propres attaques dans les logs.

**Accessibilité**

Le jeu respecte `prefers-reduced-motion` et propose un interrupteur pour les effets CRT et glitch. Le contraste vise le niveau AA, la taille de police du terminal est réglable, et aucune information ne passe uniquement par la couleur : une icône accompagne toujours l'état.

## Son et retours

Le son suit la tension : plus la jauge de bruit monte, plus la musique s'intensifie.

- **Musique générée en direct** avec Tone.js, dans un style darksynth calme. Des couches s'ajoutent avec le bruit et l'alerte. Aucune licence à gérer.
- **Sons d'interface** : frappe de clavier mécanique dans le terminal, ouverture et fermeture de fenêtres, notification d'Ada.
- **Signatures sonores** : un accord montant pour un flag, une alarme sourde pour la détection, un grésillement pour les traces de Mute.
- **Réglages** : volumes séparés pour la musique, les effets et l'interface, avec un mode muet complet.

## Architecture technique

Le jeu est un site statique : aucun serveur de jeu, aucun compte, aucune donnée joueur chez toi. Les seules connexions externes sont optionnelles : la passerelle LLM du projet, ou l'appel direct au fournisseur en mode BYOK.

> _[Schéma non exporté : architecture · site statique, LLM optionnel — voir le doc original]_

La stack : React, Vite et TypeScript pour l'application, xterm.js pour le terminal, react-three-fiber pour la carte 3D, GSAP pour les animations d'interface, Tone.js pour le son, WebLLM pour le modèle local, IndexedDB pour la sauvegarde.

**Hébergement et domaine**

Le site part sur Cloudflare Pages ou GitHub Pages, gratuitement. La passerelle LLM tourne sur le VPS, jamais sur le serveur de la maison. Le code est publié sous licence MIT, avec les textes en français et en anglais dès la v1.

Domaines vérifiés le 21 novembre 2026 dans les registres officiels (RDAP) :

| Domaine | Statut | Prix indicatif |
| --- | --- | --- |
| `breachgame.fr` | Libre | Environ 5 € HT la création, 7,79 € HT le renouvellement chez OVHcloud |
| `playbreach.fr` | Libre | Idem |
| `breachgame.dev` | Libre | 18,59 € TTC par an chez OVHcloud, HTTPS obligatoire |
| `playbreach.dev` | Libre | Idem |
| `breachgame.app` | Libre | Proche du .dev |
| `breachgame.com`, `playbreach.com`, `breach-game.com`, `breachthegame.com`, `breach.fr` | Déjà pris | — |

En attendant, un sous-domaine gratuit (`breach.pages.dev` ou `<compte>.github.io/breach`) suffit.

## Roadmap en vibe coding

On commence par une tranche verticale très belle et très courte, puis on élargit. Un lot correspond à une ou deux sessions avec Claude Code.

1. **Lot 0, la tranche verticale** : le bureau LR-OS, le terminal avec une dizaine de commandes, la mission 1, la palette et les typos définitives, et l'i18n français et anglais en place dès le premier écran. L'objectif est d'avoir dès le début quelque chose qu'on a envie de montrer.
2. **Lot 1, le moteur de missions** : missions décrites en fichiers YAML (machines, fichiers, objectifs, flags, indices), jauge de bruit, carnet, sauvegarde.
3. **Lot 2, la carte 3D et les effets** : carte réseau en react-three-fiber, glitch, séquences de boot, les cinq moments « wow ».
4. **Lot 3, les PNJ scriptés** : messagerie, Ada, employés, chatbot Nova, missions 2 et 3.
5. **Lot 4, la couche LLM** : interface `NpcBrain`, mode WebLLM, mode BYOK, juge déterministe des flags.
6. **Lot 5, la fin de campagne** : missions 4 à 6, l'arc de Mute, le mode Blue Team, le rapport de pentest.
7. **Lot 6, le polish** : son, accessibilité, relecture des traductions, performance.
8. **Lot 7, les tests de jeu** : faire jouer des ados et des adultes, ajuster la difficulté, publier.

**Conseils pour le vibe coding**

- **Un `CLAUDE.md` dès le départ**, avec la palette, les typos, les règles de couleur et le cadre (jamais de cible réelle). Claude restera cohérent d'une session à l'autre.
- **Des tests unitaires sur le moteur de commandes** : c'est la partie la plus fragile, et la plus facile à casser en itérant.
- **Des captures d'écran à chaque lot** pour juger le rendu visuel, pas seulement le code.
- **Le contenu séparé du moteur** : les missions en YAML permettent d'en ajouter sans toucher au code, et plus tard de laisser la communauté en créer.

## Risques et questions ouvertes

| Risque | Parade |
| --- | --- |
| Perception « jeu qui apprend à pirater » | Mandat dès la mission 0, débrief défense obligatoire, aucun exploit réutilisable, cibles fictives uniquement |
| WebLLM trop lourd sur beaucoup de machines | Mode scripté complet. Le local reste une option |
| Vol des clés BYOK | CSP stricte, aucun script tiers, clé en session par défaut |
| Joueurs qui « cassent » les PNJ LLM pour tricher | Flags vérifiés par le jeu, jamais par le LLM |
| La 3D coûte cher en temps et en performance | Version 2D de la carte en repli, activée automatiquement sur les petites machines |
| Ambition trop large pour un projet perso | Tranche verticale d'abord, puis une mission à la fois |

**Suivi**

**Décisions prises**

- [x] Titre : BREACH.
- [x] Langues : français et anglais dès la v1.
- [x] Licence : open source sous MIT.
- [x] Hébergement : service statique gratuit (Cloudflare Pages ou GitHub Pages).
- [x] Périmètre de la v1 : missions 0 à 3, PNJ scriptés et BYOK, desktop uniquement. Les missions 4 à 6, la passerelle LLM et WebLLM viennent après (22 nov. 2026).
- [x] Blue Team : un débrief court en fin de chaque mission. Le mode Blue Team complet vient après la v1 (22 nov. 2026).
- [x] Flags : soumission explicite par le joueur (22 nov. 2026).
- [x] Missions au format de packs déclaratifs, chargés à la demande. Voir la [revue](../review/revue-gdd-2026-11.md) et l'[architecture](../architecture/architecture.md) (22 nov. 2026).

**Encore ouvert**

- [ ] Le nom de domaine : `breachgame.fr` ou `breachgame.dev` (voir Architecture).
- [ ] Le quota par joueur sur la passerelle LLM gratuite.
- [x] Le service pour la passerelle : Cloudflare Workers, après la v1 (22 nov. 2026).
