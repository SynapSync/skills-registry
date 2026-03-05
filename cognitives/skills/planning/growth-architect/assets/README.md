# Growth Architect — Assets

Directory index for the growth-architect skill's modular assets.

## Structure

```
assets/
├── README.md                  ← You are here
├── modes/
│   └── ANALYZE.md             ← 10-step strategic analysis methodology
├── helpers/
│   └── focus-modes.md         ← CTO/Product/Growth/Investor/Technical/Brutal modifiers
└── templates/
    ├── ANALYSIS.md            ← Output file template for strategic analyses
    └── ADR.md                 ← Architecture Decision Record template + lifecycle
```

## Usage

Assets are loaded on-demand based on mode and context:

| Asset | When to Read |
|-------|-------------|
| `ANALYZE.md` | Always — the core methodology for ANALYZE mode |
| `focus-modes.md` | Only when the user specifies a focus (CTO, Product, Growth, Investor, Technical, Brutal) |
| `ANALYSIS.md` | Only when generating the output analysis file |
| `ADR.md` | Only when creating an Architecture Decision Record |
