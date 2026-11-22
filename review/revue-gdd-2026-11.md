# Revue du GDD avant développement

Nov 22, 2026 · revue du [Game Design Document](../gdd/game-design-document.md) et des [maquettes](../mockups/), avant le lot 0.

Le design visuel (DA, maquettes, effets) est revu à part. Cette revue couvre le concept, le gameplay, la technique, la sécurité et le cadre légal. L'architecture proposée en réponse est dans [architecture.md](../architecture/architecture.md).

## En bref

Le GDD est solide sur l'essentiel : un cadre légal pensé dès le départ, un juge déterministe des flags, une interface `NpcBrain` qui rend le LLM optionnel, et une roadmap qui commence par une tranche verticale. Rien ne remet le concept en cause.

Trois sujets doivent être tranchés avant d'écrire du code, parce qu'ils changent la forme du moteur :

1. **Le mode Blue Team rejoue les actions du joueur.** Il faut donc que le moteur enregistre tout ce qui se passe sous forme d'événements, et que les failles soient décrites comme des données qu'un correctif peut désactiver. Ce n'est pas un ajout du lot 5, c'est la fondation du lot 0.
2. **Les missions 2 et 3 en mode scripté** sont le plus gros risque de game design. Une injection de prompt contre un bot à mots-clés peut vite ressembler à « devine la phrase magique ». Il faut prototyper la mission 3 tôt.
3. **Le périmètre de la v1** est très large pour un projet perso : 7 missions, Blue Team, 4 modes LLM, 3D, musique générative, deux langues et un format communautaire. Une v1 resserrée est proposée plus bas.

## 1. Incohérences à lever

| # | Où | Le problème | Proposition |
| --- | --- | --- | --- |
| 1.1 | Boucle de jeu / Univers | « Aucune mission ne se termine sans le débrief côté défense », mais l'épilogue « débloque la campagne complète en mode Blue Team ». On ne sait pas si la Blue Team est un débrief court à chaque mission ou une seconde campagne. | Les deux, mais nommés différemment : un **débrief Blue Team** court en fin de chaque mission (logs, règle, correctif), et un **mode Blue Team complet** après l'épilogue, où l'on défend NovaMind contre Mute en temps réel. Le second est post-v1. |
| 1.2 | Roadmap | Le lot 0 code la mission 1 « en dur », le lot 1 crée ensuite le format YAML des missions. La mission 1 sera à réécrire. | Le format de mission (même minimal) existe dès le lot 0, et la mission 1 est le premier fichier de contenu. |
| 1.3 | Roadmap | La carte 3D est au lot 2, mais le lot 0 doit produire « quelque chose qu'on a envie de montrer ». Le moment « premier `nmap` qui allume la carte » est le plus démonstratif du jeu. | Mettre une carte simple (2D ou 3D sans post-traitement) et le moment `nmap` dans le lot 0. Le polish 3D reste au lot 2. |
| 1.4 | Lot 0 / Mission 1 | La mission 1 (OSINT) demande le plus de sous-systèmes : navigateur fictif, pages web, offres d'emploi, profils publics, DNS, `git log`. C'est la plus chère à produire alors qu'elle sert de tranche verticale. | Tranche verticale réduite : mission 0 + un morceau de mission 1 (DNS, `nmap`, un dépôt git avec un secret). Le navigateur fictif arrive au lot 1. |
| 1.5 | PNJ et IA | « Le jeu vérifie si le flag apparaît dans la réponse. » Cela ne couvre pas les missions 4 et 5 (le succès est une action de l'agent, pas un texte) ni les réponses encodées (base64, lettres espacées, traduction). | Deux juges déterministes : **soumission explicite** du flag par le joueur (`submit FLAG{...}`) pour les secrets, et **vérification des appels d'outils** dans le monde simulé pour les agents. La détection automatique dans le texte reste un bonus. |
| 1.6 | Univers | Mute apparaît à la mission 4, mais les missions 4 et 5 parlent du CV piégé et de l'agent RH. Comment le joueur trouve les traces de Mute n'est pas décrit. | Prévoir une trace de Mute dès la mission 1 (un commit étrange, une IP inconnue dans un log) que le joueur ne peut pas encore comprendre. Ça installe l'arc sans le révéler. |

## 2. Game design

**Ce qui fonctionne bien**

- Le mandat en mission 0 et la loi racontée par Ada : c'est le meilleur argument face aux parents, aux profs et aux entreprises.
- La jauge de bruit relie directement l'attaque et la défense. C'est la mécanique la plus didactique du jeu.
- « Retrouver ses propres actions dans les logs » est une très bonne idée, et peu de jeux le font.

**À préciser**

- **Jauge de bruit.** Il manque les règles : combien coûte chaque action, est-ce que la jauge redescend avec le temps, à quoi ressemble un checkpoint. Proposition : le coût est défini par commande et par option dans les données de mission (`nmap -T4` coûte plus que `nmap -T2`), la jauge redescend lentement, et les checkpoints sont déclarés dans la mission. En mission 0 et 1, une détection doit être un moment d'apprentissage (Ada explique), pas une punition.
- **Mode scripté des missions 2 et 3.** Pour que l'injection de prompt reste crédible sans LLM, le bot scripté ne doit pas attendre une phrase précise, mais reconnaître des **familles de techniques** (jeu de rôle, « ignore tes instructions », traduction, encodage, découpage en plusieurs messages, fausse autorité). Chaque niveau de garde-fou résiste à certaines familles. Le joueur apprend ainsi une taxonomie réelle (celle de l'OWASP LLM Top 10). C'est à prototyper avant le lot 3.
- **Choix de la règle de détection.** Pour l'authenticité, les trois propositions peuvent être écrites dans un format proche de Sigma, simplifié. Le joueur voit à quoi ressemble une vraie règle.
- **Rejouer l'attaque après correctif.** Pour que ce soit le vrai geste, le jeu doit réexécuter les commandes du joueur contre le monde corrigé, pas jouer une animation. Voir l'architecture (journal d'événements et failles décrites comme des données).
- **Objectifs pédagogiques.** Le GDD dit ce que fait le joueur, mais pas ce qu'il doit savoir à la fin de chaque mission. Une ligne « le joueur sait que... » par mission aiderait à écrire les débriefs, les indices et, plus tard, un mode « formateur ».
- **Public.** Ados de 16 ans, étudiants et salariés en sensibilisation n'ont ni le même ton ni les mêmes attentes (les entreprises voudront un suivi de progression). Choisir un public principal pour la v1. Proposition : les 16 à 25 ans curieux de cyber, les autres viendront avec un mode dédié.

## 3. Terminal simulé

- **Grammaire du shell.** Les joueurs taperont immédiatement `ls -la`, `cat x | grep y | wc -l`, des guillemets, `>` vers un fichier, `&&`, `cd -`, `~`, `!!` et les flèches. Il faut fixer la liste dès le départ : tokeniseur avec guillemets et échappements, pipes, `>` et `>>` vers le système de fichiers virtuel, `&&` et `;`. Pas de variables ni de scripts en v1. Toute syntaxe non supportée doit répondre une erreur réaliste, pas planter.
- **Langue des sorties.** Les vrais outils parlent anglais. Proposition : sorties des commandes toujours en anglais (authentique), et récit, messagerie, `explain` et `hint` traduits.
- **Navigateur fictif et `curl`.** Les deux interrogent les mêmes services web fictifs. Il faut un seul modèle de « service HTTP » par hôte, que le chatbot Nova utilise aussi (`curl https://api.novamind.test/chat`).
- **Temps simulé.** `john` et `nmap` prennent du temps en vrai. Le jeu doit simuler une durée courte mais visible, avec une sortie qui s'écrit au fil. C'est aussi ce qui synchronise la carte réseau.
- **Adresses IP.** Pour les adresses « publiques » de NovaMind, utiliser les plages réservées à la documentation (RFC 5737 : `192.0.2.0/24`, `198.51.100.0/24`, `203.0.113.0/24`) plutôt que des adresses quelconques. Les adresses privées (`10.x`) conviennent au réseau interne. Le validateur de contenu doit refuser tout le reste.

## 4. PNJ et LLM

- **WebLLM.** Les petits modèles exécutables dans le navigateur tiennent mal un rôle, surtout en français, et fuient leur secret trop facilement (ou jamais). Proposition : classer ce mode « expérimental » et le sortir de la v1.
- **La passerelle enseigne à contourner son propre LLM.** Les missions 3 à 5 apprennent justement à faire sortir un LLM de ses consignes. Le prompt ajouté côté serveur ne suffit pas : un joueur peut transformer Nova en chatbot généraliste par injection. Il faut une limite de longueur en entrée et en sortie, un filtre de sortie (16 ans et plus) qui ne dépend pas du LLM, et un plafond global de coût. C'est un risque réel, à traiter avant d'ouvrir la passerelle au public.
- **Conditions d'utilisation des fournisseurs.** Deux points à vérifier pour chaque fournisseur avant de choisir : (a) si les quotas gratuits peuvent être utilisés pour servir des tiers via un proxy, et (b) s'ils acceptent des utilisateurs mineurs ou des services susceptibles d'être utilisés par des mineurs. Plusieurs fournisseurs imposent des conditions sur ce second point. Cette revue ne tranche pas, c'est à vérifier dans les textes en vigueur.
- **Données personnelles.** Les messages envoyés à la passerelle peuvent venir de mineurs. Proposition : ne jamais journaliser le contenu des messages, seulement des compteurs, et l'écrire dans la page de confidentialité.
- **BYOK.** Les appels directs depuis le navigateur fonctionnent avec Anthropic (en-tête spécifique d'accès direct navigateur requis), OpenRouter et OpenAI. La CSP doit lister exactement ces hôtes dans `connect-src`. Point important : la clé du joueur et le contenu communautaire ne doivent jamais partager une origine où du code tiers pourrait s'exécuter (voir l'architecture).
- **Repli.** Si un PNJ LLM bloque le joueur trop longtemps (le modèle ne cède jamais), le jeu doit proposer le chemin scripté après un certain nombre d'essais, sans quitter la mission.

## 5. Technique

- **Sauvegarde.** IndexedDB seul a deux limites : la progression est perdue en changeant de navigateur, et Safari efface les données des sites non visités depuis 7 jours. Proposition : demander le stockage persistant (`navigator.storage.persist()`), et proposer l'export et l'import d'un fichier de sauvegarde. Le rapport de pentest final peut aussi servir de « preuve » de fin de campagne.
- **« 30 secondes et on joue ».** xterm.js, three.js, Tone.js et WebLLM pèsent lourd. Il faut un budget : premier écran jouable sous environ 300 Ko compressés, la 3D et le son chargés en différé, WebLLM uniquement à la demande.
- **Son.** Les navigateurs bloquent l'audio avant une interaction. Le clic de l'écran titre doit démarrer Tone.js.
- **Polices.** Les maquettes chargent les polices depuis Google Fonts. Dans le jeu, les héberger soi-même : c'est plus compatible avec la CSP stricte, et l'usage de Google Fonts en direct a déjà été jugé contraire au RGPD en Allemagne.
- **GSAP.** GSAP est gratuit, mais sous sa propre licence, pas MIT. C'est utilisable dans un projet MIT, mais il faut le savoir. Alternative sous licence MIT : Motion (ex-Framer Motion).
- **Mobile.** Le GDD n'en parle pas. Un terminal se joue mal sur téléphone. Proposition : le jeu est desktop, et sur mobile on affiche une page de présentation avec une bande-annonce et « reviens sur ordinateur ».
- **Mesure.** « Aucun analytics » est cohérent, mais le lot 7 a besoin de données de tests. Proposition : des sessions de test observées, et éventuellement un export volontaire du journal de partie par le joueur.

## 6. Cadre, nom et risques ajoutés

- **Le nom.** Plusieurs jeux s'appellent déjà « Breach » ou contiennent ce mot (dont *Into the Breach*, très connu). Ce n'est pas bloquant pour un projet gratuit et open source, mais ça pèse sur la recherche en ligne et c'est peut-être pourquoi les `.com` sont pris. Une recherche de marque rapide (INPI, EUIPO) avant d'acheter le domaine serait prudente.
- **Contenu communautaire et public mineur.** Ouvrir la création de niveaux, c'est aussi ouvrir la porte à des contenus inadaptés, à de vraies cibles ou à du code malveillant. C'est traité dans l'architecture : contenu déclaratif uniquement, validation automatique, et catalogue relu.

**Risques à ajouter au tableau du GDD**

| Risque | Parade |
| --- | --- |
| Le mode scripté des missions 2 et 3 paraît artificiel | Bot scripté par familles de techniques, prototypé avant le lot 3 |
| La passerelle détournée en chatbot généraliste par injection | Limites de taille, filtre de sortie indépendant du LLM, plafond global, prompts côté serveur uniquement pour le contenu relu |
| Conditions des fournisseurs incompatibles avec un public mineur ou un proxy | Vérification fournisseur par fournisseur avant le lot 4 |
| Progression perdue (Safari, changement de machine) | Stockage persistant et export de sauvegarde |
| Le format de mission se fige trop tôt et casse les packs communautaires | Format versionné, ouvert à la communauté seulement après 3 missions officielles écrites avec |
| Contenu communautaire malveillant ou inadapté | Packs déclaratifs sans code, validateur, catalogue relu, origine isolée pour tout rendu HTML |

## 7. Périmètre proposé pour la v1

| Dans la v1 | Après la v1 |
| --- | --- |
| Missions 0 à 3, chacune avec son débrief Blue Team | Missions 4 à 6 et l'arc complet de Mute (v1.1) |
| PNJ scriptés + BYOK | Passerelle BREACH, puis WebLLM en expérimental |
| Carte réseau 3D avec repli 2D | Tunnel d'exfiltration 3D (mission 6) |
| Format de mission versionné, utilisé par les missions officielles | Packs communautaires et catalogue (v1.x) |
| Son d'interface et une boucle musicale | Musique générative en couches |
| Français et anglais | Autres langues par la communauté |

Les missions 4 à 6 sont les plus fortes sur l'IA, donc il faut garder en tête qu'une v1 à 4 missions s'arrête juste avant le meilleur. L'alternative est de garder 0 à 5 et de reporter seulement la mission 6.

## 8. Décisions prises

Validées par Marco le 22 novembre 2026.

- [x] **Blue Team** : débrief court en fin de chaque mission pour démarrer. Le mode Blue Team complet viendra après la v1.
- [x] **Périmètre v1** : missions 0 à 3. Le jeu doit cependant être prêt dès la v1 à accepter facilement de nouvelles missions, avec le format de packs décrit dans l'architecture.
- [x] **Validation des flags** : soumission explicite par le joueur (`submit FLAG{...}`), et vérification des appels d'outils pour les agents.
- [x] **Langue** : sorties des commandes en anglais, récit et aides traduits.
- [x] **Plateforme** : desktop uniquement en v1.
- [x] **Passerelle LLM** : après la v1. La v1 propose les PNJ scriptés et le BYOK.
- [x] **Packs communautaires** : ouverture en v1.x, une fois 3 missions officielles écrites dans le format.
- [x] **Public principal de la v1** : les 16 à 25 ans curieux de cyber.
