# Grokking — How AI Learns to Learn

A secKC talk on the grokking phenomenon in neural networks.

## View the slides

**https://pecord.github.io/grokking-talk/**

Arrow keys to navigate.

## Run locally (with speaker notes)

```bash
npm install
npx slidev slidev/slides.md
```

Then open http://localhost:3030/presenter/ for presenter mode with notes and timer.

## What is this talk about?

We demystify how neural networks actually work — what they see, how they learn, and what's really going on under the hood. Then we look at a phenomenon called "grokking," where a tiny model trained on simple math suddenly went from memorizing answers to inventing its own geometric solution using trigonometry nobody taught it. What does it mean when the optimization discovers something on its own?

## Credits

Visuals from [Welch Labs](https://www.youtube.com/@WelchLabsVideo):

- [The most complex model we actually understand](https://youtu.be/D8GOeCFFby4)
- [ChatGPT is made from 100 million of these [The Perceptron]](https://www.youtube.com/watch?v=l-9ALe3U-Fg)

Papers:

- [Power et al. (2022) — Grokking: Generalization Beyond Overfitting on Small Algorithmic Datasets](https://arxiv.org/pdf/2201.02177)
- [Nanda et al. (2023) — Progress measures for grokking via mechanistic interpretability](https://arxiv.org/pdf/2301.05217v1)
- [Anthropic (2025) — When Models Manipulate Manifolds: Linebreaks in Claude Haiku](https://transformer-circuits.pub/2025/linebreaks/index.html)
