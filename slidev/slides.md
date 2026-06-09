---
theme: default
title: "Grokking — How AI Learns to Learn"
info: secKC Talk by Patrick Ecord
drawings:
  persist: false
transition: fade-out
class: text-center
background: black
duration: 30min
timer: countdown
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

<div class="muted text-sm mt-6">
Based on Welch Labs — <em>"The most complex model we actually understand"</em> (YouTube)
</div>

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

<div class="absolute inset-0">
  <div v-click class="absolute border-4 border-cyan rounded-sm" style="left:12%;top:18%;width:28%;height:64%">
    <span class="absolute -bottom-7 left-0 text-cyan text-base font-mono font-bold tracking-widest uppercase">inputs</span>
  </div>
  <div v-click class="absolute border-4 border-yellow rounded-sm" style="left:43%;top:18%;width:23%;height:64%">
    <span class="absolute -bottom-7 left-0 text-yellow text-base font-mono font-bold tracking-widest uppercase">weights</span>
  </div>
  <div v-click class="absolute border-4 border-green rounded-sm" style="left:68%;top:18%;width:19%;height:64%">
    <span class="absolute -bottom-7 left-0 text-green text-base font-mono font-bold tracking-widest uppercase">output</span>
  </div>
</div>

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
  <div class="relative">
    <img src="/selected/02c_model_perspective.jpg" class="h-52 object-contain" />
    <div v-click class="absolute h-1.5 bg-cyan rounded-full" style="left:13.1%;top:44%;width:5%"></div>
    <div v-click class="absolute h-1.5 bg-cyan rounded-full" style="left:22.5%;top:50%;width:5%"></div>
    <div v-click class="absolute h-1.5 bg-cyan rounded-full" style="left:29.4%;top:68%;width:5%"></div>
    <div v-click class="absolute h-1.5 bg-green rounded-full" style="left:65.7%;top:56%;width:5%"></div>
  </div>
  <div class="relative">
    <img src="/selected/02d_llama_capital_france.jpg" class="h-52 object-contain" />
    <div v-click>
      <div class="absolute h-1.5 bg-cyan rounded-full" style="left:4.9%;top:48%;width:4%"></div>
      <div class="absolute h-1.5 bg-cyan rounded-full" style="left:13.2%;top:52%;width:4%"></div>
      <div class="absolute h-1.5 bg-cyan rounded-full" style="left:20.2%;top:34%;width:4%"></div>
      <div class="absolute h-1.5 bg-cyan rounded-full" style="left:27.3%;top:59%;width:4%"></div>
      <div class="absolute h-1.5 bg-cyan rounded-full" style="left:34.3%;top:40%;width:4%"></div>
    </div>
    <div v-click class="absolute h-1.5 bg-green rounded-full" style="left:86.6%;top:66%;width:4%"></div>
  </div>
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

<v-clicks>

Wrong answer? **Nudge the dials.**

Right answer? **Mostly leave them.**

Repeat until the loss stops yelling.

</v-clicks>

<div v-click class="signal mt-8">try -> measure error -> nudge -> repeat</div>

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

What's 11 + 2 on a 12-hour clock?

<v-click>

**1.**

</v-click>

<v-click>

When the number hits the max, it wraps around.

That's modular math.

</v-click>

::right::

<img src="/selected/09_clock_modular_math.jpg" class="h-80 mx-auto mt-12 object-contain" />

<!--
Before I show you the weird thing, I need to explain one concept: modular arithmetic.

Don't let the name scare you — you already know this. What's 11 + 2 on a 12-hour clock? It's 1. Not 13 — 1. Because when you hit 12, the clock wraps back around to 1. (unless you are using 24 hour time like a nerd)

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
  <div v-click class="flex flex-col items-center gap-2">
    <img src="/selected/01_modular_addition_table.jpg" class="h-64 object-contain" />
    <div class="muted text-sm">test set held back</div>
  </div>
</div>

<!--
Researchers set up a simple experiment.

They took a tiny neural network — way smaller than anything you'd use in production — and trained it to do modular addition. Specifically, A plus B mod 113.

This is a small example showing every combination of x and y, mod 5. On the right, you can see they cut out some of the squares — that's the test set the model never sees. It trains on the ones that are left, then gets evaluated on the cut-out ones.

The model's job: figure out the pattern. Learn to do this addition.

Simple enough, right?
-->

---

<div class="case-label">trace 02 // looks like memorization</div>

## The Result

<v-clicks>

100% accuracy on training examples — memorized the answers

~0% accuracy on testing examples — can't do new problems

</v-clicks>

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

<div v-click>

Nothing improves. For a long time.

</div>

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

As the story goes, this was almost found by accident — someone left one of these models training way longer than anyone normally would. And when they came back...

...the test accuracy just... snaps to 100%.

Not gradual. Not a slow improvement. It goes from basically zero to perfect in a tiny window of training steps.

Long pause. Let it sink in.

It stopped memorizing... and started solving.

It looked like nothing was happening during training, but inside, the model was quietly building patterns. And then it clicked.

And they thought: what the hell is going on inside this model?
-->

---
layout: two-cols
---

## That's grokking

A model looks like it memorized the training set...

then suddenly generalizes to the rule.

<div v-click class="signal mt-8">overfit -> plateau -> snap</div>

<div v-click class="muted text-sm italic mt-8 pr-10">
"You cannot hate anything unless you grok it — understand it so thoroughly that you merge with it, and it merges with you."
<div class="mt-2 not-italic">— Robert A. Heinlein, <em>Stranger in a Strange Land</em></div>
</div>

::right::

<img src="/selected/07_stranger_in_strange_land.jpg" class="h-80 mx-auto mt-8 object-contain" />

<!--
This delayed snap is what the researchers called grokking.

The word comes from Robert Heinlein's Stranger in a Strange Land. To grok something is to understand it deeply, not just know the answer.

In this talk we're using it in the machine learning sense: at first the model memorizes the examples, then later finds the rule that generalizes.

The next question is the fun one: what changed inside the model?
-->

---

<div class="case-label">reverse engineering pass // no source, just traces</div>

## So we popped the hood.

<v-click>

**Expected:** lookup-table junk.

</v-click>

<v-click>

**Reality:** geometry.

</v-click>

<img src="/selected/11_neuron_evolution_forward.gif" class="h-72 mx-auto object-contain" />

<!--
Now we dig into the model — treat it like an unknown binary and reverse engineer it. We're going to look at the weights and activations and try to understand how it actually learned the rule.

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
Here's what the fully trained model looks like inside. This is a single-layer transformer — the inputs come in and get transformed into these activations.

On the left are the activations themselves — clean sine waves. On the right, we plot pairs of neurons against each other. A neuron plotted against itself is just a straight line. But plot the first neuron against the second, and it curls into a loop — a circle.

This is the moment where the autopsy gets interesting. This is supposed to be a model that does addition. Why is it drawing circles?
-->

---
layout: two-cols
---

## The model discovered something

<v-clicks>

Modular math wraps around... just like a clock.

The model learned to represent numbers as **positions on a circle.**

Addition = **rotation** around the circle.

</v-clicks>

::right::

<img src="/selected/10_clock_x_plus_y_rotation.jpg" class="h-80 mx-auto mt-8 object-contain" />

<!--
You guys remember the clock from earlier — how modular arithmetic wraps around, just like a clock face? Well, the model figured that out on its own too.

It learned to represent numbers as a position on a circle, and it discovered that that was a useful way to think about the problem.

And so addition just becomes rotation around that circle. 11 + 2? You land on 1 — you can see that in the picture here. The model's doing the same thing with its own internal representation.

The model isn't thinking in numbers, it's thinking in positions on a circle. Nobody told it this — the training data only implied it, and the model figured out it was an easy way to solve the problem.
-->

---

<div class="case-label">trace 06 // circuit shape</div>

## Inside the model

Early layers: learn **trig-like features** of the inputs.

<img src="/selected/16_pipeline_cos_sin_xy.jpg" class="h-80 mx-auto object-contain" />

<!--
Let's get specific about what's actually happening inside the model.

Our input comes in on the left and goes into a sparse linear probe — the x column gives us cosine of x and sine of x, and the y column gives us cosine of y and sine of y. Then it feeds into the attention and the multi-layer perceptron, and out comes an answer.

You might be wondering: why is it learning sine and cosine? If you remember anything from math class — or even if you don't — sine and cosine are just the x and y coordinates of a point on a circle.

The model learned to put numbers on a circle. On its own.
-->

---

<div class="case-label">trace 07 // products of waves</div>

## Middle layers

Computes **products** of those functions — cos(kx) * cos(ky)

<img src="/selected/18_surface_cos_kx_cos_ky.jpg" class="h-80 mx-auto object-contain" />

<!--
In these middle layers is where it actually gets interesting. The model starts multiplying those trig functions together.

What we're looking at is a 3D surface — the output of a single neuron as you vary both x and y. One axis is x, one is y, and the surface is the combination of them. The dominant pattern is cosine of x times cosine of y.

It probably seems random — why would multiplying cosines together help you add? Stick with me, and you'll see how it all comes together in a couple slides.
-->

---

<div class="case-label">trace 08 // it learned to add</div>

## The diagonal

<v-click>

A single neuron fires for every pair of inputs where **x + y = 65**.

</v-click>

<v-click>

It learned to **add**.

</v-click>

<img src="/selected/20_diagonal_stripes_with_sums.jpg" class="h-72 mx-auto object-contain" />

<!--
This is where it clicks.

Look at these diagonal stripes. Each stripe is a set of input pairs where the neuron fires maximally.

Look at the numbers along the top stripe: x=0 y=65, x=20 y=45, x=40 y=25, x=60 y=5. What do they have in common? They all add up to 65.

[Pause — let someone figure it out.]

So this neuron fires for every pair of inputs that sum to 65. It learned to detect addition — not by adding, but by geometry.

And the second stripe? Those pairs add up to 178. But 178 mod 113 is 65 — same answer, just wrapped around.
-->

---

<div class="case-label">trace 09 // the trick, named</div>

## The trig identity

**cos(kx)cos(ky) - sin(kx)sin(ky) = cos(k(x+y))**

A trigonometric identity converts products of trig functions into a **sum of the inputs**.

<img src="/selected/22_trig_identity_reveal.jpg" class="h-64 mx-auto object-contain" />

<!--
Remember that scary equation from the very first slide? This is how it all ties together.

We get our sine and cosine waves, they go through the attention and the perceptron, and come out as those 3D surfaces — and the answer reads out on the right.

You can see the equation here: cos(kx)cos(ky) - sin(kx)sin(ky) = cos(k(x+y)). All that's to say — you can combine sines and cosines to get addition back out. It's a trick that's been in textbooks for centuries.

Nobody wrote that identity into the model. It emerged on its own, because this representation solves the problem cleanly.

So it didn't really learn addition. It learned geometry.

(beat) ...when's the last time you had to do something like this, huh?
-->

---

<div class="case-label">trace 10 // end to end</div>

## The full picture

<img src="/selected/24_full_pipeline_with_identity.jpg" class="h-96 mx-auto object-contain" />

<!--
Here's the whole pipeline. Numbers go in on the left, the answer comes out on the right.

Walk it through: the numbers go into the sparse linear probe, which gives us cosine and sine of x and y. That feeds into the attention and the multi-layer perceptron, which gives us those 3D surfaces. And that reads out as an answer — one of those outputs lights up for the right sum, like the 65 line we just looked at.

You don't need to follow every piece. The point is every step has a purpose, and the model built this whole thing from scratch — the representation, the strategy, the math.

It learned a space where the problem becomes easy.
-->

---

## Why this is wild

<v-clicks>

- Not explicitly programmed to do this
- Not explicitly in the training data
- Appears late in training
- Hidden the whole time

</v-clicks>

<img src="/selected/23_training_evolution_noisy_to_clean.jpg" class="h-48 mx-auto object-contain" />

<!--
So let's step back and think about what just happened.

The model was never told about circles, or sine, or cosine. Nobody wrote trigonometry into the code. The training data was just examples like 14 + 87, mod 113.

The model figured out a representation that was an efficient way to solve the problem — and it just happened to be geometric, trig-like.

And it did it really late. For thousands of steps it looked like nothing was happening — most people would have walked away from the training. (The way the story goes, they only caught it because someone left it running way longer than anyone normally would.) The capability was hidden the entire time.

It looked dumb... until it didn't.
-->

---

## Meanwhile, at scale...

Anthropic found a **6-dimensional manifold** in Claude Haiku that handles line break arithmetic.

<div v-click>

The same kind of geometric structure. In a production model.

</div>

<img src="/selected/25_anthropic_manifold_paper.jpg" class="h-64 mx-auto object-contain" />

<!--
Here's the part that should make you feel a little weird.

Everything I just showed you was a tiny model — a single-layer transformer trained on a toy problem. You might think: okay, cute, but real models are way bigger, way more parameters.

Except Anthropic published a paper where they looked inside Claude Haiku — a real production model — and found the same sort of thing.

When Claude is writing text, it needs to figure out where to put a line break. To do that, it has to count how many characters it's already written on the line. And the way it does that? A six-dimensional geometric manifold — a curved surface in six-dimensional space where character count and line width are positions on a helix. You can see it in this picture: one axis is the character count, the other is how long the line should be.

So even in a model with billions of parameters, we're seeing the same kind of geometric structure — used for something as mundane as line breaks.

If a tiny model rediscovered trig to do addition... what do you think the big models have hiding inside? What else is in there?
-->

---
layout: center
class: text-center
---

## What we learned

<br>

<v-clicks>

Learning isn't linear

Understanding can emerge suddenly

Models build internal structure we didn't ask for

</v-clicks>

<br>

<div v-click>

### "It looked dumb... until it didn't."

</div>

<!--
So that's grokking.

Learning isn't linear — a model can look stuck for ages, then snap into a whole new strategy.

Understanding — or something that looks an awful lot like it — can show up all at once, with no warning.

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
