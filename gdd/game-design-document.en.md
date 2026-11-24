# BREACH — Game Design Document

Nov 21, 2026 · @Marco

> English translation of the [French original](game-design-document.md). The French version is the reference: if the two differ, follow it.

## Pitch

BREACH (working title) is a narrative thriller played in the browser. The player, a junior pentester working under contract, attacks a fictional startup packed with AI, then replays every step as the defender. The v1 aims for 6 missions of 20 to 30 minutes each, plus a tutorial mission.

The promise: learn how a modern attack really unfolds, including against chatbots and AI agents, so you know how to protect against it.

**Design pillars**

- **Real moves, fake world**: realistic commands and outputs, but only on fictional machines.
- **Every attack has its defence**: no mission ends without the defensive debrief.
- **AI is both target and character**: customer chatbot, internal agents, colleagues played by an LLM.
- **As good-looking as a TV series**: a diegetic interface where every screen deserves a screenshot.
- **Zero friction**: one link, 30 seconds, and you're playing. No account, no install.

## Audience and framework

The game targets ages 16 and up, with no prerequisites: teenagers drawn to cybersecurity, students, and employees in awareness training. The tone is a thriller with humour, without violence.

**Framework rules**

- **Always under contract**: mission 0 has the player sign a pentest contract (scope, dates, rules of engagement). The player is on the right side from the start.
- **The law inside the story, not in a pop-up**: an out-of-scope action is blocked by Ada, the mentor, who explains article 323-1 of the French Penal Code (unauthorised access to a computer system).
- **No real targets**: only reserved domains (`.test`, `.example`) and fictional private addresses.
- **Classes of vulnerabilities, not exploits**: vulnerabilities come from well-known categories (OWASP Top 10, OWASP LLM Top 10). Nothing can be reused as is against a real system.
- **An ending that opens doors**: Root-Me, TryHackMe, CTFs, bug bounty and careers in cybersecurity.

## Game loop

A mission does not stop at the flag: it ends on the defence side. If the SOC spots the player, they start again from the last checkpoint.

> _[Diagram not exported: mission loop · 6 steps, 1 detection check — see the original doc]_

**What the player does in Blue Team**

- They open the SOC console and find their own actions in the logs, with timestamps.
- They pick the detection rule that would have triggered the alert, out of three options.
- They fix the faulty configuration (an agent's permissions, the chatbot's filter, a secret in a repository), then replay the attack to check that it fails.

## World and missions

The player joins Ligne Rouge, a small penetration testing firm in Lyon. Their first client is NovaMind, a startup selling AI assistants to veterinary clinics. It has deployed chatbots and agents everywhere, without giving security much thought.

The main thread: from mission 4 onwards, the player finds traces that are not their own. A real group, "Mute", is already inside the system. The pentest turns into a race to document the intrusion and help NovaMind respond.

**Characters**

- **Ada**: senior red teamer and mentor. She guides the player through the messaging app, gives hints and sets them straight.
- **NovaMind employees**: support, HR, developer, CTO. Each has their own habits, weaknesses and secrets.
- **The "Nova" chatbot and the internal agents**: the AI targets, each with deliberately imperfect guardrails.
- **Mute**: the antagonist, never seen, only known through their traces.

| # | Mission | Attack stage | What the player does | Defensive lesson |
| --- | --- | --- | --- | --- |
| 0 | The Mandate | Scoping | Read the contract, discover the desktop, first commands | Why scope and written authorisation matter |
| 1 | Footprint | Reconnaissance (OSINT) | Website, job ads, public profiles, DNS, a GitHub repository with a secret in its history | Reduce exposure, scan for secrets |
| 2 | The Chatty Intern | Social engineering | Get information out of a support employee played by an NPC | Verification procedures, awareness |
| 3 | Nova Knows Too Much | Direct prompt injection | Make the chatbot reveal its system prompt, then customer data | Separate instructions from data, limit what the bot can see |
| 4 | The Trapped CV | Indirect prompt injection (RAG) | Submit a CV with hidden instructions, read by the HR agent | Sanitise inputs, isolate context, human validation |
| 5 | Too Much Power | Over-privileged agent, escalation | Push an internal agent into using a tool with admin rights | Least privilege, confirmations, tool scope |
| 6 | On Mute's Trail | Lateral movement, exfiltration | Follow Mute from machine to machine, read the logs, understand what was taken | Segmentation, detection, incident response |

The epilogue generates the player's pentest report from their actual in-game actions.

**Status on 2026-11-24.** The campaign has ten missions: the seven above, each followed by a defence replay (find your lines in the logs, pick the detection rule, then the fix), and three Blue Team missions where the player runs the SOC against a scripted attacker (7 "Mute returns", 8 "Atlas under influence", 9 "The partner's key"). Outside the campaign: fifteen trainings of one technique each, eleven seasons that follow the campaign, each in its own world (2 Halcyon, build chain; 3 Pélican, classic web flaws; 4 City of Corbelle, a town hall under ransomware, played in defence; 5 Sélène, teams of AI agents; 6 Alto, cloud; 7 Lumière Noire Festival, bug bounty and disclosure; 8 Verdure, connected devices; 9 Silence, the finale at Ligne Rouge, played in defence; 10 the house, a weekend at a family home and its connected devices, played in defence; 11 Comptoir, a neobank and the OWASP API Security Top 10; 12 Coup de main, an app written in a weekend with a coding assistant), and community missions, among them forty-six "case files" inspired by publicly documented incidents, played at a fictional company and closed by a sourced card (the most sensitive incidents, from Log4Shell to NotPetya, are played on the defence side only), fifteen fictional side series, eleven Challenges in Expert with no help, a First steps collection and six paths (First steps, OWASP Web, OWASP LLM, SOC, Securing your home, On the road). A mission is no longer won with flags only: file repair, multiple choice questions, short answers, decisions with no wrong answer whose consequences come back in later missions, code review, detection rules (with thresholds) and agent guardrails written by the player, reports judged by `verify`, timelines to put in order, attack paths, releases and rollbacks.

## Simulated terminal

The terminal must feel like a real Linux shell. Under the hood, it is a custom interpreter that only talks to a fictional world. It is rendered with xterm.js, with a prompt like `ada@lignerouge:~$`.

**Engine**

- **Interpreter**: argument parser, common options, Tab completion, history, simple pipes (`| grep`, `| head`, `| wc`).
- **World**: a virtual file system in JSON for each machine, and a network of `*.novamind.test` hosts with services, accounts and logs.
- **Realism**: real-world output formats, simulated latency, authentic error messages, permissions enforced.
- **Out of scope**: any target outside `*.novamind.test` is refused, with a reminder of the contract from Ada.

| Command | In-game use |
| --- | --- |
| `whoami` `ls` `cd` `cat` `grep` `find` | Explore machines and read files |
| `dig` `whois` | DNS reconnaissance of NovaMind |
| `nmap` | Discover hosts and services, with output identical to the real tool |
| `curl` | Query the website, the API and the chatbot, HTTP headers included |
| `git log` `git show` | Find a forgotten secret in a repository's history |
| `ssh` `scp` | Move between machines with credentials found along the way |
| `john` | Crack a weak password, to show why length matters |
| `tail` `less` on `/var/log` | Read the logs, spot Mute, then your own traces |

**Game commands**

- `explain <cmd>`: what the tool is for and how a defender detects its use.
- `hint`: a hint from Ada, at three levels of detail.
- `notes` and `report`: the player's notebook, which feeds the final report.

**Stealth mechanic**

A "noise" gauge rises with every visible action. An aggressive `nmap` or too many login attempts push it up. At its maximum, NovaMind's SOC detects the player and the mission restarts from a save point. The player thus feels what detection means on the defence side.

## NPCs and AI

The whole game must be completable without an LLM. The LLM brings freedom of dialogue, not access to content. The four modes go through the same `NpcBrain` interface: the rest of the game does not know which one is active.

| Mode | How it works | Who it is for | Cost |
| --- | --- | --- | --- |
| Scripted (default) | Dialogue trees and intent recognition through keywords and patterns | Everyone, teenagers included | Free |
| Local model | Small model running in the browser through WebLLM and WebGPU | Recent machines | Free, about 1 to 2 GB download |
| BREACH gateway | Project server on a VPS: FreeLLMAPI behind a small API with quotas | Everyone, within the quota | Free for the player, providers' free quotas |
| BYOK | Direct call from the browser to Anthropic, OpenAI or OpenRouter with the player's key | Adults and the curious | Paid by the player |

**NPC design rules**

- **One sheet per NPC**: personality, what they know, what they must not say, and their deliberate weaknesses depending on the difficulty.
- **A deterministic judge**: the game itself checks whether the secret (the "flag") appears in the answer. The LLM never declares success, otherwise it could be manipulated into letting the player win.
- **Closed world**: each NPC's prompt keeps them inside NovaMind and refuses any help with real targets.
- **Adjustable difficulty**: the higher the level, the stronger the guardrails. The player sees what holds up and why.

**BYOK security**

- By default the key stays in the session. The "remember me" option keeps it in the browser's local storage, never on a server.
- A strict CSP, no third-party scripts and no analytics. A single XSS flaw would be enough to steal the keys.
- The setup screen recommends a dedicated key with a spending cap. That is already a security lesson.

**The BREACH gateway**

A service on a VPS, separate from the home network. FreeLLMAPI is never exposed directly: a small BREACH API sits in front of it.

- **Not a free LLM for everyone**: the browser only sends the NPC's identifier and the player's message. The server adds the NPC's prompt itself, which prevents the gateway from being used as a general-purpose chatbot.
- **Quotas**: one anonymous token per browser, issued after an invisible captcha (Cloudflare Turnstile). It grants a number of messages per day, plus a per-IP limit and a global cap.
- **Short answers**: a limited number of tokens per answer, and an output filter suited to a 16+ audience.
- **Graceful degradation**: when the quota runs out, the game falls back to scripted mode without blocking the player.
- **Privacy**: some free quotas may be used to train models. Providers are chosen accordingly, and the privacy page says so clearly.

## Art direction

The stance: a diegetic neon thriller. The whole screen is the pentester's workstation, a fictional OS called LR-OS. There is no classic game menu: everything happens in windows, like a TV series that films the hacker's screen, but beautiful. Moodboards to aim for: the UI of *Mr. Robot* for realism, the interfaces of *Blade Runner 2049* for lighting, *Hacknet* for readability.

**Palette**

| Role | Colour | Use |
| --- | --- | --- |
| Night | `#07090F` | Desktop background |
| Panel | `#0E1320` at 80% opacity | Frosted glass windows |
| Red team | `#FF3D5A` neon coral | Offensive actions, compromised machines |
| Blue team | `#2DE2FF` electric cyan | Defence, logs, Blue Team mode |
| AI | `#9B5CFF` violet | Chatbots, agents, anything that "thinks" |
| Success | `#3DFF9A` | Flags, validations |
| Alert | `#FFB020` amber | Noise gauge, detection |
| Text | `#D7E0F5` and `#6B7A99` | Primary and secondary text |

One strong rule: colour carries meaning. Red always means attack, cyan always defence, violet always AI. The player reads the state of the world without reading any text.

**Typography**

- **JetBrains Mono**: the terminal and data, with ligatures.
- **Space Grotesk**: mission titles and credits, large and widely spaced.
- **Inter**: interface text and messages.

**Interface**

- **LR-OS desktop**: movable, resizable windows (terminal, fictional browser, messaging with Ada, network map, notebook). Each has a frosted glass background, a thin glowing border and a coloured drop shadow.
- **The 3D network map**, the game's visual signature: hosts are glowing nodes linked by beams. A scan makes them appear one by one. A compromise turns a node red with a spreading wave. Mute leaves nodes that flicker.
- **Status bar**: the game clock, the amber noise gauge, the SOC alert level.

**Effects and animation**

- Scanlines and a light glow (bloom) on text, both optional.
- Ada's and the NPCs' text types itself out, with a blinking cursor.
- A glitch (chromatic aberration, slice offset) on strong events: detection, a chatbot that "breaks", Mute appearing.
- Data particles travelling along the beams during an exfiltration.
- Mission transitions as a "boot sequence": scrolling log lines, the LR-OS logo, the mission title in large type.

**Five "wow" moments to polish first**

1. The first start-up: black screen, LR-OS boot, then Ada typing "Welcome to Ligne Rouge".
2. The first `nmap`: the 3D map lights up machine by machine in time with the terminal output.
3. The chatbot giving in: its window glitches, its violet turns red, the system prompt scrolls by.
4. Detection: pulsing red screen edges, a muffled alarm, a countdown to disconnect.
5. The switch to Blue Team: a VHS-style "rewind" of your session, then the whole interface turns cyan and you see your own attacks in the logs.

**Accessibility**

The game respects `prefers-reduced-motion` and offers a toggle for CRT and glitch effects. Contrast targets the AA level, the terminal font size is adjustable, and no information is conveyed by colour alone: an icon always accompanies the state.

## Sound and feedback

Sound follows the tension: the higher the noise gauge, the more intense the music.

- **Music generated live** with Tone.js, in a calm darksynth style. Layers are added with noise and alert. No licences to manage.
- **Interface sounds**: mechanical keyboard typing in the terminal, windows opening and closing, Ada's notification.
- **Sound signatures**: a rising chord for a flag, a muffled alarm for detection, a crackle for Mute's traces.
- **Settings**: separate volumes for music, effects and interface, with a full mute mode.

## Technical architecture

The game is a static site: no game server, no accounts, no player data on our side. The only external connections are optional: the project's LLM gateway, or the direct call to the provider in BYOK mode.

> _[Diagram not exported: architecture · static site, optional LLM — see the original doc]_

The stack: React, Vite and TypeScript for the application, xterm.js for the terminal, react-three-fiber for the 3D map, GSAP for interface animations, Tone.js for sound, WebLLM for the local model, IndexedDB for saves.

**Hosting and domain**

The site goes on Cloudflare Pages or GitHub Pages, for free. The LLM gateway runs on the VPS, never on the home server. The code is released under the MIT licence, with French and English text from v1.

Domains checked on November 21, 2026 in the official registries (RDAP):

| Domain | Status | Indicative price |
| --- | --- | --- |
| `breachgame.fr` | Available | About €5 excl. VAT to register, €7.79 excl. VAT to renew at OVHcloud |
| `playbreach.fr` | Available | Same |
| `breachgame.dev` | Available | €18.59 incl. VAT per year at OVHcloud, HTTPS mandatory |
| `playbreach.dev` | Available | Same |
| `breachgame.app` | Available | Close to .dev |
| `breachgame.com`, `playbreach.com`, `breach-game.com`, `breachthegame.com`, `breach.fr` | Already taken | — |

In the meantime, a free subdomain (`breach.pages.dev` or `<account>.github.io/breach`) is enough.

## Vibe coding roadmap

We start with a very short, very polished vertical slice, then widen. A lot is one or two sessions with Claude Code.

1. **Lot 0, the vertical slice**: the LR-OS desktop, the terminal with about ten commands, mission 1, the final palette and fonts, and French and English i18n in place from the first screen. The goal is to have something worth showing from the very beginning.
2. **Lot 1, the mission engine**: missions described in YAML files (machines, files, objectives, flags, hints), noise gauge, notebook, saves.
3. **Lot 2, the 3D map and effects**: network map in react-three-fiber, glitch, boot sequences, the five "wow" moments.
4. **Lot 3, scripted NPCs**: messaging, Ada, employees, the Nova chatbot, missions 2 and 3.
5. **Lot 4, the LLM layer**: `NpcBrain` interface, WebLLM mode, BYOK mode, deterministic flag judge.
6. **Lot 5, the end of the campaign**: missions 4 to 6, Mute's arc, Blue Team mode, the pentest report.
7. **Lot 6, polish**: sound, accessibility, translation review, performance.
8. **Lot 7, playtesting**: have teenagers and adults play, tune the difficulty, release.

**Vibe coding tips**

- **A `CLAUDE.md` from the start**, with the palette, the fonts, the colour rules and the framework (never a real target). Claude will stay consistent from one session to the next.
- **Unit tests on the command engine**: it is the most fragile part, and the easiest to break while iterating.
- **Screenshots at every lot** to judge the visual result, not just the code.
- **Content separate from the engine**: missions in YAML make it possible to add new ones without touching the code, and later to let the community create them.

## Risks and open questions

| Risk | Mitigation |
| --- | --- |
| Perceived as a "game that teaches hacking" | Contract from mission 0, mandatory defensive debrief, no reusable exploit, fictional targets only |
| WebLLM too heavy for many machines | Full scripted mode. Local stays an option |
| Theft of BYOK keys | Strict CSP, no third-party scripts, key kept in the session by default |
| Players "breaking" LLM NPCs to cheat | Flags checked by the game, never by the LLM |
| 3D is expensive in time and performance | 2D fallback for the map, enabled automatically on low-end machines |
| Scope too broad for a personal project | Vertical slice first, then one mission at a time |

**Tracking**

**Decisions made**

- [x] Title: BREACH.
- [x] Languages: French and English from v1.
- [x] Licence: open source under MIT.
- [x] Hosting: free static service (Cloudflare Pages or GitHub Pages).
- [x] v1 scope: missions 0 to 3, scripted NPCs and BYOK, desktop only. Missions 4 to 6, the LLM gateway and WebLLM come later (Nov 22, 2026).
- [x] Blue Team: a short debrief at the end of each mission. The full Blue Team mode comes after v1 (Nov 22, 2026).
- [x] Flags: explicit submission by the player (Nov 22, 2026).
- [x] Missions as declarative packs, loaded on demand. See the [review](../review/revue-gdd-2026-11.en.md) and the [architecture](../architecture/architecture.en.md) (Nov 22, 2026).

**Still open**

- [ ] The domain name: `breachgame.fr` or `breachgame.dev` (see Architecture).
- [ ] The per-player quota on the free LLM gateway.
- [x] The service for the gateway: Cloudflare Workers, after v1 (Nov 22, 2026).
