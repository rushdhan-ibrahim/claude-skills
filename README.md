# Claude Skills

A collection of reusable skills for Claude Code, documenting patterns, techniques, and best practices from various projects.

## Skills

### [cosmic-phenomena.md](./cosmic-phenomena.md)

Creating immersive cosmic visual and audio experiences for web applications. Covers:

- **Visual Effects**: Parallax starfields, nebulae, shooting stars, glow effects
- **Audio Synthesis**: Web Audio API patterns, layered soundscapes, spatial audio
- **Physics**: Spring dynamics, orbital motion, gravitational attraction
- **Optimization**: Throttling, visibility-based animation, mobile adaptation, memory management

### [admixtools-qpadm.md](./admixtools-qpadm.md)

Ancient DNA ancestry modeling using ADMIXTOOLS qpAdm. Covers:

- **Data Preparation**: 23andMe → EIGENSTRAT conversion, AADR merge pipeline
- **Model Design**: Source/outgroup selection, qpWave rank testing
- **Execution**: Parameter files, output parsing, quality criteria
- **Validation**: F4-statistics, positive controls, nested model tests
- **Robustness**: Outgroup rotation, proxy sensitivity, Steppe Certainty Protocol
- **Interpretation**: Uncertainty quantification, language constraints, caveats

## Usage

These skills can be referenced by Claude Code to apply proven patterns when building similar features. Place skill files in your project's `.claude/skills/` directory to make them available in context.

## License

MIT
