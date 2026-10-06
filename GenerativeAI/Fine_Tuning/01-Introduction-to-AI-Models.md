# 🤖 GenAI Developer Notes

> **Course:** GenAI Developer  
> **Institute:** Quality Thought (QT)  
> **Class:** 01  
> **Date:** 28-Jul-2026  
> **Version:** 1.0  
> **Prepared By:** Venkat  
> **Language:** English + Telugu  


---

# 🎯 Learning Objectives

After completing this class, you should be able to answer:

✅ What is a Model?

✅ What is an Open Source Model?

✅ What is a Vendor Model?

✅ What is a Prompt?

✅ What is an SDK?

✅ What is an API Key?

✅ What is a Token?

✅ How does an LLM generate text?

---

# 1️⃣ What is a Model?

## English
* A Model is an AI program trained using a large amount of data to perform a specific task.

* The model learns patterns from the data.

* After training, it can answer questions, generate text, write code, understand images, and perform many intelligent tasks.

### Example

When you ask

```
What is the capital of France?
```

The model answers

```
Paris
```

The model is generating the answer based on what it learned during training.

---

## Telugu

**Model అంటే**

Model అనేది చాలా పెద్ద మొత్తంలో Data పై Train చేసిన AI Program.

Training తర్వాత అది

- ప్రశ్నలకు సమాధానం చెప్పగలదు
- Code రాయగలదు
- Images అర్థం చేసుకోగలదు
- Content Generate చేయగలదు

---

### Real Life Analogy

* Imagine a student, The student studies for four years
  Finally, He becomes an Engineer.

Similarly,

* The Model studies billions of **documents**. Finally, It becomes intelligent enough to answer questions.

---

# 📌 Important Point

Model ≠ Database

* A model does not simply store answers. It learns patterns from data.

---

# 2️⃣ Types of Models

The trainer explained two types.

```
Models

├── Open Source Models

└── Vendor Models
```

---

# 2.1 Open Source Models

## English

Open Source (or **Open Weight**) Models are models whose weights are publicly available for developers to use.

You can usually:

- Download them
- Run them
- Fine-tune them

Examples

- Llama
- Qwen
- GLM (General Language Model)
- Gemma


NOTE: 
* GLM stands for: General Language Model
* It is a family of Large Language Models developed by THUDM (Knowledge Engineering Group, Tsinghua University, China) and collaborators.

---

## Telugu

Open Source Models అంటే

Developer తన Computer లేదా Cloud లో Run చేసుకునే Models. వాటిని Download చేసుకోవచ్చు. Customize చేసుకోవచ్చు.

---

# 2.2 Vendor Models

Vendor Models are owned and **maintained by companies.**

Examples

```text 
OpenAI

Claude

Gemini

Amazon
```

The company hosts the model. We simply call the API.

---

## Telugu

Vendor Models అంటే, Model మొత్తం Company దగ్గర ఉంటుంది.

మనం API ద్వారా మాత్రమే ఉపయోగిస్తాము. Model మన దగ్గర ఉండదు.

---

# 📌 Difference

| Open Source | Vendor |
|-------------|---------|
| Download Possible | No |
| Run Locally | Usually Yes |
| Managed by You | No |
| API Available | Yes |

*(This is the basic comparison covered in class. We will study it in more depth later.)*

---

# 3️⃣ Model Modalities

## What is a Modality?

### English

A Modality means

**The type of data that a model can understand or generate.**

---

### Telugu

Modality అంటే

Model ఏ Data Type తో పని చేస్తుందో దాన్ని Modality అంటారు.

---

Trainer discussed five modalities.

```
Text

Code

Image

Audio

Video
```

---

## Text Model

Input

```
Text
```

Output

```
Text
```

Example

```
What is AI?

↓

Artificial Intelligence...
```

---

## Telugu

Text Model, Text ను తీసుకుని, Text నే Generate చేస్తుంది.

---

## Code Model

Input

```
Write Python Code
```

Output

Python Program

---

## Image Model

Input

```
Generate a Cat Image
```

Output

Image

---

## Audio Model

Input
```
Speech
```
Output
```
Text
```
or
```
Speech
```
---

## Video Model

Input
```
Text
```
Output
```
Video
```
---

# 4️⃣ Accessing Models

The trainer explained

There are two ways.

```
Developer

↓

Model

```

How?

---

## Open Source

```
Laptop

↓

Model

↓

Answer
```

or

```
AWS

↓

Model

↓

Answer
```

---

## Vendor

```
Application

↓

API

↓

Vendor

↓

Answer
```

---

# 5️⃣ Programmatically Accessing Models

To use AI inside our application, we need three things.

```
Prompt

+

API Key

+

SDK
```

---

# 5.1 Prompt

## English

A Prompt is the instruction given to the AI Model.

Example

```
What is the capital of France?
```

---

## Telugu

Prompt అంటే, AI కి ఇచ్చే Instruction లేదా Question.

---

# 5.2 API Key

## English

API Key is a **secret key** used to identify and authenticate your application. Never share it publicly.

---

## Telugu

API Key అంటే

మన Application ని గుర్తించే Secret Key. **దాన్ని GitHub లో Upload చేయకూడదు.**

---

# 5.3 SDK

**SDK stands for**
```
Software Development Kit
```

SDK is a library that makes it easier for developers to communicate with AI Models.

---

## Telugu

SDK అంటే Ready-made Library. దీనివల్ల API Call చేయడం చాలా సులభం అవుతుంది.

---

# Flow Diagram

```
Developer

↓

Python Program

↓

SDK

↓

API Key

↓

AI Model

↓

Response
```

---

# 6️⃣ Exercise 1

Trainer Task

```
Go to Google AI Studio

↓

Generate API Key

↓

Call Gemini Model

↓

Prompt

"What is capital of France?"
```

---

# 7️⃣ Exercise 2

Task

```
Convert Selfie

↓

Passport Size Photo
```

Question
```
Which Model?
```
Answer
```
Vision Model
```
---

# 8️⃣ Exercise 3

Task

Generate a Song for Spain winning FIFA.

Possible Technologies

- Speech to Text

- Text to Speech

- Voice Library

- Music Generation Model

---

# 9️⃣ Understanding LLM

Trainer said

LLM generates

```
One Token

↓

Another Token

↓

Another Token

↓

Another Token
```

Until

- End Token

or

- Maximum Token Limit.

---

## English

The model does NOT generate the entire answer at once. It predicts one token after another.

---

## Telugu

LLM మొత్తం Sentence ఒకేసారి Generate చేయదు. ఒక్కో Token ని Predict చేస్తూ ముందుకు వెళుతుంది.

---

# 🔟 Big Question

* If the Model predicts only one token, Why does it look intelligent?
* This question will be answered in the upcoming classes. For now,

Remember

```
Prediction

+

Huge Training Data

=

Useful Response
```

---

# 📝 Class Summary

Today we learned

✅ AI Models

✅ Types of Models

✅ Open Source Models

✅ Vendor Models

✅ Modalities

✅ Prompt

✅ SDK

✅ API Key

✅ Vision Models

✅ Music Generation

✅ LLM Basics

---

# 📚 Abbreviations

| Short Form | Full Form |
|------------|-----------|
| AI | Artificial Intelligence |
| LLM | Large Language Model |
| API | Application Programming Interface |
| SDK | Software Development Kit |
| GCP | Google Cloud Platform |
| AWS | Amazon Web Services |

---

# 📖 Vocabulary

| English | Telugu |
|----------|---------|
| Model | మోడల్ |
| Prompt | సూచన / ప్రశ్న |
| Token | చిన్న టెక్స్ట్ భాగం |
| API | అప్లికేషన్ కమ్యూనికేషన్ ఇంటర్‌ఫేస్ |
| SDK | సాఫ్ట్‌వేర్ డెవలప్‌మెంట్ కిట్ |
| Vendor | సేవ అందించే సంస్థ |
| Open Source | అందరికీ అందుబాటులో ఉన్న మోడల్ |
| Cloud | క్లౌడ్ సర్వీస్ |
| Vision | చిత్రం గుర్తింపు |
| Audio | శబ్దం |
| Video | వీడియో |
| Text | వచనం |
| Code | ప్రోగ్రామింగ్ కోడ్ |
| Response | సమాధానం |
| Training | శిక్షణ |
| Prediction | అంచనా |

---

# 🎯 What You Should Remember

> **A Model is an AI system trained on data to perform tasks.**

> **Developers interact with models using a Prompt, an API Key, and an SDK.**

> **LLMs generate responses one token at a time.**

---

# 🚀 Next Class Preview

In the next class, we will go deeper into:

- Tokens
- Tokenization
- Embeddings
- Vectors
- How LLMs understand language

```
Today's Foundation ➜ Tomorrow's Deep Dive
```
