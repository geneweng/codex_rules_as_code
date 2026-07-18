# Rules as Code: An AI-Oriented Survey

This repository contains a research survey of **Rules as Code (RaC)**, with particular attention to the opportunities and risks introduced by artificial intelligence.

## Documents

- [`rules-as-code-survey.pdf`](rules-as-code-survey.pdf) — publication-ready survey
- [`rules-as-code-survey.md`](rules-as-code-survey.md) — editable source with linked references

## Scope

The survey covers:

- definitions and the Rules as Code maturity spectrum;
- benefits, suitable use cases, and technical approaches;
- international activity and representative tools;
- AI-assisted rule extraction, formalization, testing, simulation, and explanation;
- the use of executable rules to constrain AI agents;
- legal, institutional, and technical risks;
- a maturity assessment and practical adoption roadmap.

The central finding is that AI is most useful around a deterministic, governed rule core: helping people extract, draft, test, maintain, and explain formal rules. An unconstrained language model should not itself serve as the authoritative rule engine for consequential decisions.

## Build

Requirements:

- [Pandoc](https://pandoc.org/)
- a LaTeX distribution providing `pdflatex`

Run:

```sh
pandoc rules-as-code-survey.md \
  --from=markdown \
  --pdf-engine=pdflatex \
  -V geometry:margin=0.8in \
  -V colorlinks=true \
  -V linkcolor=blue \
  -V urlcolor=blue \
  -o rules-as-code-survey.pdf
```

## Research note

The source survey was prepared on **18 July 2026** from linked government, intergovernmental, project, and academic sources. Rules, programs, and AI regulation continue to change; readers should verify current legal and operational details before relying on the document for a specific decision.

## License

The repository does not currently declare a license. The linked third-party sources remain subject to their respective terms.
