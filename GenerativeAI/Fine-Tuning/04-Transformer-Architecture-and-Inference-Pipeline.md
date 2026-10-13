# Chapter 04 - Transformer Architecture and Inference Pipeline

> **Course:** GenAI Developer
>
> **Institute:** Quality Thought (QT)
>
> **Class:** 04
>
> **Date:** 03-Aug-2026

---

# 🧭 Learning Journey

```
Chapter 01

Introduction to AI Models

        │

        ▼

Chapter 02

Evolution of Large Language Models

        │

        ▼

Chapter 03

Inside a Large Language Model

        │

        ▼

Chapter 04

Transformer Architecture and Inference Pipeline

        │

        ▼

Chapter 05

Prompt Engineering (Upcoming)
```

---

# 🌍 Big Picture

In the previous chapter we learned

```
Sentence

↓

Tokenizer

↓

Tokens

↓

Embeddings

↓

Transformer

↓

Probability Distribution

↓

Next Token
```

But we didn't see **what actually happens inside the Transformer.**

This chapter explains every stage.

---

# What Does a Transformer Actually Do?

Imagine you ask ChatGPT:

```
Explain Democracy in simple words.
```

To us,

it looks like magic.

Within a few seconds,

we receive an answer.

But internally,

the Transformer performs many steps.

```
Input Text

↓

Tokenization

↓

Embeddings

↓

Self Attention

↓

Multi-Head Attention

↓

Softmax

↓

Probability Distribution

↓

Next Token

↓

Repeat

↓

Final Response
```

This pipeline is called

**Inference Pipeline**.

---

# What is Inference?

Before understanding Transformers,

we should understand one important word.

## Definition

**Inference** is the process of using a trained AI model to generate predictions or responses for new input.

Training teaches the model.

Inference uses the model.

---

### తెలుగులో

Training సమయంలో

Model నేర్చుకుంటుంది.

Inference సమయంలో

నేర్చుకున్న Knowledge ఉపయోగించి

కొత్త ప్రశ్నలకు సమాధానం చెబుతుంది.

---

## Real World Example

Think about a student.

```
Study

↓

Exam
```

Studying

↓

Training

Writing the exam

↓

Inference

The student is not learning during the exam.

He is using what he already learned.

Similarly,

LLMs do not learn while answering your prompt.

They use the knowledge already learned during training.

---

## Memory Trick

```
Training

↓

Learning

Inference

↓

Using
```

Remember

> **Training teaches. Inference answers.**

---

# Running a Transformer

The trainer demonstrated

how to run

a Transformer model

using Python.

```python
from transformers import AutoTokenizer, AutoModelForCausalLM
```

Before understanding the code,

let us understand

what each component means.

---

# Hugging Face Transformers

The package

```
transformers
```

is a Python library.

It provides ready-made classes

to download and run

pre-trained Transformer models.

Without this library,

running modern LLMs would be much more difficult.

---

### తెలుగు

Transformers Library అంటే

ముందే Train చేసిన Models ని

సులభంగా Download చేసి

Run చేయడానికి ఉపయోగించే

Python Library.

---

## Why Do We Need It?

Imagine buying a new car.

You don't build the engine yourself.

You simply drive it.

Similarly,

the Transformers library

gives us ready-made models.

We don't need to build the Transformer architecture from scratch.

---

# AutoTokenizer

The trainer used

```python
AutoTokenizer.from_pretrained(model_name)
```

---

## What is a Tokenizer?

A Tokenizer converts

human-readable text

into tokens

that the model understands.

---

### Example

Input

```
Explain Democracy in simple words.
```

Output

```
Explain

Democracy

in

simple

words

.
```

Internally,

each token is also assigned

a unique token ID.

Example

| Token | Token ID *(Example)* |
|--------|----------------------:|
| Explain | 15342 |
| Democracy | 9821 |
| in | 287 |
| simple | 6543 |
| words | 889 |
| . | 13 |

> **Note:** These IDs are examples. Every model has its own vocabulary and token IDs.

---

### తెలుగు

Tokenizer

మనం ఇచ్చిన Sentence ని

Model కి అర్థమయ్యే చిన్న చిన్న Tokens గా మార్చుతుంది.

తర్వాత

ప్రతి Token కి

ఒక ప్రత్యేక ID ఇస్తుంది.

---

## Input

```
Explain Democracy in simple words.
```

## Output

```
Tokens

+

Token IDs
```

---

## Memory Trick

```
Sentence

↓

Tokenizer

↓

Tokens
```

---

# AutoModelForCausalLM

The trainer used

```python
AutoModelForCausalLM
```

---

## What Does It Mean?

Let's break it down.

```
Auto

↓

Automatically chooses

the correct model class.

------------------------

Model

↓

The AI Model

------------------------

Causal

↓

Predicts

Next Token

------------------------

LM

↓

Language Model
```

Together,

```
AutoModelForCausalLM
```

means

> Automatically load a language model that generates text by predicting the next token.

---

### తెలుగు

AutoModelForCausalLM అంటే

తర్వాత వచ్చే Token ని Predict చేస్తూ

Text Generate చేసే

Language Model ని

Automatic గా Load చేసే Class.

---

## Why "Causal"?

The word

"Causal"

does **not** mean

Cause and Effect.

Here,

it means

the model predicts

future tokens

using only

the tokens that came before.

Example

```
The capital of France is
```

The model predicts

```
Paris
```

using only the previous words.

It does not know future words.

---

## Trainer Note

Models like

- GPT
- Llama
- Qwen
- DeepSeek

are

**Causal Language Models**

because they generate text

one token at a time.

---

# Think Like an Engineer

Question

Why do we need

```
AutoTokenizer

and

AutoModel
```

Why can't we use only the model?

---

## Answer

Because

the model understands

only Token IDs.

Humans understand

sentences.

The Tokenizer acts as a translator

between humans and the model.

Without a Tokenizer,

the model cannot process

plain English.

---

# Memory Trick

```
Human

↓

Tokenizer

↓

Model

↓

Tokenizer

↓

Human
```

Tokenizer works

in both directions.

It converts

Text → Tokens

and later

Tokens → Text.

---

# Flow So Far

```
User

↓

Prompt

↓

AutoTokenizer

↓

Tokens

↓

Token IDs

↓

Embeddings

↓

Transformer

↓

Response
```

Everything up to this point

is preparing the input

for the Transformer.

In the next section,

we'll finally enter

the Transformer itself.

---

# Entering the Transformer

So far, our pipeline looks like this.

```
User Prompt

↓

Tokenizer

↓

Tokens

↓

Token IDs

↓

Embeddings

↓

Transformer
```

Now the embeddings enter the Transformer.

Inside the Transformer,

the first important task is

**understanding relationships between words.**

This is done using

**Self-Attention.**

---

# Why Was Self-Attention Invented?

Imagine someone says

```
Virat Kohli hit a century.

He was awarded Player of the Match.
```

Question

Who is

```
He
```

referring to?

As humans,

we immediately know

```
He

↓

Virat Kohli
```

But how does a computer know?

It must discover

the relationship between

different words in the sentence.

This is exactly

what Self-Attention does.

---

## Another Example

```
The cat sat on the mat because it was tired.
```

Question

Who was tired?

```
Cat

or

Mat?
```

Humans understand

```
It

↓

Cat
```

The Transformer also needs to learn this relationship.

Self-Attention makes that possible.

---

### తెలుగులో

Sentence లో

ఒక Word కి

ఇంకో Word కి

ఉన్న సంబంధాన్ని

గుర్తించడం

Self Attention యొక్క ముఖ్యమైన పని.

---

## Trainer Note

Transformer

↓

Understands relationships

↓

using

↓

Self Attention

Remember this.

---

# What is Self-Attention?

## Definition

Self-Attention is a mechanism that allows every token in a sentence to look at every other token and decide which ones are important.

Instead of processing words independently,

every word asks

```
Who should I pay attention to?
```

---

### తెలుగు

Self Attention అంటే

Sentence లోని

ప్రతి Token

మిగతా Tokens ని చూసి

ఏవి ముఖ్యమో

నిర్ణయించుకోవడం.

---

## Real World Analogy

Imagine

a classroom discussion.

Every student

listens

to every other student.

Finally,

each student decides

whose opinion is most important.

Exactly the same thing happens

inside Self-Attention.

---

## Visualization

Sentence

```
Virat

Kohli

hit

a

century

He
```

Self Attention

```
He

────────► Virat

────────► Kohli

────────► century

────────► hit
```

Notice

"He"

looks at

every word.

Then decides

which words are important.

---

## Memory Trick

```
Every Token

↓

Looks

↓

At Every Other Token
```

Remember

Self Attention

=

Looking around before deciding.

---

# Why "Self"?

Many beginners ask

Why is it called

Self Attention?

Because

the attention happens

**inside the same sentence.**

The model is

paying attention

to its own input.

Hence the name

Self Attention.

---

# Think Like an Engineer

Question

Without Self Attention,

what happens?

---

## Answer

The model sees words

individually.

It struggles to understand

relationships,

references,

and context.

Responses become

less accurate.

---

# Query, Key and Value (Q, K, V)

Now comes

the most famous concept

inside Transformers.

Every Token creates

three vectors.

```
Query

Key

Value
```

Every Token.

Not just one.

---

## Why Three Vectors?

Imagine

you enter a library.

You want

a Python book.

First,

you ask

```
Where is Python?
```

This is

**Query.**

The librarian checks

every shelf.

Each shelf has

a label.

Those labels are

**Keys.**

When the correct shelf is found,

you receive

the book.

That book is

**Value.**

Exactly the same idea

is used

inside the Transformer.

---

## Query (Q)

### English

Query represents

"What information am I looking for?"

---

### తెలుగు

Query అంటే

"నేను ఏ సమాచారాన్ని వెతుకుతున్నాను?"

అనే ప్రశ్న.

---

## Key (K)

### English

Key represents

"What information do I have?"

---

### తెలుగు

Key అంటే

"నా దగ్గర ఏ సమాచారం ఉంది?"

అనే గుర్తింపు.

---

## Value (V)

### English

Value represents

the actual information

that will be passed

to the next layer.

---

### తెలుగు

Value అంటే

చివరికి ఉపయోగపడే

అసలు సమాచారం.

---

# Simple Analogy

Imagine

College Library.

```
Student

↓

Query

↓

Shelf Number

↓

Key

↓

Book

↓

Value
```

---

## Another Analogy

Think about YouTube Search.

You search

```
Python Tutorial
```

Search Text

↓

Query

Videos

↓

Keys

Selected Video

↓

Value

---

## Important Point

Every Token

creates

its own

```
Query

Key

Value
```

So

if there are

10 Tokens,

the Transformer creates

```
10 Queries

10 Keys

10 Values
```

---

## Trainer Note

Q

K

V

are

not words.

They are

mathematical vectors

derived from embeddings.

We don't need the mathematics yet.

Understanding their purpose

is enough for now.

---

## Memory Trick

```
Query

↓

Searching

Key

↓

Matching

Value

↓

Information
```

Remember

Query asks.

Key matches.

Value answers.

---

# Multi-Head Attention

Suppose

five teachers

read the same essay.

Teacher 1

checks Grammar.

Teacher 2

checks Vocabulary.

Teacher 3

checks Creativity.

Teacher 4

checks Logic.

Teacher 5

checks Presentation.

Every teacher

looks at

the same essay

from

a different perspective.

Now combine

all their feedback.

The result is

much better

than using

only one teacher.

This is exactly

what Multi-Head Attention does.

---

## Definition

Instead of using

only one attention mechanism,

the Transformer

uses

multiple attention heads.

Each head

learns

different relationships

between words.

---

### తెలుగు

ఒకే విధంగా

Sentence ని చూడకుండా

చాలా విధాలుగా

విశ్లేషించడం

Multi Head Attention.

ప్రతి Head

వేర్వేరు Patterns నేర్చుకుంటుంది.

---

## Visualization

```
Sentence

↓

Head 1

↓

Grammar

----------------

Head 2

↓

Meaning

----------------

Head 3

↓

Context

----------------

Head 4

↓

Relationship

----------------

Combine

↓

Better Understanding
```

---

## Why Multi-Head?

One attention head

may focus

on grammar.

Another

may focus

on meaning.

Another

may focus

on sentence structure.

Combining them

creates

a richer understanding.

---

## Think Like an Engineer

Question

Why not use

only one Attention Head?

---

## Answer

One head

can only learn

limited relationships.

Multiple heads

capture

different kinds

of information

simultaneously.

That makes

the model

more intelligent.

---

## Memory Trick

```
One Head

↓

One View

Many Heads

↓

Many Views
```

Remember

Multi-Head Attention

means

looking at the same sentence

from multiple perspectives.

---

# Softmax

We have learned

```
Input

↓

Tokenizer

↓

Embeddings

↓

Self Attention

↓

Multi Head Attention
```

After all this processing,

the Transformer produces

**scores** for every possible token.

These scores are called

**Logits**.

However,

logits are not probabilities.

We need to convert them into probabilities.

This is where

**Softmax** comes into the picture.

---

## What is Softmax?

### Definition

Softmax is a mathematical function that converts raw scores (logits) into probabilities.

After applying Softmax,

all probabilities

- become positive
- add up to **1 (100%)**

---

### తెలుగు

Softmax అంటే

Transformer ఇచ్చిన Raw Scores ని

Probabilities గా మార్చే Mathematical Function.

Softmax తర్వాత

అన్ని Probabilities కలిపితే

1 (100%)

వస్తుంది.

---

## Why Do We Need Softmax?

Suppose the Transformer gives

```
Paris

8.2

London

3.1

Berlin

2.4
```

These are only scores.

Humans understand

probabilities better.

Softmax converts them into

| Token | Probability |
|--------|------------:|
| Paris | 95% |
| London | 3% |
| Berlin | 2% |

Now the model can choose the next token.

---

## Softmax Formula

```
             e^xi
P(xi) = -----------------
         Σ e^xj
```

Don't worry about memorizing the mathematics.

Just remember the purpose.

> **Softmax converts scores into probabilities.**

---

## Memory Trick

```
Scores

↓

Softmax

↓

Probabilities
```

---

# Probability Distribution

After Softmax,

the model has

a probability

for every possible token

in its vocabulary.

This is called

a

**Probability Distribution**.

---

## Example

Prompt

```
The capital of France is
```

The model calculates

| Token | Probability |
|--------|------------:|
| Paris | 96% |
| London | 2% |
| Berlin | 1% |
| Rome | 1% |

Notice something important.

The model has **not generated any text yet.**

It has only calculated probabilities.

---

### తెలుగు

Transformer

ముందుగా

ప్రతి Token కి

Probability లెక్కిస్తుంది.

తర్వాత మాత్రమే

ఒక Token ని ఎంచుకుంటుంది.

---

## Think Like an Engineer

Question

Does the model store

the answer

"Paris"

inside itself?

---

### Answer

No.

The model computes

probabilities

for every possible next token.

Then

it selects one

based on

the probability distribution

and settings like Temperature.

---

# Selecting the Next Token

Now comes the final decision.

The model selects

one token

from the probability distribution.

Example

```
Paris

96%
```

↓

Selected

That token becomes

the next word

in the response.

---

## Generation Loop

This process

does not happen

only once.

It repeats again

and again.

```
Sentence

↓

Next Token

↓

Sentence becomes longer

↓

Next Token

↓

Sentence becomes longer

↓

Repeat

↓

<EOS>

↓

Stop
```

This is called

**Autoregressive Text Generation.**

---

### తెలుగు

LLM

ఒక్కోసారి

ఒక్క Token మాత్రమే

Generate చేస్తుంది.

తర్వాత

ఆ కొత్త Token ని కూడా

Sentence లో కలిపి

మళ్లీ

Next Token Predict చేస్తుంది.

ఈ Process

<EOS>

వచ్చే వరకు

కొనసాగుతుంది.

---

# Complete Transformer Pipeline

```
User Prompt

↓

Tokenizer

↓

Tokens

↓

Token IDs

↓

Embeddings

↓

Self Attention

↓

Multi Head Attention

↓

Transformer Layers

↓

Logits

↓

Softmax

↓

Probability Distribution

↓

Temperature

↓

Next Token

↓

Repeat

↓

<EOS>

↓

Final Response
```

This is the complete inference pipeline of a modern LLM.

---

# Running a Small LLM

The trainer demonstrated

running

```
Qwen/Qwen3-0.6B
```

using

Hugging Face Transformers.

---

## What Happens Internally?

### Step 1

```python
from transformers import AutoTokenizer
```

Loads the tokenizer.

---

### Step 2

```python
AutoModelForCausalLM
```

Loads a text-generation model.

---

### Step 3

```python
prompt = "Explain Democracy..."
```

Creates the user prompt.

---

### Step 4

```python
tokenizer(...)
```

Converts

text

↓

tokens

↓

token IDs

---

### Step 5

```python
model.generate(...)
```

Runs

the complete Transformer pipeline

and generates

new tokens.

---

### Step 6

```python
tokenizer.decode(...)
```

Converts

generated token IDs

back into

human-readable text.

---

## Simple Flow

```
Prompt

↓

Tokenizer

↓

Transformer

↓

Generated Tokens

↓

Decode

↓

Response
```

---

# Running a Translation Model

The trainer also demonstrated

```
Helsinki-NLP/opus-mt-en-hi
```

This is

a Translation Transformer.

---

## Input

```
Hello, how are you?
```

---

## Output

```
नमस्ते, आप कैसे हैं?
```

---

## Important Observation

Both

Text Generation

and

Translation

use

Transformers.

The only difference

is

the task

for which they were trained.

---

# What Problems Do Transformers Solve?

Transformers are used in many applications.

---

## Text Generation

Example

ChatGPT

Gemini

Claude

---

## Translation

Example

English

↓

Hindi

English

↓

Telugu

---

## Summarization

Long document

↓

Short summary

---

## Sentiment Analysis

Input

```
The movie was fantastic.
```

Output

```
Positive
```

---

## Image Captioning

Input

Image

↓

Output

```
A boy is playing football.
```

---

## Memory Trick

```
Transformer

↓

Many Applications

↓

One Architecture
```

Remember

The same Transformer architecture

can solve many different tasks.

---

# Trainer Exercise

The trainer gave this prompt:

> "Explain every stage of the Transformer in plain English and show the output of each stage."

This is an excellent learning exercise because it forces you to understand

**what enters each stage** and **what comes out**.

---

# Stage-wise Summary

| Stage | Input | Output |
|--------|-------|--------|
| Tokenizer | Text | Tokens |
| Token IDs | Tokens | Numbers |
| Embeddings | Token IDs | Embedding Vectors |
| Self Attention | Embeddings | Context-aware Representations |
| Multi-Head Attention | Context-aware Representations | Richer Understanding |
| Transformer Layers | Hidden Representations | Logits |
| Softmax | Logits | Probability Distribution |
| Temperature | Probabilities | Token Selection Strategy |
| Decoder | Selected Token IDs | Human-readable Text |

---

# Chapter Summary

Today we learned

✅ How a Transformer processes text

✅ Why Self Attention is important

✅ Query, Key and Value

✅ Multi-Head Attention

✅ Softmax

✅ Probability Distribution

✅ Token Selection

✅ Autoregressive Generation

✅ Running a Transformer

✅ Running a Translation Model

✅ Transformer Applications

---

# Vocabulary

| English | Telugu |
|----------|---------|
| Logits | ప్రాబబిలిటీకి ముందు వచ్చే రా స్కోర్లు |
| Softmax | స్కోర్లను ప్రాబబిలిటీలుగా మార్చే ఫంక్షన్ |
| Probability Distribution | అన్ని టోకెన్లకు సంభావ్యత పంపిణీ |
| Decoder | టోకెన్ ఐడీలను టెక్స్ట్‌గా మార్చడం |
| Translation | అనువాదం |
| Summarization | సారాంశం |
| Sentiment | భావోద్వేగ విశ్లేషణ |
| Caption | చిత్రం వివరణ |
| Generation | టెక్స్ట్ రూపొందించడం |

---

# 🎯 Quick Revision

Remember these six ideas:

✅ Transformers process the entire sentence together.

✅ Self Attention helps each token understand other tokens.

✅ Q = Query, K = Key, V = Value.

✅ Softmax converts logits into probabilities.

✅ The model generates **one token at a time**.

✅ The same Transformer architecture powers text generation, translation, summarization, and many other AI tasks.
