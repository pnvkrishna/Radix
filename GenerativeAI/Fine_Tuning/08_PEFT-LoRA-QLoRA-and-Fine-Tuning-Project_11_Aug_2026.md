# Chapter 08 - PEFT, LoRA, QLoRA and Fine-Tuning Project

> **Course:** Gen-AI Developer
>
> **Institute:** Quality Thought (QT)
>
> **Class:** 08
>
> **Date:** 11-Aug-2026

---

# 01. PEFT, LoRA and QLoRA

This class introduces three closely related concepts:

```text
PEFT
 ↓
LoRA
 ↓
QLoRA
````

These are used to make Fine-Tuning more efficient.

---

# 02. PEFT

## Full Form

**PEFT = Parameter-Efficient Fine-Tuning**

---

## What is PEFT?

PEFT is a family of techniques that allow us to fine-tune a model by training only a small portion of parameters instead of updating all parameters of the original model.

Recall from the previous classes:

```text
Full Fine-Tuning

Model
 ↓
Update All / Most Parameters
```

This can require a lot of:

* GPU memory
* Compute
* Training time
* Storage

PEFT tries to reduce these requirements.

```text
PEFT

Base Model
 ↓
Keep most parameters frozen
 ↓
Train a small number of additional parameters
```

---

# 03. Why Do We Need PEFT?

Imagine we have a model with:

```text
7 Billion Parameters
```

With Full Fine-Tuning:

```text
7 Billion Parameters
        ↓
Need to update them
        ↓
Large memory requirement
```

With PEFT:

```text
7 Billion Parameter Model
        ↓
Freeze Base Model
        ↓
Train Small Additional Parameters
```

The number of trainable parameters becomes much smaller.

---

### Telugu

PEFT అంటే

మొత్తం Model Parameters ని train చేయకుండా,

చిన్న number of parameters ని మాత్రమే train చేసి

Model Behavior ని మార్చడం.

---

# 04. Important Terminology

Before going further, understand these three terms.

### Parameter

A learned value inside the model.

```text
Model
 ↓
Parameters
 ↓
Weights
```

### Trainable Parameter

A parameter that is allowed to change during training.

### Frozen Parameter

A parameter that is kept unchanged during training.

---

# 05. Full Fine-Tuning vs PEFT

| Full Fine-Tuning                  | PEFT                                         |
| --------------------------------- | -------------------------------------------- |
| Updates most/all model parameters | Updates a small subset/additional parameters |
| High memory requirement           | Lower memory requirement                     |
| More expensive                    | More efficient                               |
| Large optimizer state             | Smaller optimizer state                      |
| Large training cost               | Lower training cost                          |

---

# 06. LoRA

## Full Form

**LoRA = Low-Rank Adaptation**

LoRA is one of the most popular PEFT techniques.

The basic idea is:

> Instead of updating the original large weight matrix directly, freeze it and learn a low-rank update using two much smaller matrices.

---

# 07. Why is it Called Low-Rank?

Suppose the original weight matrix is:

```text
100 × 100
```

That contains:

```text
100 × 100 = 10,000
```

values.

Instead of learning a full:

```text
100 × 100
```

update,

LoRA represents the update using two smaller matrices.

For example:

```text
A = 100 × 3

B = 3 × 100
```

Their multiplication produces:

```text
(100 × 3) × (3 × 100)

        ↓

100 × 100
```

So:

```text
Original Weight Matrix
        ↓
      100 × 100

LoRA
        ↓
A = 100 × 3
B = 3 × 100
```

---

# 08. Parameter Comparison

Full matrix:

```text
100 × 100

= 10,000 parameters
```

LoRA matrices:

```text
100 × 3
+
3 × 100

= 300 + 300

= 600 parameters
```

So instead of learning:

```text
10,000
```

parameters,

we learn:

```text
600
```

parameters.

This is the basic intuition behind Low-Rank Adaptation.

---

# 09. LoRA Mathematical Idea

The original weight is:

```text
W
```

LoRA does not directly replace `W`.

Instead, the effective weight becomes approximately:

```text
W' = W + ΔW
```

where:

```text
ΔW = B × A
```

Therefore:

```text
W' = W + B × A
```

The important idea is:

```text
W
 ↓
Frozen

B × A
 ↓
Trainable
```

---

# 10. Simple LoRA Diagram

```text
                  Original Model
                       │
                       ▼
                Weight Matrix W
                       │
                    FROZEN
                       │
                       │
                       ├─────────────────┐
                       │                 │
                       │                 ▼
                       │            LoRA Adapter
                       │                 │
                       │          ┌──────┴──────┐
                       │          ▼             ▼
                       │       Matrix A      Matrix B
                       │          │             │
                       │          └──────┬──────┘
                       │                 │
                       │              B × A
                       │                 │
                       └────────┬────────┘
                                ▼
                         Updated Behaviour
```

---

# 11. Telugu Explanation of LoRA

Suppose Model లో ఒక పెద్ద Matrix ఉంది.

```text
100 × 100
```

దాన్ని మొత్తం train చేయకుండా,

రెండు చిన్న matrices తీసుకుంటాం.

```text
100 × 3

3 × 100
```

ఈ రెండు matrices ని train చేస్తాం.

Original Model weights:

```text
Frozen
```

LoRA matrices:

```text
Trainable
```

అందువల్ల training కి కావాల్సిన resources తగ్గుతాయి.

---

# 12. What Does LoRA Actually Learn?

This is very important.

LoRA does not learn a completely new model from scratch.

Instead:

```text
Pretrained Model
       +
LoRA Adapter
       ↓
Specialized Behaviour
```

The pretrained model retains its original capabilities.

The adapter learns task-specific changes.

---

# 13. QLoRA

## Full Form

**QLoRA = Quantized Low-Rank Adaptation**

QLoRA combines:

```text
Quantization
+
LoRA
```

---

# 14. What is Quantization?

Quantization means representing model values using fewer bits.

For example:

```text
32-bit
 ↓
16-bit
 ↓
8-bit
 ↓
4-bit
```

Fewer bits generally means less memory is required to store the model.

---

# 15. Why Quantization Helps

Imagine a model requires a large amount of GPU memory.

If we represent its parameters using fewer bits:

```text
Original Model
       ↓
Quantization
       ↓
Smaller Memory Footprint
```

This can make it possible to load and fine-tune larger models on more limited hardware.

---

# 16. QLoRA Mental Model

```text
                Original Model
                      ↓
                 Quantization
                      ↓
                Low-bit Model
                      ↓
             Freeze Base Model
                      ↓
                Add LoRA
                      ↓
             Train LoRA Matrices
```

So:

```text
LoRA

=
Frozen Base Model
+
Trainable LoRA Adapters
```

and:

```text
QLoRA

=
Quantized Base Model
+
Trainable LoRA Adapters
```

---

# 17. LoRA vs QLoRA

| Feature                            | LoRA                      | QLoRA         |
| ---------------------------------- | ------------------------- | ------------- |
| PEFT technique                     | Yes                       | Yes           |
| Base model frozen                  | Yes                       | Yes           |
| LoRA adapters                      | Yes                       | Yes           |
| Quantization                       | Not required              | Yes           |
| Lower memory than full fine-tuning | Yes                       | Yes           |
| Even lower memory usage            | Usually higher than QLoRA | Usually lower |

---

# 18. Important Concept

Do not confuse:

```text
LoRA
```

with:

```text
Quantization
```

They solve different problems.

### LoRA

Reduces the number of **trainable parameters**.

### Quantization

Reduces the number of **bits used to represent model parameters**.

### QLoRA

Combines both.

```text
LoRA
 ↓
Fewer parameters to train

Quantization
 ↓
Less memory for model representation

QLoRA
 ↓
Both
```

---

# 19. Today's Use Case

The trainer introduced a practical project.

We want to build:

> **A customer-service chatbot for a fictional pizza company called Pizza Palace.**

The chatbot should:

* Help customers
* Use a polite tone
* Follow a specific communication style
* Provide menu information
* Work with current menu information

---

# 20. Understanding the Requirement

There are two different requirements.

### Requirement 1 — Behavior

The chatbot should:

```text
Be polite

Use Pizza Palace style

Behave like customer support

Respond consistently
```

This is suitable for:

```text
Fine-Tuning
```

---

### Requirement 2 — Facts

The chatbot needs:

```text
Latest Menu

Latest Prices

Available Items
```

These can change.

This is better handled using:

```text
RAG / External Data
```

---

# 21. Important Design Principle

The trainer intentionally puts menu information inside the training examples but says:

> We want to tune the model for behavior, not for memorizing menu items.

This is an important concept.

The dataset uses the menu as **context** so that the model learns how a Pizza Palace assistant should behave.

We should not assume that fine-tuning is the correct mechanism for keeping prices permanently up to date.

---

# 22. System Prompt

The training example contains a System Prompt.

Conceptually:

```text
System
 ↓
Defines Assistant Role
 ↓
Defines Company
 ↓
Provides Context
```

Example:

```text
You are a helpful assistant at Pizza Palace
guiding customers.
```

Then the menu is supplied as context.

---

# 23. Why Use a System Prompt?

A system prompt can establish:

* Role
* Behavior
* Tone
* Rules
* Context

For example:

```text
You are a helpful assistant at Pizza Palace.
Respond politely and clearly.
Help customers choose pizzas.
```

This establishes the desired behavior.

---

# 24. Sample Conversation

The trainer provided an example.

### User

```text
What are options for Veg Today?
```

### Assistant

```text
Welcome to Pizza Palace,
We have wonderful options for you today.

Farmhouse – ₹239

Peppy Paneer – ₹239

Would you like to have
Cheese Burst Crust for ₹99
or Garlic Bread?
```

This example teaches the model things such as:

```text
Greeting
+
Polite Tone
+
Relevant Recommendations
+
Menu Formatting
+
Customer Engagement
```

---

# 25. What Does the Dataset Teach?

Suppose we provide hundreds of conversations.

The model may learn patterns such as:

```text
Customer asks question
        ↓
Greet customer
        ↓
Answer politely
        ↓
Provide relevant options
        ↓
Offer an additional choice
```

This is the behavior we want.

---

# 26. Why 300–400 Conversations?

The trainer suggested creating:

```text
300 – 400
```

conversational examples.

The goal is to provide enough examples representing the desired behavior.

Examples could cover:

```text
Pizza recommendations

Veg pizza questions

Non-veg pizza questions

Crust questions

Toppings

Dips

Desserts

Price questions

Polite greetings

Customer follow-up questions

Different customer styles
```

The important principle is:

> **Dataset diversity matters.**

---

# 27. Synthetic Dataset Generation

The trainer suggested using an LLM such as Claude to generate the conversations.

This is called:

**Synthetic Data Generation**

Conceptually:

```text
Human-designed Rules
        ↓
LLM
        ↓
Generate Conversations
        ↓
Review / Clean
        ↓
JSONL Dataset
```

---

# 28. Important Warning About Synthetic Data

Do not blindly generate 400 examples and immediately fine-tune.

Generated data should be checked for:

* Incorrect prices
* Duplicate conversations
* Repetitive wording
* Wrong menu items
* Incorrect behavior
* Contradictory responses
* Poor formatting

The quality of your training dataset strongly affects the resulting model.

---

# 29. JSONL Dataset

The final dataset might conceptually look like:

```json
{"messages":[{"role":"system","content":"You are a helpful Pizza Palace assistant..."},{"role":"user","content":"What are options for Veg today?"},{"role":"assistant","content":"Welcome to Pizza Palace..."}]}
{"messages":[{"role":"system","content":"You are a helpful Pizza Palace assistant..."},{"role":"user","content":"Do you have paneer pizza?"},{"role":"assistant","content":"Yes, we have Peppy Paneer..."}]}
```

One conversation:

```text
One JSON object
```

One line:

```text
One training example
```

---

# 30. Preparing for Tuning

Before training, the trainer asks:

> Where are we going to run the model?

The class mentions:

```text
Ollama
vLLM
```

These are primarily related to running/serving the model.

For the actual fine-tuning exercise, the trainer plans to use:

```text
Google Colab
```

---

# 31. Why Google Colab?

Google Colab provides a cloud-based notebook environment.

For this exercise:

```text
Your Computer
      ↓
Google Colab
      ↓
GPU
      ↓
Fine-Tuning
```

This is useful when your local machine does not have enough GPU memory.

---

# 32. Training vs Serving

Do not mix these two concepts.

### Training

```text
Google Colab
     ↓
Fine-Tune Model
```

### Serving

```text
Fine-Tuned Model
     ↓
Ollama / vLLM / Other Runtime
     ↓
Application
```

They are different stages.

---

# 33. Unsloth Notebook

The trainer's approach is:

> Take an existing Unsloth notebook and understand how it works, then adapt it to our model and dataset.

This is a good practical learning strategy.

Instead of writing everything from scratch:

```text
Existing Working Notebook
        ↓
Understand
        ↓
Change Model
        ↓
Change Dataset
        ↓
Change Configuration
        ↓
Run Fine-Tuning
```

---

# 34. Parameters and Metrics

Before training, we must understand the important parameters.

From previous classes:

```text
LoRA Rank

LoRA Alpha

Learning Rate

Batch Size

Gradient Accumulation Steps

Epochs
```

And metrics:

```text
Learning Loss

Validation Loss
```

We should know what each parameter controls before blindly changing values.

---

# 35. Dataset Split

The trainer said:

> Split the dataset into 80–20.

This means:

```text
100% Dataset
       ↓
┌──────────────┬──────────────┐
│              │              │
80%            20%
Training       Validation
Data           Data
```

---

# 36. Training Dataset

The 80% training data is used to actually update the trainable parameters.

```text
Training Dataset
      ↓
Model
      ↓
Loss
      ↓
Gradients
      ↓
Update LoRA Parameters
```

---

# 37. Validation Dataset

The remaining 20% is used to evaluate how the model performs on examples it did not train on.

```text
Validation Dataset
       ↓
Model
       ↓
Validation Loss
```

The model does not use validation examples to directly update its weights during normal training.

---

# 38. Why 80–20?

The idea is:

```text
80%
 ↓
Learn

20%
 ↓
Evaluate Generalization
```

The exact split is not a universal rule.

Other splits such as:

```text
90 / 10
80 / 20
70 / 30
```

can be used depending on the dataset and project.

For today's exercise, we follow the trainer's:

```text
80 / 20
```

---

# 39. Complete Project Flow

Now connect everything from today's class.

```text
                   Pizza Palace
                        ↓
                    Use Case
                        ↓
              Desired Behavior
                        ↓
                Sample Conversations
                        ↓
              Generate 300–400 Examples
                        ↓
                  Review Dataset
                        ↓
                    JSONL
                        ↓
                  80 / 20 Split
                  ↙          ↘
             Training       Validation
                ↓               ↓
             80% Data        20% Data
                ↓
            Google Colab
                ↓
             Unsloth
                ↓
           LoRA / QLoRA
                ↓
             Fine-Tuning
                ↓
        Learning / Validation Loss
                ↓
             Evaluation
                ↓
        Fine-Tuned Model / Adapter
                ↓
       ┌────────┴─────────┐
       ↓                  ↓
    Ollama              vLLM
       ↓                  ↓
     Local             Serving
```

---

# 40. Connection to Previous Classes

## Class 05

```text
Weights
 ↓
Fine-Tuning
 ↓
LoRA
```

## Class 06

```text
Dataset
 ↓
Hyperparameters
 ↓
Training
 ↓
Loss
 ↓
Validation
```

## Class 07

```text
Fine-Tuning vs RAG
 ↓
JSONL
 ↓
Open Models
 ↓
Ollama
 ↓
llama.cpp
 ↓
vLLM
```

## Class 08

```text
PEFT
 ↓
LoRA
 ↓
QLoRA
 ↓
Real Project
 ↓
Pizza Palace
 ↓
Dataset
 ↓
80/20 Split
 ↓
Google Colab
 ↓
Unsloth
 ↓
Fine-Tuning
```

---

# 41. 🎯 What You Must Understand From This Class

Do not memorize the Pizza Palace menu.

The menu is just the training example.

You need to understand these concepts:

### 1. PEFT

Train fewer parameters instead of the whole model.

### 2. LoRA

Use small low-rank matrices to learn the model update.

### 3. QLoRA

Combine quantization with LoRA.

### 4. Behavior

Fine-tuning can teach the model a particular style, tone, format, or task behavior.

### 5. Dataset

The model learns from examples.

### 6. JSONL

One JSON object per line.

### 7. Synthetic Data

An LLM can help generate training examples, but they must be reviewed.

### 8. Training / Validation Split

```text
80%
Training

20%
Validation
```

### 9. Google Colab

Used as the training environment for the exercise.

### 10. Unsloth

Used to make the fine-tuning workflow easier and more efficient.

---

# 42. Abbreviations

| Abbreviation | Full Form                         |
| ------------ | --------------------------------- |
| PEFT         | Parameter-Efficient Fine-Tuning   |
| LoRA         | Low-Rank Adaptation               |
| QLoRA        | Quantized Low-Rank Adaptation     |
| JSONL        | JSON Lines                        |
| LLM          | Large Language Model              |
| RAG          | Retrieval-Augmented Generation    |
| GPU          | Graphics Processing Unit          |
| CPU          | Central Processing Unit           |
| API          | Application Programming Interface |
| SFT          | Supervised Fine-Tuning            |
| VRAM         | Video Random Access Memory        |

---

# 43. Vocabulary — English → Telugu

| English        | Telugu                                                |
| -------------- | ----------------------------------------------------- |
| Parameter      | మోడల్‌లో శిక్షణ పొందిన విలువ                          |
| Trainable      | శిక్షణలో మార్చగలిగేది                                 |
| Frozen         | శిక్షణ సమయంలో మార్చకుండా ఉంచినది                      |
| Adapter        | మోడల్‌కు జోడించే చిన్న Trainable భాగం                 |
| Rank           | LoRA Adapter సామర్థ్యాన్ని నిర్ణయించే పరిమాణం         |
| Quantization   | తక్కువ bits ఉపయోగించి model values ను represent చేయడం |
| Matrix         | సంఖ్యల వరుసలు మరియు నిలువు వరుసల సమూహం                |
| Behavior       | ప్రవర్తన                                              |
| Dataset        | డేటా సమాహారం                                          |
| Conversation   | సంభాషణ                                                |
| Synthetic Data | AI ద్వారా రూపొందించిన డేటా                            |
| Training       | మోడల్‌కు నేర్పించే ప్రక్రియ                           |
| Validation     | కొత్త data పై మోడల్‌ను పరీక్షించడం                    |
| Split          | డేటాను భాగాలుగా విభజించడం                             |
| Inference      | శిక్షణ పొందిన మోడల్‌తో prediction చేయడం               |
| Serving        | Applicationలకు modelని అందించడం                       |
| Fine-Tuning    | ప్రత్యేక task/behavior కోసం modelని tune చేయడం        |
| Context        | మోడల్‌కు అందించిన నేపథ్య సమాచారం                      |
| Tone           | మాట్లాడే / స్పందించే విధానం                           |
| Style          | స్పందించే శైలి                                        |

---

# 44. Interview Questions

## Q1. What is PEFT?

PEFT stands for Parameter-Efficient Fine-Tuning. It is a family of methods that fine-tune a model by training only a small number of parameters instead of updating the entire model.

---

## Q2. What is LoRA?

LoRA stands for Low-Rank Adaptation. It freezes the original model weights and learns a low-rank update using small trainable matrices.

---

## Q3. Why is LoRA called Low-Rank Adaptation?

Because the weight update is represented using the product of smaller matrices with a low rank instead of directly training the full weight matrix.

---

## Q4. What is QLoRA?

QLoRA combines quantization with LoRA. The base model is loaded in a lower-bit representation while LoRA adapters are trained.

---

## Q5. What is the difference between LoRA and QLoRA?

LoRA uses adapter-based fine-tuning without requiring quantization. QLoRA adds quantization to reduce the memory required to load the base model.

---

## Q6. What is the purpose of quantization?

Quantization represents model parameters using fewer bits, reducing memory requirements.

---

## Q7. Why use an instruct model for this project?

Because the model is already trained to understand instructions and conversational interactions, making it a useful starting point for learning the Pizza Palace customer-support behavior.

---

## Q8. Why use JSONL?

JSONL provides a convenient structure where each line represents an independent training example.

---

## Q9. Why generate 300–400 conversations?

To provide enough examples covering different customer questions and desired response behaviors. The exact number required depends on the task and dataset quality.

---

## Q10. Why should synthetic data be reviewed?

An LLM generating the dataset can introduce incorrect, duplicated, inconsistent, or low-quality examples. Training on bad data can produce bad model behavior.

---

## Q11. Why split the dataset into 80/20?

The 80% portion is used for training and the 20% portion is used for validation in this exercise.

---

## Q12. What is the purpose of validation data?

To evaluate how the model performs on examples that were not used to directly update the trainable parameters during training.

---

## Q13. Why use Google Colab?

It provides a cloud notebook environment and may provide access to GPU resources useful for fine-tuning.

---

## Q14. What is Unsloth used for?

Unsloth provides tools and optimized workflows for efficient fine-tuning and inference of supported models.

---

## Q15. What is the difference between training and serving?

Training changes trainable model parameters to learn a task. Serving makes the trained model available for applications to send inference requests.

---

# 45. 🧠 Final Mental Model

The most important picture from today's class is:

```text
                     USE CASE
                        ↓
              Need New Behavior?
                        ↓
                   Fine-Tuning
                        ↓
                    Dataset
                        ↓
                   JSONL
                        ↓
             Generate / Collect Data
                        ↓
                  Review Data
                        ↓
                    80 / 20
                  ↙         ↘
             Training     Validation
                ↓             ↓
             80% Data      20% Data
                ↓
             Unsloth
                ↓
          LoRA / QLoRA
                ↓
         Google Colab GPU
                ↓
            Fine-Tuning
                ↓
         Loss / Evaluation
                ↓
         Fine-Tuned Model
                ↓
        ┌───────┴────────┐
        ↓                ↓
      Ollama            vLLM
        ↓                ↓
      Local           Production
```

---

# ⭐ One-Line Memory Trick

```text
PEFT = Train Less

LoRA = Train Small Matrices

QLoRA = Quantize + LoRA

Dataset = Teach with Examples

80/20 = Train / Validate

Unsloth = Efficient Fine-Tuning Workflow
```

---

# 🔥 The Most Important Lesson

The Pizza Palace project is not really about pizza.

It is teaching you the complete fine-tuning workflow:

```text
Real Business Requirement
        ↓
Identify Behavior
        ↓
Create Training Examples
        ↓
Prepare Dataset
        ↓
JSONL
        ↓
Split Dataset
        ↓
Choose Model
        ↓
Choose LoRA / QLoRA
        ↓
Configure Hyperparameters
        ↓
Fine-Tune
        ↓
Evaluate
        ↓
Deploy
```

Once you can do this with Pizza Palace, you can apply the same architecture to:

```text
Insurance Customer Support
Banking Assistant
HR Assistant
IT Helpdesk
E-commerce Support
Technical Support
```

The **domain changes**.

The **fine-tuning workflow remains largely the same**.

