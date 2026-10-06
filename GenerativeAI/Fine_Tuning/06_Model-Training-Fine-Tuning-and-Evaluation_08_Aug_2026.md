# Chapter 06 - Model Training, Fine-Tuning and Evaluation

> **Course:** Gen-AI Developer
>
> **Institute:** Quality Thought (QT)
>
> **Class:** 06
>
> **Date:** 08-Aug-2026

---

# 🧭 Learning Journey

```text
Chapter 01
Introduction to AI Models
        ↓
Chapter 02
Evolution of Large Language Models
        ↓
Chapter 03
Inside a Large Language Model
        ↓
Chapter 04
Transformer Architecture
        ↓
Chapter 05
Fine-Tuning Foundations
        ↓
Chapter 06
Model Training, Fine-Tuning and Evaluation
```

---

# 🌍 Big Picture

In the previous class, we learned:

```text
Model
   ↓
Weights
   ↓
Change Weights
   ↓
Change Model Behaviour
```

We also learned that instead of changing all the weights, techniques such as **LoRA** can train a much smaller number of additional parameters.

Now we need to understand:

> **How does a model actually become knowledgeable, learn to follow instructions, and get evaluated during fine-tuning?**

The trainer introduced a typical three-stage training journey:

```text
Stage 1
Base Model

↓

Stage 2
Instruct Model

↓

Stage 3
RLHF / Alignment
```

---

# 1️⃣ Stage 1 — Base Model

## What is a Base Model?

A Base Model is a model that has been pretrained on a very large amount of data to learn language patterns and predict the next token.

The core objective is:

```text
Previous Tokens
      ↓
Predict
      ↓
Next Token
```

---

## Example

Suppose the training text is:

```text
The capital of France is Paris.
```

The model may see:

```text
The capital
```

and learn to predict:

```text
of
```

Then:

```text
The capital of
```

↓

```text
France
```

Then:

```text
The capital of France
```

↓

```text
is
```

And eventually:

```text
The capital of France is
```

↓

```text
Paris
```

This process is repeated over enormous amounts of training data.

---

## Important Point

The Base Model is not initially being taught:

```text
"Be a helpful assistant."

"Follow this user's instruction."

"Answer in this format."
```

Its fundamental pretraining objective is learning language patterns through **next-token prediction**.

---

### తెలుగులో

Base Model అంటే

చాలా పెద్ద మొత్తంలో Data పై Training చేసి,

Language Patterns నేర్చుకుని,

తర్వాత వచ్చే Token ని Predict చేయడం నేర్చుకున్న Model.

```text
Previous Tokens
      ↓
Next Token Prediction
```

---

# 🧠 Think Like an Engineer

Question:

### What is the main learning objective of a Base Model?

Answer:

> **Predict the next token based on the tokens that came before it.**

Remember this.

---

# 2️⃣ Stage 2 — Instruct Model

A Base Model knows language very well.

But knowing language is not the same as knowing how to behave like an assistant.

For example, a Base Model may be given:

```text
Explain Kubernetes in simple words.
```

Instead of directly answering,

it may continue the text in a way that resembles its training data.

We want the model to understand:

> "The user is giving me an instruction. I should follow it."

This is where **Supervised Fine-Tuning** comes in.

---

# SFT

## Full Form

**SFT = Supervised Fine-Tuning**

---

## What is Supervised Fine-Tuning?

Supervised Fine-Tuning is a training process where a pretrained model is trained on examples containing desired inputs and outputs.

Simple structure:

```text
Instruction
     ↓
Expected Response
```

The model learns:

```text
User asks something

↓

Generate an appropriate response
```

---

## Example Dataset

```text
Instruction:
Explain Kubernetes in simple words.

Expected Response:
Kubernetes is a system used to manage
containerized applications.
```

Another example:

```text
Instruction:
Convert 10 USD to INR.

Expected Response:
The value depends on the current
exchange rate.
```

The model sees many such examples.

It learns the desired behavior.

---

### తెలుగు

Supervised Fine-Tuning అంటే

Model కి

**Question / Instruction**

మరియు

**Expected Answer**

ఇచ్చి,

సరైన విధంగా Response ఇవ్వడం నేర్పించడం.

```text
Instruction
     ↓
Expected Answer
     ↓
Training
     ↓
Better Instruction Following
```

---

# Base Model vs Instruct Model

| Base Model | Instruct Model |
|---|---|
| Learns language patterns | Learns language + instruction following |
| Next-token prediction | Trained on instruction-response examples |
| General pretrained model | Assistant-oriented model |
| Not necessarily optimized for conversation | Designed to follow user instructions |

---

## Memory Trick

```text
Base Model

↓

Learn Language

Instruct Model

↓

Learn How to Follow Instructions
```

---

# 3️⃣ Stage 3 — RLHF

The trainer introduced:

**RLHF**

---

## Full Form

**RLHF = Reinforcement Learning from Human Feedback**

---

## Why Do We Need RLHF?

Suppose a model has two possible responses.

### Response A

```text
Helpful
Clear
Safe
Relevant
```

### Response B

```text
Confusing
Unhelpful
Unsafe
Irrelevant
```

We want the model to prefer Response A.

Human feedback can be used to teach the model which types of responses are preferred.

---

## Simple Concept

```text
Model generates responses

↓

Humans provide feedback

↓

Preferred responses are identified

↓

Training / optimization

↓

Model learns preferred behavior
```

---

### తెలుగు

RLHF అంటే

Model ఇచ్చే Responses పై

మనుషులు Feedback ఇవ్వడం.

ఏ Response మంచిది?

ఏది ఉపయోగకరంగా ఉంది?

ఏది Safe?

ఏది Relevant?

అనే Preferences ఆధారంగా

Model Behaviour ని మెరుగుపరచడం.

---

# Important Clarification

RLHF does **not** simply mean:

> "Make the model answer the right questions."

A better understanding is:

> **Use human preferences to improve the model's behavior and alignment with desired responses.**

This can include qualities such as:

- Helpfulness
- Relevance
- Safety
- Following preferences
- Appropriate behavior

---

# Three-Stage Mental Model

Now connect everything.

```text
                 MODEL TRAINING

                       │
                       ▼

              ┌─────────────────┐
              │   Base Model    │
              │                 │
              │ Learn Language  │
              │ Next Token      │
              └────────┬────────┘
                       │
                       ▼
              ┌─────────────────┐
              │ Instruct Model  │
              │                 │
              │ Follow Human    │
              │ Instructions    │
              └────────┬────────┘
                       │
                       ▼
              ┌─────────────────┐
              │ RLHF / Alignment│
              │                 │
              │ Improve Desired │
              │ Behaviour       │
              └─────────────────┘
```

---

# 4️⃣ Fine-Tuning with Unsloth

The trainer then introduced:

> **Unsloth for Fine-Tuning**

The goal is:

```text
Existing Model
      ↓
Fine-Tuning
      ↓
Changed / Specialized Behaviour
```

Unsloth provides tools that make fine-tuning supported models more efficient and easier to perform.

---

# What Do We Need for Fine-Tuning?

Your trainer listed four important things:

```text
1. Dataset

2. Hyperparameters

3. Metrics

4. Training Steps
```

Let's understand each one.

---

# 5️⃣ Dataset

A Dataset contains the examples used to train or fine-tune the model.

Example:

```text
Question
    ↓
Expected Answer
```

For an instruction-following model:

```text
User:
Explain Docker.

Assistant:
Docker is a platform...
```

---

## Why Dataset Quality Matters

Suppose your dataset contains:

```text
Bad Answers
Incorrect Information
Duplicate Examples
Poor Formatting
```

The model may learn undesirable patterns.

Therefore:

> **Good fine-tuning starts with good data.**

---

### తెలుగు

Dataset ఎంత Quality గా ఉంటే,

Model Learning కూడా అంత మంచిగా ఉండే అవకాశం ఉంటుంది.

```text
Good Data
   ↓
Better Learning

Bad Data
   ↓
Poor Learning
```

---

# 6️⃣ Hyperparameters

## What are Hyperparameters?

Hyperparameters are settings chosen by the developer before or during training that control how the training process works.

They are **not learned automatically in the same way as model weights**.

Examples:

```text
Rank
Alpha
Learning Rate
Batch Size
Gradient Accumulation Steps
Epochs
```

---

# 7️⃣ LoRA Rank

The trainer mentioned:

> Rank is generally a multiple of 8.

In LoRA,

**Rank (`r`)** controls the capacity/size of the low-rank adapter.

Example:

```text
r = 8
r = 16
r = 32
r = 64
```

Higher rank generally means:

```text
More trainable parameters
        +
More memory
        +
More adapter capacity
```

But higher is **not automatically better**.

---

## Memory Trick

```text
LoRA Rank

↓

Adapter Capacity
```

---

# 8️⃣ Alpha

The trainer mentioned:

> Alpha is a scaling factor.

In LoRA,

`alpha` controls the scaling applied to the LoRA update.

A commonly used starting relationship is:

```text
alpha ≈ 2 × rank
```

For example:

```text
rank = 16

alpha = 32
```

This is a **starting guideline**, not a universal rule.

---

# 9️⃣ Learning Rate

Learning Rate controls

> **How much the model's trainable parameters are updated during each training step.**

Imagine learning to ride a bicycle.

Very small adjustments:

```text
Slow learning
```

Very large adjustments:

```text
You may lose control
```

Similarly,

learning rate that is too high can make training unstable.

Learning rate that is too low can make training very slow.

---

### Simple Mental Model

```text
Learning Rate

Low
 ↓
Small Updates

High
 ↓
Large Updates
```

---

# 🔟 Batch Size

Batch Size means:

> **How many training examples are processed together before calculating an update.**

Example:

```text
Batch Size = 4
```

means four examples are processed together in one batch.

---

## Why Does Batch Size Matter?

Larger batch:

```text
More GPU Memory
```

Smaller batch:

```text
Less GPU Memory
```

Batch size affects training speed and memory requirements.

---

# 1️⃣1️⃣ Gradient Accumulation Steps

Sometimes your GPU cannot fit a large batch.

Suppose you want an effective batch size of:

```text
16
```

But your GPU can only process:

```text
4
```

examples at once.

You can accumulate gradients across multiple small batches.

For example:

```text
Batch = 4

Gradient Accumulation = 4

Effective Batch Size ≈ 16
```

Conceptually:

```text
Small Batch
   ↓
Calculate Gradient
   ↓
Accumulate
   ↓
Small Batch
   ↓
Accumulate
   ↓
Update Model
```

---

### Telugu

GPU Memory తక్కువగా ఉన్నప్పుడు

పెద్ద Batch ని ఒకేసారి Process చేయలేకపోవచ్చు.

అప్పుడు చిన్న Batches యొక్క Gradients ని

కొన్ని Steps వరకు Accumulate చేసి,

తర్వాత Model Parameters ని Update చేయవచ్చు.

---

# 1️⃣2️⃣ Epoch

An Epoch means:

> **One complete pass through the training dataset.**

Suppose your dataset contains:

```text
1,000 examples
```

One epoch means:

```text
All 1,000 examples

↓

Processed once
```

If:

```text
epochs = 3
```

the model trains over the dataset three times.

---

## More Epochs ≠ Always Better

Too few:

```text
Underfitting
```

Too many:

```text
Potential Overfitting
```

---

# 1️⃣3️⃣ Metrics

Training isn't simply:

```text
Start Training
      ↓
Wait
      ↓
Done
```

We need to observe what is happening.

That's why we use **metrics**.

Your trainer specifically mentioned:

```text
Learning Loss

Validation Loss
```

---

# 1️⃣4️⃣ Learning Loss

Loss measures how far the model's predictions are from the desired training target.

In simple terms:

> **Lower loss generally means the model is doing better on the training data.**

Example:

```text
Step 1
Loss = 2.5

Step 2
Loss = 1.8

Step 3
Loss = 1.2

Step 4
Loss = 0.8
```

The model is learning the training data better.

---

# 1️⃣5️⃣ Validation Loss

Training Loss tells us how well the model performs on the training data.

But we also want to know:

> **Can the model perform well on examples it did not train on?**

That's where validation data comes in.

Validation Loss measures performance on a separate validation dataset.

---

# Learning Loss vs Validation Loss

```text
Training Dataset
       ↓
Learning Loss

Validation Dataset
       ↓
Validation Loss
```

This distinction is extremely important.

---

# 1️⃣6️⃣ Understanding the Loss Curves

The trainer gave this exercise:

> "Express in a tabular form how to respond to different behaviors of learning loss and validation loss during fine-tuning using Unsloth."

This is an excellent exercise.

Here is the mental model you should learn.

| Learning Loss | Validation Loss | What It May Mean | What You Should Consider |
|---|---|---|---|
| ↓ | ↓ | Training and generalization improving | Continue monitoring |
| ↓ | → | Training improves but validation stops improving | Watch for overfitting |
| ↓ | ↑ | Training improves but unseen-data performance worsens | Possible overfitting |
| → | → | Little learning happening | Check learning rate, data, setup |
| High | High | Model is not learning sufficiently | Review data and hyperparameters |
| Very low | High | Possible memorization / overfitting | Reduce epochs, review dataset, regularization |
| High | ↓ | Unusual situation; inspect data/metrics | Check pipeline and evaluation setup |

**Important:** Don't react to a single loss value. Look at the **trend over multiple steps/epochs**.

---

# 1️⃣7️⃣ Overfitting

This is one of the most important ideas in fine-tuning.

Suppose:

```text
Training Loss

↓

Very Low
```

but:

```text
Validation Loss

↓

Increasing
```

The model may be learning the training examples too specifically.

This is called:

**Overfitting.**

---

## Analogy

Imagine a student memorizes exactly 100 questions.

Exam contains the same 100 questions:

```text
100/100
```

But when the teacher changes the questions slightly:

```text
40/100
```

The student memorized instead of understanding.

That is similar to overfitting.

---

### Telugu

Training Data ని మాత్రమే

చాలా బాగా గుర్తుపెట్టుకుని,

కొత్త Data పై సరిగా Perform చేయకపోతే

దాన్ని Overfitting అంటాం.

---

# 1️⃣8️⃣ Underfitting

The opposite problem is:

```text
Training Loss

↓

Still High
```

The model hasn't learned enough from the training data.

This can happen when:

- Training is insufficient
- Learning rate is inappropriate
- Dataset is poor
- Model capacity is insufficient
- Training configuration needs adjustment

---

# 1️⃣9️⃣ The Fine-Tuning Loop

Now connect everything you've learned.

```text
                  Existing Model
                        ↓
                     Dataset
                        ↓
                  Tokenization
                        ↓
                 LoRA / QLoRA
                        ↓
                 Hyperparameters
                        ↓
                     Training
                        ↓
                ┌───────┴───────┐
                ↓               ↓
          Learning Loss    Validation Loss
                ↓               ↓
                └───────┬───────┘
                        ↓
                    Evaluate
                        ↓
              Adjust Configuration
                        ↓
                     Retrain
```

This is the basic fine-tuning workflow.

---

# 🧠 Think Like an Engineer

Imagine your training results are:

```text
Epoch 1

Training Loss     = 2.4
Validation Loss   = 2.1

Epoch 2

Training Loss     = 1.4
Validation Loss   = 1.3

Epoch 3

Training Loss     = 0.8
Validation Loss   = 1.1

Epoch 4

Training Loss     = 0.4
Validation Loss   = 1.6
```

What happened?

Training loss continued decreasing.

But validation loss started increasing.

This is a warning sign for:

> **Overfitting.**

---

# 🔬 Behind the Scenes

During fine-tuning:

```text
Dataset
   ↓
Tokenizer
   ↓
Input Tokens
   ↓
Model
   ↓
Prediction
   ↓
Compare with Target
   ↓
Calculate Loss
   ↓
Calculate Gradients
   ↓
Update Trainable Parameters
   ↓
Next Training Step
```

With LoRA:

```text
Base Model Weights
        ↓
      Frozen

LoRA Parameters
        ↓
     Trainable
        ↓
     Updated
```

This connects today's class directly to the previous class.

---

# 🔗 Connection to Previous Class

Class 05 taught:

```text
Model
 ↓
Weights
 ↓
Fine-Tuning
 ↓
LoRA
```

Class 06 adds:

```text
Fine-Tuning
 ↓
Dataset
 ↓
Hyperparameters
 ↓
Training
 ↓
Loss
 ↓
Validation
 ↓
Evaluation
```

So now you understand not only **what LoRA is**, but also **how we know whether the fine-tuning is working**.

---

# 🎯 Quick Revision

Remember these concepts:

### Base Model

```text
Learn Language
+
Next Token Prediction
```

### Instruct Model

```text
Base Model
+
Supervised Fine-Tuning
↓
Instruction Following
```

### RLHF

```text
Model Responses
+
Human Preferences
↓
Improved Behaviour / Alignment
```

### Fine-Tuning

```text
Existing Model
+
Dataset
+
Training
↓
Changed Behaviour
```

### Hyperparameters

```text
Rank
Alpha
Learning Rate
Batch Size
Gradient Accumulation
Epochs
```

### Metrics

```text
Learning Loss
Validation Loss
```

### Main Warning

```text
Training Loss ↓
Validation Loss ↑
        ↓
Possible Overfitting
```

---

# 📚 Abbreviations

| Abbreviation | Full Form |
|---|---|
| SFT | Supervised Fine-Tuning |
| RLHF | Reinforcement Learning from Human Feedback |
| LoRA | Low-Rank Adaptation |
| QLoRA | Quantized Low-Rank Adaptation |
| GPU | Graphics Processing Unit |
| LLM | Large Language Model |
| AI | Artificial Intelligence |

---

# 📖 Vocabulary — English → Telugu

| English | Telugu |
|---|---|
| Base Model | ప్రాథమిక మోడల్ |
| Instruct Model | సూచనలను అనుసరించే మోడల్ |
| Fine-Tuning | ప్రత్యేక అవసరానికి మోడల్‌ను శిక్షణ ఇవ్వడం |
| Supervised | పర్యవేక్షణతో కూడిన |
| Feedback | అభిప్రాయం |
| Alignment | అనుగుణీకరణ / సరైన ప్రవర్తనకు అనుసంధానం |
| Dataset | డేటా సమాహారం |
| Hyperparameter | శిక్షణ సెట్టింగ్ |
| Rank | LoRA సామర్థ్యాన్ని నియంత్రించే పరిమాణం |
| Scaling Factor | స్కేలింగ్ కారకం |
| Learning Rate | అభ్యాస రేటు |
| Batch | ఒకేసారి ప్రాసెస్ చేసే డేటా సమూహం |
| Gradient | మోడల్ అప్‌డేట్‌కు ఉపయోగించే గణిత దిశ / మార్పు సమాచారం |
| Epoch | మొత్తం Dataset పై ఒక పూర్తి Training Pass |
| Loss | మోడల్ తప్పిదాన్ని కొలిచే విలువ |
| Validation | కొత్త / చూడని డేటాపై పరీక్ష |
| Overfitting | Training Data కి అతిగా సరిపోవడం |
| Underfitting | Training Data ని తగినంతగా నేర్చుకోకపోవడం |
| Metric | పనితీరును కొలిచే ప్రమాణం |
| Behaviour | ప్రవర్తన |

---

# 🎓 Interview Questions

### Q1. What is a Base Model?

A pretrained model primarily trained to learn language patterns through next-token prediction.

### Q2. What is SFT?

Supervised Fine-Tuning trains a pretrained model using examples of desired inputs and outputs so that it learns instruction-following behavior.

### Q3. What is RLHF?

Reinforcement Learning from Human Feedback uses human preference information to improve a model's behavior and alignment.

### Q4. What is a hyperparameter?

A training configuration chosen by the developer rather than learned directly as a model weight.

### Q5. What is learning rate?

The learning rate controls the size of parameter updates during training.

### Q6. What is an epoch?

One complete pass through the training dataset.

### Q7. What is the difference between training loss and validation loss?

Training loss measures performance on training data, while validation loss measures performance on a separate validation dataset.

### Q8. What does decreasing training loss and increasing validation loss indicate?

It is a common sign of possible overfitting.

### Q9. Why use validation data?

To estimate how well the fine-tuned model generalizes to examples it wasn't trained on.

### Q10. What is the purpose of LoRA?

To fine-tune a model efficiently by freezing the base model and training a much smaller set of additional parameters.