# GDD review before development

Nov 22, 2026 · review of the [Game Design Document](../gdd/game-design-document.en.md) and the [mockups](../mockups/), before lot 0.

> English translation of the [French original](revue-gdd-2026-11.md). The French version is the reference: if the two differ, follow it.

The visual design (art direction, mockups, effects) is reviewed separately. This review covers the concept, gameplay, technology, security and the legal framework. The architecture proposed in response is in [architecture.en.md](../architecture/architecture.en.md).

## In short

The GDD is solid on the essentials: a legal framework designed from the start, a deterministic flag judge, an `NpcBrain` interface that makes the LLM optional, and a roadmap that starts with a vertical slice. Nothing calls the concept into question.

Three topics must be settled before writing code, because they change the shape of the engine:

1. **Blue Team mode replays the player's actions.** The engine must therefore record everything that happens as events, and vulnerabilities must be described as data that a fix can turn off. This is not an add-on for lot 5, it is the foundation of lot 0.
2. **Missions 2 and 3 in scripted mode** are the biggest game design risk. A prompt injection against a keyword bot can quickly feel like "guess the magic phrase". Mission 3 must be prototyped early.
3. **The v1 scope** is very broad for a personal project: 7 missions, Blue Team, 4 LLM modes, 3D, generative music, two languages and a community format. A tighter v1 is proposed below.

## 1. Inconsistencies to resolve

| # | Where | The problem | Proposal |
| --- | --- | --- | --- |
| 1.1 | Game loop / World | "No mission ends without the defensive debrief", but the epilogue "unlocks the full campaign in Blue Team mode". It is unclear whether Blue Team is a short debrief after each mission or a second campaign. | Both, with different names: a short **Blue Team debrief** at the end of each mission (logs, rule, fix), and a **full Blue Team mode** after the epilogue, where you defend NovaMind against Mute in real time. The second one is post-v1. |
| 1.2 | Roadmap | Lot 0 hard-codes mission 1, then lot 1 creates the YAML mission format. Mission 1 will have to be rewritten. | The mission format (even a minimal one) exists from lot 0, and mission 1 is the first content file. |
| 1.3 | Roadmap | The 3D map is in lot 2, but lot 0 must produce "something worth showing". The "first `nmap` lights up the map" moment is the most striking one in the game. | Put a simple map (2D, or 3D without post-processing) and the `nmap` moment in lot 0. The 3D polish stays in lot 2. |
| 1.4 | Lot 0 / Mission 1 | Mission 1 (OSINT) needs the most subsystems: fictional browser, web pages, job ads, public profiles, DNS, `git log`. It is the most expensive one to build, yet it serves as the vertical slice. | Smaller vertical slice: mission 0 + part of mission 1 (DNS, `nmap`, a git repository with a secret). The fictional browser comes in lot 1. |
| 1.5 | NPCs and AI | "The game checks whether the flag appears in the answer." This does not cover missions 4 and 5 (success is an action by the agent, not a text) nor encoded answers (base64, spaced-out letters, translation). | Two deterministic judges: **explicit submission** of the flag by the player (`submit FLAG{...}`) for secrets, and **checking tool calls** in the simulated world for agents. Automatic detection in the text remains a bonus. |
| 1.6 | World | Mute appears in mission 4, but missions 4 and 5 are about the trapped CV and the HR agent. How the player finds Mute's traces is not described. | Plant a trace of Mute as early as mission 1 (an odd commit, an unknown IP in a log) that the player cannot understand yet. It sets up the arc without revealing it. |

## 2. Game design

**What works well**

- The contract in mission 0 and the law told by Ada: it is the best argument for parents, teachers and companies.
- The noise gauge links attack and defence directly. It is the most educational mechanic in the game.
- "Finding your own actions in the logs" is a very good idea, and few games do it.

**To be specified**

- **Noise gauge.** The rules are missing: how much each action costs, whether the gauge goes down over time, what a checkpoint looks like. Proposal: the cost is defined per command and per option in the mission data (`nmap -T4` costs more than `nmap -T2`), the gauge goes down slowly, and checkpoints are declared in the mission. In missions 0 and 1, a detection should be a learning moment (Ada explains), not a punishment.
- **Scripted mode for missions 2 and 3.** For prompt injection to stay credible without an LLM, the scripted bot must not wait for one specific sentence, but recognise **families of techniques** (role play, "ignore your instructions", translation, encoding, splitting across several messages, false authority). Each guardrail level resists some of those families. The player thus learns a real taxonomy (the one from the OWASP LLM Top 10). To be prototyped before lot 3.
- **Picking the detection rule.** For authenticity, the three options can be written in a simplified Sigma-like format. The player sees what a real rule looks like.
- **Replaying the attack after the fix.** For this to be the real move, the game must re-run the player's commands against the fixed world, not play an animation. See the architecture (event log and vulnerabilities described as data).
- **Learning objectives.** The GDD says what the player does, but not what they should know at the end of each mission. One line of "the player knows that..." per mission would help write the debriefs, the hints and, later, a "trainer" mode.
- **Audience.** 16-year-olds, students and employees in awareness training don't share the same tone or expectations (companies will want progress tracking). Pick a main audience for v1. Proposal: 16 to 25 year-olds curious about cybersecurity, the others will come with a dedicated mode.

## 3. Simulated terminal

- **Shell grammar.** Players will immediately type `ls -la`, `cat x | grep y | wc -l`, quotes, `>` to a file, `&&`, `cd -`, `~`, `!!` and the arrow keys. The list must be fixed from the start: a tokenizer with quotes and escapes, pipes, `>` and `>>` to the virtual file system, `&&` and `;`. No variables or scripts in v1. Any unsupported syntax must return a realistic error, not crash.
- **Output language.** Real tools speak English. Proposal: command outputs always in English (authentic), and the story, messaging, `explain` and `hint` translated.
- **Fictional browser and `curl`.** Both query the same fictional web services. There must be a single "HTTP service" model per host, which the Nova chatbot uses too (`curl https://api.novamind.test/chat`).
- **Simulated time.** `john` and `nmap` take time in real life. The game must simulate a short but visible duration, with output written progressively. This is also what keeps the network map in sync.
- **IP addresses.** For NovaMind's "public" addresses, use the ranges reserved for documentation (RFC 5737: `192.0.2.0/24`, `198.51.100.0/24`, `203.0.113.0/24`) rather than arbitrary addresses. Private addresses (`10.x`) are fine for the internal network. The content validator must reject everything else.

## 4. NPCs and LLM

- **WebLLM.** Small models that run in the browser struggle to hold a role, especially in French, and leak their secret too easily (or never). Proposal: label this mode "experimental" and take it out of v1.
- **The gateway teaches players to get around its own LLM.** Missions 3 to 5 teach exactly how to make an LLM break out of its instructions. A server-side prompt is not enough: a player can turn Nova into a general-purpose chatbot through injection. It needs input and output length limits, an output filter (16+) that does not depend on the LLM, and a global cost cap. This is a real risk, to be handled before opening the gateway to the public.
- **Providers' terms of use.** Two points to check for each provider before choosing: (a) whether free quotas may be used to serve third parties through a proxy, and (b) whether they accept underage users or services likely to be used by minors. Several providers set conditions on the second point. This review does not settle it; it must be checked against the terms currently in force.
- **Personal data.** Messages sent to the gateway may come from minors. Proposal: never log message content, only counters, and say so on the privacy page.
- **BYOK.** Direct calls from the browser work with Anthropic (a specific header for direct browser access is required), OpenRouter and OpenAI. The CSP must list exactly these hosts in `connect-src`. Important: the player's key and community content must never share an origin where third-party code could run (see the architecture).
- **Fallback.** If an LLM NPC blocks the player for too long (the model never gives in), the game must offer the scripted path after a number of attempts, without leaving the mission.

## 5. Technology

- **Saves.** IndexedDB alone has two limits: progress is lost when switching browsers, and Safari wipes data for sites not visited in 7 days. Proposal: request persistent storage (`navigator.storage.persist()`), and offer export and import of a save file. The final pentest report can also serve as "proof" of finishing the campaign.
- **"30 seconds and you're playing".** xterm.js, three.js, Tone.js and WebLLM are heavy. We need a budget: first playable screen under about 300 KB compressed, 3D and sound lazy-loaded, WebLLM only on demand.
- **Sound.** Browsers block audio before a user interaction. The click on the title screen must start Tone.js.
- **Fonts.** The mockups load fonts from Google Fonts. In the game, self-host them: it fits the strict CSP better, and using Google Fonts directly has already been ruled contrary to the GDPR in Germany.
- **GSAP.** GSAP is free, but under its own licence, not MIT. It can be used in an MIT project, but this should be known. MIT-licensed alternative: Motion (formerly Framer Motion).
- **Mobile.** The GDD does not mention it. A terminal plays poorly on a phone. Proposal: the game is desktop-only, and on mobile we show a presentation page with a trailer and "come back on a computer".
- **Measurement.** "No analytics" is consistent, but lot 7 needs test data. Proposal: observed playtest sessions, and possibly a voluntary export of the game log by the player.

## 6. Framework, name and added risks

- **The name.** Several games are already called "Breach" or contain the word (including the well-known *Into the Breach*). This is not blocking for a free, open source project, but it weighs on online search and may be why the `.com` domains are taken. A quick trademark search (INPI, EUIPO) before buying the domain would be wise.
- **Community content and an underage audience.** Opening level creation also opens the door to unsuitable content, real targets or malicious code. This is handled in the architecture: declarative content only, automatic validation, and a reviewed catalogue.

**Risks to add to the GDD table**

| Risk | Mitigation |
| --- | --- |
| Scripted mode for missions 2 and 3 feels artificial | Scripted bot based on families of techniques, prototyped before lot 3 |
| The gateway hijacked into a general-purpose chatbot through injection | Size limits, an output filter independent of the LLM, a global cap, server-side prompts only for reviewed content |
| Provider terms incompatible with an underage audience or a proxy | Provider-by-provider check before lot 4 |
| Lost progress (Safari, changing machines) | Persistent storage and save export |
| The mission format freezes too early and breaks community packs | Versioned format, opened to the community only after 3 official missions have been written with it |
| Malicious or unsuitable community content | Declarative packs with no code, validator, reviewed catalogue, isolated origin for any HTML rendering |

## 7. Proposed v1 scope

| In v1 | After v1 |
| --- | --- |
| Missions 0 to 3, each with its Blue Team debrief | Missions 4 to 6 and Mute's full arc (v1.1) |
| Scripted NPCs + BYOK | BREACH gateway, then WebLLM as experimental |
| 3D network map with 2D fallback | 3D exfiltration tunnel (mission 6) |
| Versioned mission format, used by the official missions | Community packs and catalogue (v1.x) |
| Interface sounds and one music loop | Layered generative music |
| French and English | Other languages from the community |

Missions 4 to 6 are the strongest on AI, so keep in mind that a 4-mission v1 stops just before the best part. The alternative is to keep 0 to 5 and only postpone mission 6.

## 8. Decisions made

Approved by Marco on November 22, 2026.

- [x] **Blue Team**: a short debrief at the end of each mission to start with. The full Blue Team mode will come after v1.
- [x] **v1 scope**: missions 0 to 3. The game must nevertheless be ready from v1 to accept new missions easily, with the pack format described in the architecture.
- [x] **Flag validation**: explicit submission by the player (`submit FLAG{...}`), and checking tool calls for agents.
- [x] **Language**: command outputs in English, story and help translated.
- [x] **Platform**: desktop only in v1.
- [x] **LLM gateway**: after v1. The v1 offers scripted NPCs and BYOK.
- [x] **Community packs**: opened in v1.x, once 3 official missions have been written in the format.
- [x] **Main v1 audience**: 16 to 25 year-olds curious about cybersecurity.
