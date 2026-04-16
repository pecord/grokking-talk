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

# Grokking

## How AI Learns to Learn

<style>
  h1 { color: #fff !important; }
  h2 { color: #ccc !important; }
</style>

<!--
Title card — on screen while people are settling in. No need to say anything yet.
-->

---
layout: center
class: text-center
---

## Visuals courtesy of **Welch Labs**

"The most complex model we actually understand"

https://youtu.be/D8GOeCFFby4

<!--
"Before we get started — a lot of the visuals tonight come from this incredible Welch Labs video called 'The most complex model we actually understand.' If you want to go deeper on anything I talk about tonight, this is the video. I highly recommend it."
-->

---
layout: image
image: /selected/22_trig_identity_reveal.jpg
backgroundSize: contain
---

<!--
Let the trig identity sit on screen for a second. Let people read it and groan.

"So... when was the last time you learned a trigonometric identity function?"

Pause. Let people laugh or groan.

"Yeah... same."

Beat.

"Yeah well neither did this clanker."

This should get a good laugh and set the tone — we're going to talk about something surprising, but we're not going to be stuffy about it.
-->

---
layout: image
image: /selected/02a_perceptron_board.jpg
backgroundSize: contain
---

<!--
"Hello everyone, it's great to see your beautiful faces. My name is Patrick and tonight we're going to talk about how AI learns."

"We're going to look at how it actually works under the hood, how it 'learns' — in quotes — and what we can learn from watching it do that."

Point at the perceptron board on screen:
"This is a perceptron. It's a single artificial neuron built as a physical machine back in the 1950s. Switches on the left are inputs, those dials in the middle are the weights, and the meter on the right is the output. That's it. That's an AI. Well — one neuron of one."

"We're going to start simple and build up from here."
-->

---
layout: image
image: /selected/07_stranger_in_strange_land.jpg
backgroundSize: contain
---

<!--
"So — grokking. You might be wondering what the word means."

"It comes from this book — Stranger in a Strange Land by Robert Heinlein, 1961. The main character is a human raised on Mars, and he uses the Martian word 'grok' throughout the book."

"It means to understand something so deeply that you merge with it and it merges with you. Not just knowing the answer — actually understanding the structure of the problem."

"Keep that in mind. We're going to see a model go from memorizing answers to — arguably — grokking the problem."
-->

---
layout: two-cols
---

## What does a model see?

We say "1 + 2 = 3"

The model sees patterns of switches flipped on and off.

We call it "1 + 2 = 3" — the model just sees **patterns mapped to patterns**.

::right::

<img src="/selected/02c_model_perspective.jpg" class="h-80 mx-auto mt-12" />

<!--
"So here's the problem 1 + 2 = 3. That's what WE see. We see numbers, an operation, and an answer."

"But the model? It doesn't know what 1 means. It doesn't know what addition is. It doesn't know what equals means."

Point at the strips on the left:
"Each number gets its own column of dots. To say '1', you light up the dot in the 1 position and leave everything else off. To say '2', you light up the 2 position. The equals sign gets its own column too."

"So the problem '1 + 2 =' — to the model — is just: second dot on, third dot on, last dot on. That's it. Three columns of mostly-off switches with one lit up in each."

Point at the output on the right:
"And the output? Same deal. The model lights up the dot in the 3 position. We look at that and go 'it got the answer right!' But the model has no idea it just did math. It matched an input pattern to an output pattern. That's all it knows how to do."

"Now imagine someone hands you thousands of these patterns — just dots on and dots off — and says 'figure out the rule.' That's what training a model is."
-->

---
layout: image
image: /selected/02d_llama_capital_france.jpg
backgroundSize: contain
---

<!--
"And this is how the real thing works too."

"When you type 'the capital of France is' into ChatGPT — each word gets turned into one of these patterns. 'The' lights up switch 791. 'Capital' lights up switch 6864. And so on."

"The strips are way bigger — Llama here has 128,000 positions instead of 6 — but it's the exact same idea. One dot lit up per word."

"And the model's job? Light up the right dot on the output. In this case — the dot for 'Paris.'"

"It doesn't know what France is. It doesn't know what a capital is. It doesn't know what Paris is. It found a pattern in the training data that says: when these input dots are lit, this output dot should be lit. That's it."
-->

---
layout: two-cols
---

## The Perceptron

A single artificial neuron as a physical machine.

- **Switches** = inputs
- **Dials** = weights
- **Meter** = output

"Weights" are just dials — numbers that control how much each input matters.

Welch Labs — [The Perceptron](https://www.youtube.com/watch?v=l-9ALe3U-Fg)

::right::

<img src="/selected/02a_perceptron_board.jpg" class="h-80 mx-auto mt-12" />

<!--
"So let's talk about how a model actually learns to match these patterns."

"Back to our perceptron. Switches on the left — those are your inputs. They're either on or off. Dials in the middle — those are the weights. And the meter on the right — that's the output."

"Each dial controls how much that particular input matters to the final answer. Turn a dial up, and that input has more influence. Turn it down, less influence. Set it to zero, and that input is completely ignored."

"A modern model like ChatGPT? It's roughly 100 million of these neurons wired together, with the dials turned automatically instead of by hand. But the principle is the same."

"There's another great Welch Labs video on the perceptron if you want to dig into the details — link is on the slide."
-->

---
layout: two-cols
---

## How does it learn?

Wrong answer? **Turn the dials.**

Right answer? **Leave them alone.**

Repeat a billion times.

::right::

<img src="/selected/02e_perceptron_labeled.jpg" class="h-80 mx-auto mt-12" />

<!--
"The learning algorithm is dead simple. You can see the four cases here."

"Case 1: you wanted a positive answer but the meter reads negative. You got it wrong. Turn the dials."

"Case 2: you wanted a negative answer but the meter reads positive. Also wrong. Turn the dials the other way."

"Cases 3 and 4: you got the right answer already. Don't touch anything."

"That's it. That's the whole algorithm. Try, check, nudge. Try, check, nudge. Over and over."

"Think of it like adjusting a shower. Too hot? Turn down the hot knob. But the key is — you turn it proportionally to how far off you are. If it's scalding, you crank it down. If it's just a little warm, you barely nudge it. That proportional adjustment is what makes this work."

"Frank Rosenblatt figured this out in 1957. And backpropagation — what we use to train every model today — is basically this same idea, just extended through multiple layers of neurons."
-->

---
layout: image
image: /selected/02b_model_one_hot_encoding.jpg
backgroundSize: contain
---

<!--
"So to recap — a model sees patterns of dots, not numbers or words. It learns by turning dials until the output matches what we want. There's no understanding happening. It's pure optimization."

"It's easy to forget that the symbols we assign to a model's inputs and outputs have this extra meaning that WE attach to them. But to the model, they're just patterns of inputs and patterns of outputs."

"It doesn't 'learn' the way you learn. It finds patterns. We're the ones who assign meaning to those patterns."

Pause. Let that land.

"So with that in mind... let me show you something weird."
-->

---
layout: two-cols
---

## Modular Arithmetic

"What's 10 + 5 on a 12-hour clock?"

**3.**

When the number hits the max, it wraps around.

That's modular math.

You already do this every time you read a clock.

::right::

<img src="/selected/09_clock_modular_math.jpg" class="h-80 mx-auto mt-12" />

<!--
"Before I show you the weird thing, I need to explain one concept: modular arithmetic."

"Don't let the name scare you — you already know this. What's 10 + 5 on a 12-hour clock? It's 3. Not 15 — 3. Because when you hit 12, the clock wraps back around to 1."

"That's all modular arithmetic is. You pick a number — the modulus — and when your result hits that number, it wraps around. Like a clock."

"You do this every single day. 'It's 11am, the meeting is in 3 hours — that's 2pm.' You just did modular arithmetic."

"Keep this in your head — it's going to be important."
-->

---

## The Experiment

Researchers trained a tiny model on modular addition.

All the combinations of A + B mod 113. Held back some for testing.

<img src="/selected/01_modular_addition_table.jpg" class="h-72 mx-auto" />

<!--
"So in 2021, a team at OpenAI set up a simple experiment."

"They took a tiny neural network — way smaller than anything you'd use in production — and trained it to do modular addition. Specifically, A plus B mod 113."

"That table on screen shows the dataset. Every combination of two numbers, added together, mod 113. They held back some of those combinations as a test set — problems the model would never see during training."

"The model's job: figure out the pattern. Learn to do this addition."

"Simple enough, right?"
-->

---

## The Result

100% accuracy on training examples — memorized the answers

~0% accuracy on testing examples — can't do new problems

<img src="/selected/03_training_curve_memorized.jpg" class="h-72 mx-auto" />

<!--
"And here's what happened. The blue line is accuracy on the training data — the problems it saw during training. It shoots up to 100%. The model memorized every single answer."

"But the yellow line — that's accuracy on the test set. The problems it hadn't seen. Flat. Basically zero."

"It's like a student who memorized every answer on the practice exam but can't solve a single new problem. It didn't learn addition — it learned a lookup table."

"If you're a machine learning engineer, you look at this and say 'classic overfitting' and you stop training. You're done. The model learned what it's going to learn."
-->

---

## The Plateau

Training: 100%. Testing: still flat at 0%.

Nothing improves. For a long time.

<img src="/selected/04c_grokking_complete.jpg" class="h-72 mx-auto" />

<!--
"And if you keep training... nothing happens. For a long time."

"The training accuracy stays at 100%. The test accuracy stays near zero. Step after step after step. Nothing changes."

"You would absolutely stop training here. There is zero indication that anything useful is going to happen. You'd be wasting compute."

"But here's the thing — one of the researchers at OpenAI accidentally left a model training while they went on vacation."

"They came back to something no one expected."
-->

---

## Then...

<img src="/selected/05_grokking_curve_labeled.jpg" class="h-96 mx-auto" />

<!--
Let the image do the work. Pause before speaking.

"...the test accuracy just... snaps to 100%."

"Not gradual. Not a slow improvement. It goes from basically zero to perfect in a tiny window of training steps."

Long pause. Let it sink in.

"It didn't get better. It changed strategies."

"It stopped memorizing... and started solving."

"Somewhere during all those flat training steps where nothing appeared to be happening — the model was quietly building something. And then it clicked."

"The researchers called this phenomenon 'grokking' — from the Robert Heinlein novel Stranger in a Strange Land. To grok something means to understand it so deeply that you merge with it."

"And they thought: what the hell is going on inside this model?"
-->

---

## So we popped the hood.

**Expected:** messy lookup behavior — a memorized table.

**Reality:**

<img src="/selected/11_neuron_evolution_forward.gif" class="h-72 mx-auto" />

<!--
"So researchers popped the hood. They looked inside the model at the actual neuron activations."

"What they expected to find was a mess — some kind of memorized lookup table. Just a bunch of arbitrary weights that happened to produce the right answers."

"What they actually found... watch this."

Let the gif play through once. It shows structure emerging from noise.

"You're watching the internal structure of the model evolve as it trains. It goes from complete noise — random scattered dots — to clean geometric patterns. Waves. Loops. Circles."

"Something organized is happening inside this model. Something nobody asked it to do."
-->

---

## Structure emerges

Waves. Loops. Circles.

<img src="/selected/11_neuron_scatter_plots.jpg" class="h-80 mx-auto" />

<!--
"Here's what the fully trained model looks like inside."

"These are actual neuron activations — the outputs of individual neurons plotted against each other. And look at the structure."

"Clean sine waves on the left. Those loops and circles on the right? Those are pairs of neurons plotted against each other, and they're forming perfect geometric shapes."

"This is where it gets weird. This is supposed to be a model that does addition. Why is it making circles?"

"This part broke my brain a little bit when I first saw it."
-->

---
layout: two-cols
---

## The model discovered something

Modular math wraps around... just like a clock.

The model learned to represent numbers as **positions on a circle.**

Addition = **rotation** around the circle.

::right::

<img src="/selected/10_clock_x_plus_y_rotation.jpg" class="h-80 mx-auto mt-8" />

<!--
"Remember the clock from earlier? Modular arithmetic wraps around — just like a clock face."

"Well... the model figured that out on its own."

"It learned to represent each number as a position on a circle. Not because anyone told it to — it discovered that this is a useful way to think about the problem."

"And addition? Addition becomes rotation around the circle. 10 + 5 on a clock? Start at 10, rotate 5 positions, land on 3. The model is doing the same thing — in its own internal representation."

"The model stops thinking in numbers and starts thinking in positions."

"Nobody programmed this. It's not in the training data. The model independently discovered what a clock already does physically."
-->

---

## Inside the model

Early layers: computes **cos** and **sin** of its inputs.

<img src="/selected/16_pipeline_cos_sin_xy.jpg" class="h-80 mx-auto" />

<!--
"Now let's get specific about what's actually happening inside the model."

"In the early layers — right after the input — the model converts its raw inputs into sine and cosine waves. You can see them here — cos of x, sin of x up top, and cos of y, sin of y on the bottom."

"Why sine and cosine? Because that's how you put a point on a circle. If you remember anything from trig class — or even if you don't — sine and cosine are just the x and y coordinates of a point on a circle."

"The model learned to put numbers on a circle. On its own."
-->

---

## Middle layers

Computes **products** of those functions — cos(kx) * cos(ky)

<img src="/selected/18_surface_cos_kx_cos_ky.jpg" class="h-80 mx-auto" />

<!--
"In the middle layers, things get more interesting. The model starts multiplying these trig functions together."

"What you're looking at is a 3D surface — the output of a single neuron as you vary both inputs x and y. And the dominant pattern is cos of x times cos of y."

"Now this might seem random. Why would multiplying cosines together help you do addition? Bear with me for two more slides."
-->

---

## The diagonal

A single neuron fires for every pair of inputs where **x + y = 65**.

It learned to **add**.

<img src="/selected/20_diagonal_stripes_with_sums.jpg" class="h-72 mx-auto" />

<!--
"This is where it clicks."

"Look at these diagonal stripes. Each stripe represents a set of input pairs where the neuron fires maximally."

"Look at the numbers along the top stripe: x=0 y=65, x=20 y=45, x=40 y=25, x=60 y=5, x=65 y=0. What do they all have in common?"

Pause. Let someone in the audience figure it out.

"They all add up to 65."

"This neuron fires for every pair of inputs whose sum is 65. It learned to detect addition. Not by adding — by geometry. By the orientation of a wave in its internal space."

"And the second stripe? Those pairs add to 178. But 178 mod 113 is... 65. Same answer, wrapped around."
-->

---

## The trig identity

**cos(kx)cos(ky) - sin(kx)sin(ky) = cos(k(x+y))**

A trigonometric identity converts products of trig functions into a **sum of the inputs**.

<img src="/selected/22_trig_identity_reveal.jpg" class="h-64 mx-auto" />

<!--
"And here's the punchline."

"Remember that scary equation from the first slide? This is it. This is what the model learned."

"There's a trick in math where if you multiply trig functions together in the right way, you get addition out the other side. It's a shortcut that's been in textbooks for centuries."

"The model found that shortcut. On its own. Nobody told it about trig. Nobody told it about this identity. It just turned out to be the most efficient way to solve the problem."

"So yeah... it reinvented trig."

Pause.

"It didn't learn addition. It learned a geometry."
-->

---

## The full picture

<img src="/selected/24_full_pipeline_with_identity.jpg" class="h-96 mx-auto" />

<!--
"Here's the whole pipeline. Numbers go in on the left, correct answers come out on the right."

"You don't need to follow every piece of this — the point is that every step has a purpose. The model built this entire process from scratch. It invented the representation, the strategy, and the math to make it work."

"Sine and cosine waves on the left. Those 3D surfaces in the middle. The trig identity up in the top right. All of it — discovered by the model."

"It learned a space where the problem becomes easy."
-->

---

## Why this is wild

- Not programmed to do this
- Not in the training data
- Appears late in training
- Hidden the whole time

<img src="/selected/23_training_evolution_noisy_to_clean.jpg" class="h-48 mx-auto" />

<!--
"So let's step back and think about what just happened."

"Nobody told this model about circles. Nobody told it about sine or cosine. Nobody told it about trig identities. None of that is in the training data — the training data is literally just 'what's 14 plus 87 mod 113.'"

"The model invented a geometric representation, discovered trigonometric functions, and applied a trig identity — all because that turned out to be an efficient way to solve the problem."

"And it did it late. For thousands of training steps it looked like nothing was happening. Any reasonable engineer would have stopped training. The capability was hidden the entire time."

"It looked dumb... until it didn't."
-->

---

## Meanwhile, at scale...

Anthropic found a **6-dimensional manifold** in Claude Haiku that handles line break arithmetic.

The same kind of geometric structure. In a production model.

<img src="/selected/25_anthropic_manifold_paper.jpg" class="h-64 mx-auto" />

<!--
"Now here's the part that should make you a little uncomfortable."

"Everything I just showed you was a tiny model. A single-layer transformer trained on a toy problem. You might think — okay, cute, but real models are different."

"Except... Anthropic recently published a paper where they looked inside Claude Haiku — a real, production language model — and found the same kind of thing."

"When Claude is writing text and needs to figure out when to insert a line break, it has to count how many characters it's written on the current line. And the way it does this? A six-dimensional geometric manifold. A curved surface in six-dimensional space where character count and line width are represented as positions on a helix."

"The same kind of geometric structure. In a production model with billions of parameters. Being used for something as mundane as line breaks."

"If a tiny model rediscovers trig to do addition... what are bigger models doing? What geometric structures are hiding inside GPT-4 or Claude that we haven't found yet?"

"What else is in there?"
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
"So that's grokking."

"Learning isn't linear. A model can look like it's doing nothing for a long time and then suddenly snap into a completely different strategy."

"Understanding — or something that looks a lot like understanding — can emerge suddenly and without warning."

"And models build internal structure that nobody asked for. Structure that turns out to be elegant, geometric, and mathematically sophisticated."

"We started tonight by saying AI is just pattern matching. Just statistics. Just optimization. And that's true — technically. But when the optimization discovers trigonometry on its own... maybe 'just' is doing a lot of heavy lifting in that sentence."

Pause.

"It looked dumb... until it didn't."

"Thank you."
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
