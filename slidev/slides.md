---
theme: default
title: "Grokking — How AI Learns to Learn"
info: secKC Talk by Patrick Ecord
drawings:
  persist: false
transition: fade-out
class: text-center
background: black
---

<style>
  :root {
    --sec-bg: #050505;
    --sec-fg: #f5f5f0;
    --sec-dim: #a8a8a0;
    --sec-cyan: #54f1ff;
    --sec-green: #6cff9d;
    --sec-red: #ff4d4d;
  }

  .slidev-layout {
    background-color: var(--sec-bg) !important;
    color: var(--sec-fg) !important;
  }

  .slidev-layout h1,
  .slidev-layout h2,
  .slidev-layout h3 {
    color: var(--sec-fg) !important;
    letter-spacing: 0 !important;
  }

  .case-label {
    color: var(--sec-cyan);
    font-family: "Fira Code", monospace;
    font-size: 0.95rem;
    letter-spacing: 0 !important;
    text-transform: uppercase;
  }

  .signal {
    color: var(--sec-green);
    font-family: "Fira Code", monospace;
  }

  .danger {
    color: var(--sec-red);
  }

  .muted {
    color: var(--sec-dim);
  }

  .cold-open {
    height: 100%;
    display: flex;
    align-items: center;
    justify-content: center;
  }

  .cold-open img {
    max-width: 100%;
    max-height: 92%;
    object-fit: contain;
  }
</style>

<div class="cold-open">
  <img src="/selected/22_trig_identity_reveal.jpg" />
</div>

<!--
Let the trig identity sit on screen for a second. Let people read it and groan.

So... when was the last time you learned a trigonometric identity?

Pause.

Yeah. Same.

Beat.

Yeah well neither did this clanker.

Tonight we're going to look at how a model learns math. Kinda sorta.
-->

---
layout: center
class: text-center
---

<div class="case-label">secKC // reverse engineering // gradient descent did what?</div>

# Grokking

## How AI Learns to Learn

<div class="muted mt-8">Patrick Ecord</div>

<!--
Hello everyone. My name is Patrick. It's great to see your beautiful faces.

Tonight we're going to talk about how AI learns to learn. We're going under the hood — we'll see what the model sees, and how training works, how a pattern emerges from noise.

And hopefully by the end of tonight, we'll have a better idea of how it works at a base level.
-->

---
layout: image
image: /selected/02a_perceptron_board.jpg
backgroundSize: contain
---

<!--
This is an artificial neuron, also called a perceptron, built as a physical machine in the 1950s.

The switches on the left are the inputs, the dials are the weights, the meter on the right is the output.

Right now the inputs are set to a T — you can see that T lit up, those are turned on, everything else off. The dials in the middle are tuned to detect that T, so when the neuron sees it, the meter on the right swings positive.

Modern models have billions or trillions of these, but the idea is basically the same: you feed in a pattern, you adjust the weights, and you get the output you want.

We're going to start with this simple machine and reverse engineer how a tiny model learned modulus addition.
-->

---
layout: two-cols
---

## What does a model see?

We say "1 + 2 = 3"

The model sees **switch patterns**.

Input pattern in. Output pattern out.

No numbers. No meaning. Just mappings.

The **`=`** is just another switch — the model's placeholder for the answer.

Same trick scales: a real model maps **"The capital of France is" → Paris**.

::right::

<div class="flex flex-col items-center gap-4 mt-6">
  <img src="/selected/02c_model_perspective.jpg" class="h-52 object-contain" />
  <img src="/selected/02d_llama_capital_france.jpg" class="h-52 object-contain" />
</div>

<!--
So here's an example: 1 + 2 = 3. Pretty straightforward, right? (at least to some of us)

But the model doesn't know what a number is, or an equals sign — it's us humans who give these symbols meaning. Imagine the numbers covered up: all the model sees is a switch flipped on for "1", then "2", then "=", and it learns that when it sees that pattern, it outputs a 3.

You might be wondering why we have an equals symbol but no addition symbol — the equals sign gives the model a placeholder for where the answer goes. It's kind of like the end of a sentence.

And the same thing happens with a real LLM. That second screenshot is Llama 3.2 (1B), with a 128k vocab — plug in "the capital of France is" and those input tokens light up, and it outputs Paris.

But to the model, this is pattern in, pattern out. That's the whole game.

So if we want to know whether it learned the rule, we need to inspect what happens inside while the dials are being turned.
-->

---
layout: two-cols
---

## How does it learn?

Wrong answer? **Nudge the dials.**

Right answer? **Mostly leave them.**

Repeat until the loss stops yelling.

<div class="signal mt-8">try -> measure error -> nudge -> repeat</div>

::right::

<img src="/selected/02e_perceptron_labeled.jpg" class="h-80 mx-auto mt-12 object-contain" />

<!--
The way it learns isn't magic, it's a feedback loop.

You try an answer, measure how wrong it was, nudge the dials in the right direction, and do that an absurd number of times.

Here's the actual rule — the four cases on screen. If the target should be positive but the output came out negative, you turn the weights up. The other way around — target negative, output positive — you turn them down. And if the output already matches the target, you leave it alone. No change.

This is basically backpropagation. Think back to the capital of France: early in training the model guesses wrong, you show it the right answer, it measures how far off it was, and it nudges its weights to do better next time.

A real model has way more weights than you could ever turn by hand, but the mental model is the same — training nudges the weights until the output looks right.

And this rule is older than you'd think — Frank Rosenblatt came up with the perceptron back in the 1950s.
-->

---
layout: two-cols
---

## Modular Arithmetic

What's 10 + 5 on a 12-hour clock?

**3.**

When the number hits the max, it wraps around.

That's modular math.

::right::

<img src="/selected/09_clock_modular_math.jpg" class="h-80 mx-auto mt-12 object-contain" />

<!--
Before I show you the weird thing, I need to explain one concept: modular arithmetic.

Don't let the name scare you — you already know this. What's 10 + 5 on a 12-hour clock? It's 3. Not 15 — 3. Because when you hit 12, the clock wraps back around to 1. (unless you are using 24 hour time like a nerd)

That wraparound is the whole trick. Keep the clock in your head, because the model is going to independently discover a version of it.
-->

---

<div class="case-label">trace 01 // toy problem, real behavior</div>

## The Experiment

Researchers trained a tiny model on modular addition.

All the combinations of x + y, mod 5 (the real run used mod 113). Held back some for testing.

<div class="flex justify-center items-center gap-8 mt-4">
  <div class="flex flex-col items-center gap-2">
    <img src="/selected/01b_modular_table_full.jpg" class="h-64 object-contain" />
    <div class="muted text-sm">full dataset</div>
  </div>
  <div class="flex flex-col items-center gap-2">
    <img src="/selected/01_modular_addition_table.jpg" class="h-64 object-contain" />
    <div class="muted text-sm">test set held back</div>
  </div>
</div>

<!--
Researchers set up a simple experiment.

They took a tiny neural network — way smaller than anything you'd use in production — and trained it to do modular addition. Specifically, A plus B mod 113.

This is a small example — every combination of x and y, mod 5. They held some back as a test set the model never sees.

The model's job: figure out the pattern. Learn to do this addition.

Simple enough, right?
-->

---

<div class="case-label">trace 02 // looks like memorization</div>

## The Result

100% accuracy on training examples — memorized the answers

~0% accuracy on testing examples — can't do new problems

<img src="/selected/03_training_curve_memorized.jpg" class="h-72 mx-auto object-contain" />

<!--
And here's what happened. The blue line is accuracy on the training data — the problems it saw during training. It shoots up to 100%. The model memorized every single answer.

But the yellow line — that's accuracy on the test set. The problems it hadn't seen. Flat. Basically zero.

It's like a student who memorized every answer on the practice exam but can't solve a single new problem. It didn't learn addition — it learned a lookup table.

If you're a machine learning engineer, you look at this and say 'classic overfitting' and you stop training. You're done. The model learned what it's going to learn.
-->

---

<div class="case-label">trace 03 // the boring part is suspicious</div>

## The Plateau

Training: 100%. Testing: still flat at 0%.

Nothing improves. For a long time.

<img src="/selected/04c_grokking_complete.jpg" class="h-72 mx-auto object-contain" />

<!--
And if you keep training... nothing happens. For a long time.

The training accuracy stays at 100%. The test accuracy stays near zero. Step after step after step. Nothing changes.

You would absolutely stop training here. There's zero indication that anything useful is going to happen. It looks like wasted compute.

But the experiment keeps going. And the weird part is that the boring flat line isn't actually boring inside the model.
-->

---

<div class="case-label">trace 04 // strategy shift</div>

## Then...

<img src="/selected/05_grokking_curve_labeled.jpg" class="h-96 mx-auto object-contain" />

<!--
Let the image do the work. Pause before speaking.

...the test accuracy just... snaps to 100%.

Not gradual. Not a slow improvement. It goes from basically zero to perfect in a tiny window of training steps.

Long pause. Let it sink in.

It didn't get better. It changed strategies.

It stopped memorizing... and started solving.

Somewhere during all those flat training steps where nothing appeared to be happening — the model was quietly building something. And then it clicked.

And they thought: what the hell is going on inside this model?
-->

---
layout: two-cols
---

## That's grokking

A model looks like it memorized the training set...

then suddenly generalizes to the rule.

<div class="signal mt-8">overfit -> plateau -> snap</div>

::right::

<img src="/selected/07_stranger_in_strange_land.jpg" class="h-80 mx-auto mt-8 object-contain" />

<!--
This delayed snap is what the researchers called grokking.

The word comes from Robert Heinlein's Stranger in a Strange Land. To grok something is to understand it deeply, not just know the answer.

In this talk I'm using it in the machine learning sense: the model first fits the examples, then much later finds a rule that generalizes.

The next question is the fun one: what changed inside the model?
-->

---

<div class="case-label">reverse engineering pass // no source, just traces</div>

## So we popped the hood.

**Expected:** lookup-table junk.

**Reality:** geometry.

<img src="/selected/11_neuron_evolution_forward.gif" class="h-72 mx-auto object-contain" />

<!--
Now we're doing the hacker part. Treat the trained model like a binary we didn't write.

No source code that says 'solve modular addition.' Just weights, activations, and traces.

The obvious expectation is messy lookup behavior: arbitrary internal state that happens to produce the right answers.

What they actually found... watch this.

Let the gif play through once. It shows structure emerging from noise.

You're watching the internal structure of the model evolve as it trains. It goes from complete noise — random scattered dots — to clean geometric patterns. Waves. Loops. Circles.

Something organized is happening inside this model. Something nobody asked it to do.
-->

---

<div class="case-label">trace 05 // activations organize</div>

## Structure emerges

Waves. Loops. Circles.

<img src="/selected/11_neuron_scatter_plots.jpg" class="h-80 mx-auto object-contain" />

<!--
Here's what the fully trained model looks like inside.

These are actual neuron activations — the outputs of individual neurons plotted against each other. And look at the structure.

Clean sine waves on the left. Those loops and circles on the right? Those are pairs of neurons plotted against each other, and they're forming perfect geometric shapes.

This is the moment where the autopsy gets interesting. This is supposed to be a model that does addition. Why is it drawing circles?
-->

---
layout: two-cols
---

## The model discovered something

Modular math wraps around... just like a clock.

The model learned to represent numbers as **positions on a circle.**

Addition = **rotation** around the circle.

::right::

<img src="/selected/10_clock_x_plus_y_rotation.jpg" class="h-80 mx-auto mt-8 object-contain" />

<!--
Remember the clock from earlier? Modular arithmetic wraps around — just like a clock face.

Well... the model figured that out on its own.

It learned to represent each number as a position on a circle. Not because anyone told it to — it discovered that this is a useful way to think about the problem.

And addition? Addition becomes rotation around the circle. 10 + 5 on a clock? Start at 10, rotate 5 positions, land on 3. The model is doing the same thing — in its own internal representation.

The model stops thinking in numbers and starts thinking in positions.

Nobody explicitly programmed this. The training data only implies the wraparound rule. The model found a representation where that rule is easy.
-->

---

<div class="case-label">trace 06 // circuit shape</div>

## Inside the model

Early layers: learn **trig-like features** of the inputs.

<img src="/selected/16_pipeline_cos_sin_xy.jpg" class="h-80 mx-auto object-contain" />

<!--
Now let's get specific about what's actually happening inside the model.

In the early layers, the model learns features that look like sine and cosine waves of the inputs. You can see them here — cos of x, sin of x up top, and cos of y, sin of y on the bottom.

Why sine and cosine? Because that's how you put a point on a circle. If you remember anything from trig class — or even if you don't — sine and cosine are just the x and y coordinates of a point on a circle.

The model learned to put numbers on a circle. On its own.
-->

---

<div class="case-label">trace 07 // products of waves</div>

## Middle layers

Computes **products** of those functions — cos(kx) * cos(ky)

<img src="/selected/18_surface_cos_kx_cos_ky.jpg" class="h-80 mx-auto object-contain" />

<!--
In the middle layers, things get more interesting. The model starts multiplying these trig functions together.

What you're looking at is a 3D surface — the output of a single neuron as you vary both inputs x and y. And the dominant pattern is cos of x times cos of y.

Now this might seem random. Why would multiplying cosines together help you do addition? Bear with me for two more slides.
-->

---

<div class="case-label">trace 08 // it learned to add</div>

## The diagonal

A single neuron fires for every pair of inputs where **x + y = 65**.

It learned to **add**.

<img src="/selected/20_diagonal_stripes_with_sums.jpg" class="h-72 mx-auto object-contain" />

<!--
This is where it clicks.

Look at these diagonal stripes. Each stripe represents a set of input pairs where the neuron fires maximally.

Look at the numbers along the top stripe: x=0 y=65, x=20 y=45, x=40 y=25, x=60 y=5, x=65 y=0. What do they all have in common?

Pause. Let someone in the audience figure it out.

They all add up to 65.

This neuron fires for every pair of inputs whose sum is 65. It learned to detect addition. Not by adding — by geometry. By the orientation of a wave in its internal space.

And the second stripe? Those pairs add to 178. But 178 mod 113 is... 65. Same answer, wrapped around.
-->

---

<div class="case-label">trace 09 // the trick, named</div>

## The trig identity

**cos(kx)cos(ky) - sin(kx)sin(ky) = cos(k(x+y))**

A trigonometric identity converts products of trig functions into a **sum of the inputs**.

<img src="/selected/22_trig_identity_reveal.jpg" class="h-64 mx-auto object-contain" />

<!--
And here's the punchline.

Remember that scary equation from the first slide? This is the shape of the trick.

There's a trick in math where if you multiply trig functions together in the right way, you get addition out the other side. It's a shortcut that's been in textbooks for centuries.

The model learned a circuit that behaves like this. Nobody wrote the identity into the model. It emerged because this representation solves the problem cleanly.

So yeah... it reinvented the useful part of trig.

Pause.

It didn't learn addition. It learned a geometry.
-->

---

<div class="case-label">trace 10 // end to end</div>

## The full picture

<img src="/selected/24_full_pipeline_with_identity.jpg" class="h-96 mx-auto object-contain" />

<!--
Here's the whole pipeline. Numbers go in on the left, correct answers come out on the right.

You don't need to follow every piece of this — the point is that every step has a purpose. The model built this entire process from scratch. It invented the representation, the strategy, and the math to make it work.

Sine and cosine waves on the left. Those 3D surfaces in the middle. The trig identity up in the top right. All of it — discovered by the model.

It learned a space where the problem becomes easy.
-->

---

## Why this is wild

- Not explicitly programmed to do this
- Not explicitly in the training data
- Appears late in training
- Hidden the whole time

<img src="/selected/23_training_evolution_noisy_to_clean.jpg" class="h-48 mx-auto object-contain" />

<!--
So let's step back and think about what just happened.

Nobody told this model about circles. Nobody told it about sine or cosine. Nobody wrote a trig identity into the code. The training data is just examples like 'what's 14 plus 87 mod 113.'

The model found a geometric representation and trig-like features because that turned out to be an efficient way to solve the problem.

And it did it late. For thousands of training steps it looked like nothing was happening. Any reasonable engineer would have stopped training. The capability was hidden the entire time.

It looked dumb... until it didn't.
-->

---

## Meanwhile, at scale...

Anthropic found a **6-dimensional manifold** in Claude Haiku that handles line break arithmetic.

The same kind of geometric structure. In a production model.

<img src="/selected/25_anthropic_manifold_paper.jpg" class="h-64 mx-auto object-contain" />

<!--
Now here's the part that should make you a little uncomfortable.

Everything I just showed you was a tiny model. A single-layer transformer trained on a toy problem. You might think — okay, cute, but real models are different.

Except... Anthropic published a paper where they looked inside Claude Haiku — a real, production language model — and found the same kind of thing.

When Claude is writing text and needs to figure out when to insert a line break, it has to count how many characters it's written on the current line. And the way it does this? A six-dimensional geometric manifold. A curved surface in six-dimensional space where character count and line width are represented as positions on a helix.

The same kind of geometric structure. In a production model with billions of parameters. Being used for something as mundane as line breaks.

If a tiny model rediscovers trig to do addition... what are bigger models doing? What geometric structures are hiding inside GPT-4 or Claude that we haven't found yet?

What else is in there?
-->

---
layout: center
class: text-center
---

## What we learned

<br>

Learning isn't linear

Understanding can emerge suddenly

Models build internal structure we didn't ask for

<br>

### "It looked dumb... until it didn't."

<!--
So that's grokking.

Learning isn't linear. A model can look like it's doing nothing for a long time and then suddenly snap into a completely different strategy.

Understanding — or something that looks a lot like understanding — can emerge suddenly and without warning.

And models build internal structure that nobody asked for. Structure that turns out to be elegant, geometric, and mathematically sophisticated.

We started tonight by saying AI is just pattern matching. Just statistics. Just optimization. And that's true — technically. But when the optimization discovers trigonometry on its own... maybe 'just' is doing a lot of heavy lifting in that sentence.

Pause.

It looked dumb... until it didn't.

Thank you.
-->

---
layout: center
class: text-center
---

## Links

<br>

Welch Labs — The most complex model we actually understand
https://youtu.be/D8GOeCFFby4

Welch Labs — The Perceptron
https://youtube.com/watch?v=l-9ALe3U-Fg

<br>

Power et al. (2022) — Grokking
https://arxiv.org/pdf/2201.02177

Nanda et al. (2023) — Mechanistic Interpretability
https://arxiv.org/pdf/2301.05217v1

Anthropic (2025) — Linebreaks in Claude Haiku
https://transformer-circuits.pub/2025/linebreaks/index.html

<!--
Leave this up for Q&A. People can take a photo.
-->
