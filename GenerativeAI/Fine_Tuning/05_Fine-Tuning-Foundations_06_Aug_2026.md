# Part 2 – LLM Customization

# Chapter 05 - Fine-Tuning Foundations

> **Course:** GenAI Developer
>
> **Institute:** Quality Thought (QT)
>
> **Class:** 05
>
> **Date:** 06-Aug-2026

---

# 🧭 Learning Journey

```
Part 1

AI Models

↓

Evolution

↓

LLMs

↓

Transformers

↓

Inference

-----------------------------

Part 2

Fine-Tuning

↓

LoRA

↓

QLoRA

↓

PEFT

↓

Custom Models
```

---

# 🌍 Big Picture

Until now,

we have learned

how a Large Language Model

reads,

understands,

and generates text.

Now,

a new question arises.

> **Can we change the behaviour of a Large Language Model?**

The answer is

**Yes.**

This process is called

**Fine-Tuning.**

---

# Why Do We Need Fine-Tuning?

Imagine you ask

a general-purpose model

about medicine.

It can answer.

But now imagine

you want

an AI assistant

that only answers

questions related to

your hospital.

Or

**your company's HR policy.**

Or

**your banking system.**

How can we make

the model behave differently?

**This is exactly**

**what Fine-Tuning solves**.

---

## Real World Example

Think about

a new employee.

On the first day,

the employee has

general knowledge.

After joining your company,

the employee learns

your company's

- Rules
- Products
- Customers
- Internal Processes

The employee's behaviour changes.

Similarly,

Fine-Tuning changes

the behaviour

of a model.

---

### తెలుగు

Fine-Tuning అంటే

ఇప్పటికే Knowledge ఉన్న Model కి

కొత్త Knowledge

లేదా

కొత్త Behaviour

నేర్పించడం.

---

# What Makes a Model Intelligent?

This is one of the most important questions.

Inside every AI model,

there are

millions or billions of numbers.

These numbers are called

**Weights**.

---

# What are Weights?

## English Definition

Weights are numerical values inside a neural network that store the knowledge learned during training.

They determine

how the model responds

to different inputs.

---

### తెలుగు

Weights అంటే

Model లో ఉండే

Numbers.

ఈ Numbers లోనే

Model నేర్చుకున్న

Knowledge ఉంటుంది.

---

## Simple Analogy

Imagine

a student's brain.

```
Brain

↓

Knowledge
```

Now imagine

an AI model.

```
Weights

↓

Knowledge
```

Just as a student's knowledge

is stored

in the brain,

an AI model's knowledge

is stored

in its weights.

---

# Important Observation

Changing

Weights

↓

Changes

Behaviour.

This is the key idea

behind Fine-Tuning.

---

# Trainer Note

Trainer said

```
Model

↓

Weights

↓

Adjust Weights

↓

Behaviour Changes
```

This is the central concept

of today's class.

---

# Think Like an Engineer

Question

Suppose

ChatGPT

suddenly starts

answering

every question

in Telugu.

What changed?

---

## Answer

The behaviour changed

because

the model's parameters

(weights)

were adjusted

during training

or fine-tuning.

---

# Encoder and Decoder (Quick Recap)

Before learning

Fine-Tuning,

the trainer reviewed

Encoders

and

Decoders.

---

# Encoder

Purpose

↓

Understand

Input

```
Question

↓

Meaning
```

Encoders are commonly used

for

- Search
- Classification
- Embeddings
- Sentiment Analysis

---

# Decoder

Purpose

↓

Generate

Output

```
Prompt

↓

Next Token

↓

Next Token

↓

Response
```

Modern LLMs

such as

- GPT
- Claude
- Llama
- Qwen
- Gemma

are

primarily

**Decoder-Only Models.**

---

## Why Decoder Only?

Because

their main task

is

Text Generation.

They continuously predict

the next token

until

```
<EOS>
```

is generated.

---

### తెలుగు

GPT వంటి Models

కొత్త Text Generate చేయాలి.

అందుకే

వాటికి

Decoder మాత్రమే

చాలా సందర్భాల్లో సరిపోతుంది.

---

# Memory Trick

```
Encoder

↓

Understand

-----------------

Decoder

↓

Generate
```

Remember

> Encoder understands.

> Decoder generates.

---

# Image Generation

The trainer briefly introduced

Image Generation.

Instead of processing

words,

image models

process

small pieces

called

**Patches.**

---

# What is a Patch?

Imagine

this picture.

```
🖼️
```

Instead of looking

at the entire image,

the model divides it into

many small squares.

```
□□□□

□□□□

□□□□
```

Each small square

is called

a

**Patch.**

---

## Why Patches?

Exactly like

Text

↓

Tokens

Images

↓

Patches

Both are converted

into embeddings

before entering

the Transformer.

---

# Comparison

| Text Model | Image Model |
|------------|-------------|
| Sentence | Image |
| Tokens | Patches |
| Embeddings | Patch Embeddings |
| Transformer | Transformer |

---

# Memory Trick

```
Text

↓

Tokens

-------------------

Image

↓

Patches
```

---

# Chapter Summary

Today we learned

the beginning

of Fine-Tuning.

We discovered

that

a model's behaviour

depends on

its

Weights.

To change behaviour,

we must

adjust those weights.

There are

two approaches.

```
Option 1

↓

Full Fine-Tuning

-------------------

Option 2

↓

Train Small Matrices

↓

LoRA
```

The next chapter

will explain

why

LoRA became

one of the biggest breakthroughs

in LLM Fine-Tuning.

---

# 📚 Abbreviations

| Abbreviation | Full Form |
|--------------|-----------|
| LLM | Large Language Model |
| GPT | Generative Pre-trained Transformer |
| EOS | End Of Sequence |
| AI | Artificial Intelligence |

---

# 📖 Vocabulary (English → Telugu)

| English | Telugu |
|----------|---------|
| Fine-Tuning | ప్రత్యేక అవసరానికి మోడల్‌ను తిరిగి శిక్షణ ఇవ్వడం |
| Weight | మోడల్‌లో జ్ఞానాన్ని నిల్వ చేసే సంఖ్య |
| Behaviour | ప్రవర్తన |
| Encoder | ఇన్‌పుట్‌ను అర్థం చేసుకునే భాగం |
| Decoder | అవుట్‌పుట్‌ను రూపొందించే భాగం |
| Patch | చిత్రం యొక్క చిన్న భాగం |
| Parameter | మోడల్‌లోని శిక్షణ పొందిన విలువ |

---

# 🎯 Quick Revision

Remember these six ideas:

✅ A model's knowledge is stored in its **weights**.

✅ Changing weights changes the model's behaviour.

✅ Fine-Tuning adjusts model weights for a new task or domain.

✅ Encoder models focus on understanding input.

✅ Decoder models focus on generating output.

✅ Images are divided into **patches**, just as text is divided into **tokens**.