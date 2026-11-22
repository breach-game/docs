<p align="center">
  <picture>
    <source media="(prefers-color-scheme: dark)" srcset="assets/breach-wordmark-dark-bg.svg">
    <img alt="BREACH" src="assets/breach-wordmark-light-bg.svg" width="380">
  </picture>
</p>

# BREACH · docs

Design documentation for [BREACH](https://github.com/breach-game/breach), a cybersecurity thriller that runs in your browser: you attack a fictional AI startup under contract, then replay every step as the defender.

**Play:** https://breach-game.github.io/breach/ · **Code:** [breach-game/breach](https://github.com/breach-game/breach) · **Community missions:** [breach-game/packs](https://github.com/breach-game/packs)

Most documents were first written in French. Each one now has an English translation next to it; the French original stays the reference.

| Document | English | French (original) | Description |
| --- | --- | --- | --- |
| Game Design Document | [EN](gdd/game-design-document.en.md) | [FR](gdd/game-design-document.md) | Pitch, framework, game loop, missions, simulated terminal, NPCs and AI, art direction, architecture, roadmap |
| GDD review | [EN](review/revue-gdd-2026-11.en.md) | [FR](review/revue-gdd-2026-11.md) | Critical review before development: inconsistencies, risks, v1 scope, decisions to make |
| Architecture (draft) | [EN](architecture/architecture.en.md) | [FR](architecture/architecture.md) | Event-log engine, mission pack format, community levels, security |
| [Mockups](mockups/) | | | Title screen, LR-OS desktop, Blue Team console, 3D network map, 3D exfiltration |

Live version of the GDD (with diagrams, in French): https://claude.ai/artifact/LxRR4mbsTq5oRgqeECdhZj

The full visual identity lives in [`breach-game/.github`](https://github.com/breach-game/.github/tree/main/brand).

## Contributing

Questions and suggestions about the design are welcome as [issues](https://github.com/breach-game/docs/issues). When you change a document, update both languages: the French file stays the reference, and its English translation sits next to it as `<name>.en.md`. Everyone taking part follows the [Code of Conduct](https://github.com/breach-game/.github/blob/main/CODE_OF_CONDUCT.md).

## Licence

The documents and mockups in this repository are © 2026 Marco Mornati and released under the [Creative Commons Attribution 4.0 International licence](LICENSE) (CC BY 4.0): you can share and adapt them, including commercially, as long as you credit "BREACH (breach-game)" with a link to this repository. The game's code is under the MIT licence in [breach-game/breach](https://github.com/breach-game/breach).
