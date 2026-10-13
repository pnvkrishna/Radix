# Chapter 03 - Inside a Large Language Model (LLM)

> **Course:** GenAI Developer
>
> **Institute:** Quality Thought (QT)
>
> **Class:** 03
>
> **Date:** 30-Jul-2026

---

# 🧭 Learning Journey

```
Chapter 01

Introduction to AI Models

        │

        ▼

Chapter 02

Evolution of LLMs

        │

        ▼

Chapter 03

Inside a Large Language Model

        │

        ▼

Chapter 04

Tokenization
```

Every chapter builds upon the previous one.

Do not skip chapters.

---

# 🌍 Big Picture

Before learning each component,

let us first understand how an LLM generates a response.

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

This entire chapter explains each block one by one.

---

# 1️⃣ Embeddings

In the previous chapter,

we learned that **Embeddings evolved from Word2Vec.**

Now, let us understand

how Embeddings are actually used inside an LLM.

---

## Why Do We Need Embeddings?

Computers cannot understand English, Telugu, Hindi or any human language. Computers understand only **Numbers.**

Therefore, before an LLM can process any sentence, the words must first be converted into numbers. This conversion is done using **Embeddings.**

---

## Definition

An Embedding is a numerical representation of a token.

Instead of storing

```
Apple
```

the model stores something like

```
[0.82, -0.14, 0.37, 0.55, ...]
```

This list of floating-point numbers is called an

**Embedding Vector.**

---

### తెలుగులో

Computer కి

English,

Telugu,

Hindi

ఏ Language కూడా నేరుగా అర్థం కాదు.

Computer కి Numbers మాత్రమే అర్థమవుతాయి.

అందుకే

ప్రతి Word ని

Numbers గా మార్చాలి.

ఈ పని చేసే Technique

**Embedding.**

---

## Trainer Note

Trainer said

```
Embedding

↓

Converts Words

↓

Vectors

↓

Floating Point Numbers
```

This is the most important statement of today's class.

---

# Real World Analogy

Suppose

every student in a college has

```
Hall Ticket Number
```

Instead of calling

```
Rahul
```

the college system stores

```
22102045
```

Similarly,

LLMs internally work with numbers.

However,

Embeddings are much richer.

Instead of

one number,

they use

hundreds or thousands of floating-point numbers.

---

# Example

Sentence

```
I love AI
```

Embeddings

```
I

↓

[0.42, -0.17, ...]

Love

↓

[-0.61, 0.72, ...]

AI

↓

[0.94, -0.12, ...]
```

The model never understands

```
Love
```

It understands

```
[-0.61, 0.72, ...]
```

---

# Think Like an Engineer

Question

Why don't we simply assign

```
Apple = 10

Orange = 20

Dog = 30
```

instead of Embeddings?

---

## Answer

Because

```
10

20

30
```

carry no meaning.

Embeddings preserve

relationships,

similarity,

and semantics.

---

# Memory Trick

```
Word

↓

Embedding

↓

Numbers

↓

Computer Understanding
```

Remember

> **Embeddings convert language into mathematics.**

---

# 2️⃣ Vocabulary

Before understanding Tokens,

we must first understand

Vocabulary.

---

## What is Vocabulary?

A Vocabulary is the complete collection of all tokens known to a model.

Think of it as

the model's dictionary.

If a token exists in the vocabulary,

the model already knows it.

---

### Telugu

Vocabulary అంటే

Model కి తెలిసిన

మొత్తం Tokens Collection.

దీనిని

Model Dictionary

అని కూడా అనుకోవచ్చు.

---

## Example

Imagine

Vocabulary contains

```
Apple

Orange

Cat

Dog

Run

Beautiful
```

Every token has

its own embedding.

---

## Important Note

Trainer mentioned

```
Embeddings contain

whole vocabulary.
```

That means

every token in the vocabulary

has its own embedding vector.

---

# Memory Trick

```
Vocabulary

↓

All Tokens

↓

Every Token

↓

One Embedding
```

---

# 3️⃣ Tokens

Now comes

one of the most important concepts.

---

## What is a Token?

A Token is the smallest unit of text processed by an LLM.

It is **not always a word.**

A token can be

- Word

- Part of a Word

- Number

- Symbol

- Punctuation

---

### Telugu

Token అంటే

LLM Process చేసే

చిన్న Text Unit.

ఇది

ఎప్పుడూ పూర్తి Word కావాల్సిన అవసరం లేదు.

కొన్నిసార్లు

ఒక Word లోని భాగం కూడా Token అవుతుంది.

---

## Example

Sentence

```
Artificial Intelligence
```

May become

```
Artificial

Intelligence
```

or

```
Art

ificial

Intelligence
```

depending on the tokenizer.

---

## Trainer Note

Trainer said

```
Embeddings have

Vocabulary

↓

Tokens
```

Remember

LLMs do not directly process words.

They process

**Tokens.**

---

## Input Tokens

When a user sends

```
Hello
```

↓

Input Token

---

## Output Tokens

When the model replies

```
Hi

How

are

you
```

↓

Output Tokens

---

## Important Point

Trainer said

```
LLMs

Accept

Input Tokens

Generate

Output Tokens
```

This sentence is extremely important.

---

# Pricing

Hosted LLMs

charge based on

```
Input Tokens

+

Output Tokens
```

The more tokens,

the higher the cost.

That is why developers care about

Token Count.

---

# Real World Example

Imagine

Bus Ticket.

More Distance

↓

More Fare.

Similarly,

More Tokens

↓

More API Cost.

---

# Memory Trick

```
More Tokens

↓

More Cost
```

Remember

> API Pricing is usually based on Token Usage.

---

# 🤔 Stop & Think

Question

If a paragraph contains

100 words,

does it always become

100 Tokens?

Think before reading further.

---

## Answer

No.

Depending on the tokenizer,

100 words

may become

80,

120,

or

150 tokens.

We will study this in detail

in the next chapter.

---

# 4️⃣ Transformer Processing

In the previous section,

we learned that

```
Sentence

↓

Tokens

↓

Embeddings
```

Now the question is,

**What happens after Embeddings?**

The answer is

**Transformer.**

The Transformer is the "brain" of a Large Language Model.

It takes embedding vectors as input,

understands the relationship between them,

and predicts the next token.

---

## LLM Processing Pipeline

```
User Prompt

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

↓

Repeat

↓

<EOS>

↓

Final Response
```

Every response generated by ChatGPT, Gemini, Claude, Llama, and Qwen follows this basic pipeline.

---

## What Does the Transformer Do?

The Transformer receives

```
Embedding Vectors
```

instead of words.

It studies

- the meaning of each token
- the relationship between tokens
- the context of the sentence

After understanding everything,

it predicts

**the most probable next token.**

---

### తెలుగులో

Transformer

Words ని చూడదు.

Vectors ని మాత్రమే చూస్తుంది.

అవి ఒకదానితో ఒకటి ఎలా సంబంధం కలిగి ఉన్నాయో అర్థం చేసుకుని

తర్వాత వచ్చే Token ఏదో Predict చేస్తుంది.

---

## Example

Prompt

```
The capital of France is
```

Tokenizer

↓

```
The

capital

of

France

is
```

Embeddings

↓

```
[ ...]

[ ...]

[ ...]

[ ...]

[ ...]
```

Transformer processes all these vectors.

Finally,

it predicts

```
Paris
```

---

## Trainer Note

Remember

Transformer never predicts

an entire sentence.

It predicts

**one token at a time.**

---

# 5️⃣ Position Information

Your trainer mentioned

```
Tokens are captured with positions.
```

This is an important concept.

Imagine this sentence.

```
Dog bites man.
```

Now change the order.

```
Man bites dog.
```

Both sentences contain

exactly the same words.

But the meaning changes completely.

Why?

Because

**word position matters.**

---

## Why Position is Important

Suppose the model only receives

```
Dog

Man

Bites
```

without order.

It cannot determine

who bit whom.

Therefore,

the Transformer also receives

the position of every token.

---

## Example

```
Token         Position

The              1

capital          2

of               3

France           4

is               5
```

Now the model knows

both

- the token
- the token's position

---

### తెలుగులో

Sentence లో

Words ఏ క్రమంలో ఉన్నాయో

చాలా ముఖ్యమైన విషయం.

అందుకే

Transformer కి

Word మాత్రమే కాదు,

దాని Position కూడా ఇవ్వబడుతుంది.

---

## Memory Trick

```
Token

+

Position

↓

Correct Meaning
```

Remember

Without position,

language loses meaning.

---

# 6️⃣ Probability Distribution

This is one of the most important concepts in AI.

Your trainer wrote

```
Transformer Output

↓

Probability Distribution
```

---

## What is Probability Distribution?

The Transformer

does not immediately choose a word.

Instead,

it calculates

the probability of

every possible next token.

---

## Example

Prompt

```
The capital of France is
```

The model internally calculates something like

| Token | Probability |
|--------|------------:|
| Paris | 96% |
| London | 2% |
| Berlin | 1% |
| Rome | 1% |

Notice

The model does not say

```
Paris
```

immediately.

First,

it calculates probabilities.

---

## Another Example

Prompt

```
The sun rises in the
```

Possible probabilities

| Token | Probability |
|--------|------------:|
| East | 98% |
| West | 1% |
| North | 0.5% |
| South | 0.5% |

Again,

the model chooses

the token with the highest probability

(or another token depending on Temperature, which we'll see next).

---

### తెలుగులో

Transformer

ముందుగా

అన్ని Tokens కి

Probability లెక్కిస్తుంది.

తర్వాత

ఏ Token వచ్చే అవకాశం ఎక్కువగా ఉందో

దాన్ని ఎంచుకుంటుంది.

---

## Think Like an Engineer

Question

Does an LLM know the answer?

---

### Answer

Not exactly.

An LLM predicts

the **most probable next token**

based on the context.

That is why LLMs are called

**Next Token Prediction Models.**

---

## Memory Trick

```
Sentence

↓

Transformer

↓

Probability

↓

Token
```

Remember

The Transformer predicts

probabilities,

not answers.

---

# 7️⃣ Temperature

Your trainer mentioned

```
Temperature

0 — 2
```

Temperature controls

**how the model chooses the next token from the probability distribution.**

---

## English Definition

Temperature is a parameter that controls the randomness or creativity of the model's output.

Lower temperature

↓

More deterministic.

Higher temperature

↓

More creative.

---

### తెలుగులో

Temperature అంటే

Model ఎంత Creativity తో

Response ఇవ్వాలో నిర్ణయించే Setting.

---

## Temperature = 0

Prompt

```
2 + 2 =
```

Response

```
4
```

Almost every time,

the answer will be

```
4
```

Very consistent.

---

## Temperature = 1

The model becomes

more flexible.

It may generate

slightly different wording

while still remaining sensible.

---

## Temperature = 2

The model becomes

highly creative.

Sometimes,

the responses may also become

less predictable.

---

## Simple Visualization

```
Temperature

0

↓

Safe

↓

Accurate

↓

Less Creative



Temperature

2

↓

Creative

↓

Random

↓

Less Predictable
```

---

## When Should We Use Different Temperatures?

| Task | Temperature |
|------|------------:|
| Mathematics | Low |
| Programming | Low |
| Translation | Low |
| Story Writing | High |
| Poetry | High |
| Brainstorming | High |

---

## Trainer Note

Temperature

does not change

the knowledge of the model.

It only changes

how the model selects the next token.

---

## Common Misconception

❌ Higher Temperature means a smarter model.

✅ Higher Temperature means a more creative model.

---

## Memory Trick

```
Low Temperature

↓

Accuracy

High Temperature

↓

Creativity
```

Remember

Temperature controls

**creativity,

not intelligence.**

---

# 8️⃣ End of Sequence (EOS) Token

Until now, we learned that an LLM generates one token at a time.

A natural question is:

> **How does the model know when to stop?**

The answer is:

**EOS Token**

---

## What is an EOS Token?

**EOS** stands for

**End Of Sequence**.

It is a special token that tells the model:

> "The response is complete. Stop generating more tokens."

---

### తెలుగు

EOS (End Of Sequence) అంటే

**"ఇక్కడితో సమాధానం పూర్తయింది"**

అని Model కి తెలియజేసే ప్రత్యేక Token.

ఈ Token వచ్చిన వెంటనే

Model Generation ఆపేస్తుంది.

---

## Example

Suppose the model generates

```
India

is

a

beautiful

country

<EOS>
```

After `<EOS>`

↓

Generation Stops.

---

## Why Do We Need EOS?

Imagine there is no EOS token.

The model might generate

```
Hello

Hello

Hello

Hello

Hello...
```

and never stop.

EOS tells the model

when the response is finished.

---

## Trainer Note

Trainer said

```
LLMs continue generation

↓

Until

<EOS>

or

Maximum Token Limit
```

Remember this sentence.

It is frequently asked in interviews.

---

## Memory Trick

```
<EOS>

↓

Stop
```

---

# 9️⃣ Maximum Token Limit

Sometimes,

the model may not generate an EOS token quickly.

To prevent infinite generation,

every LLM has

a **Maximum Token Limit**.

---

## English Definition

The Maximum Token Limit defines

the maximum number of tokens

the model is allowed to process or generate.

After reaching this limit,

generation automatically stops.

---

### తెలుగు

Maximum Token Limit అంటే

Model

గరిష్టంగా ఎంత Tokens వరకు

Process చేయాలో

లేదా Generate చేయాలో

నిర్ణయించే Limit.

ఈ Limit పూర్తయితే

Response ఆగిపోతుంది.

---

## Example

Suppose

Maximum Output Tokens

=

100

Even if the model has more to say,

it stops after generating

100 tokens.

---

## Why Is This Important?

Limits help in

- Faster responses
- Lower API cost
- Better resource utilization

---

## Memory Trick

```
<EOS>

OR

Max Tokens

↓

Generation Stops
```

---

# 🔟 Training Datasets

A Large Language Model becomes intelligent

because it is trained on

massive datasets.

---

## What is a Dataset?

A Dataset is

a collection of information

used for training AI models.

Think of it as

the model's study material.

---

### తెలుగు

Dataset అంటే

Model కి నేర్పించడానికి ఉపయోగించే

సమాచారం Collection.

దీనిని

Model యొక్క Study Material

అని కూడా అనుకోవచ్చు.

---

## Trainer Mentioned Popular Datasets

### Common Crawl

A massive collection of publicly available web pages.

Used to expose models to

real-world internet content.

---

### Wikipedia

Provides

well-structured,

fact-based knowledge.

---

### Books

Books help models learn

language,

writing style,

storytelling,

and grammar.

---

### GitHub Open Source Code

Helps coding models learn

- Python
- Java
- JavaScript
- Go
- Terraform
- YAML
- Docker
- Kubernetes

and many other programming languages.

---

### arXiv

A research paper repository.

Helps models learn

scientific knowledge,

mathematics,

AI,

physics,

computer science,

and engineering concepts.

---

## Trainer Note

Different datasets teach different skills.

For example,

GitHub improves coding ability,

while Wikipedia improves factual knowledge.

---

## Memory Trick

```
More Quality Data

↓

Better Learning
```

Remember

A model is only as good as

its training data.

---

# 1️⃣1️⃣ Raw Training vs Distillation

Trainer mentioned

```
Models can be trained

↓

Raw Datasets

OR

Distillation
```

---

## Raw Training

The model learns directly

from large datasets

such as

- Common Crawl
- Wikipedia
- Books
- GitHub

This requires

enormous

time,

hardware,

and cost.

---

### తెలుగు

Raw Training అంటే

Model

నేరుగా Original Data నుండి

నేర్చుకోవడం.

---

## Distillation

Instead of learning directly

from huge datasets,

a smaller model learns

from a larger,

more capable model.

Think of it like

a teacher teaching a student.

---

### Real World Analogy

Teacher

↓

Student

The student doesn't read

every book in the library.

Instead,

the teacher explains

the important concepts.

That is similar to

Model Distillation.

---

### తెలుగు

Distillation అంటే

పెద్ద Model

చిన్న Model కి

నేర్పించడం.

Teacher

↓

Student

అనే విధంగా.

---

## Memory Trick

```
Raw Training

↓

Original Data

Distillation

↓

Teacher Model
```

---

# 1️⃣2️⃣ Human Alignment

Trainer mentioned

```
Models also go through training

on

how to respond to humans.
```

This is an extremely important concept.

---

## Why?

Imagine a model that knows

millions of facts

but answers

rudely,

incorrectly,

or unsafely.

That would not be useful.

Therefore,

after learning knowledge,

models are further trained

to interact with humans.

---

### తెలుగు

Model కి Knowledge ఉండటం మాత్రమే సరిపోదు.

మనుషులతో

మర్యాదగా,

సురక్షితంగా,

ఉపయోగకరంగా

మాట్లాడడం కూడా నేర్పిస్తారు.

---

## Goal

Teach the model

- Helpful responses
- Safe responses
- Polite responses
- Better conversation

---

## Trainer Note

Knowledge Training

≠

Human Interaction Training

Both are different stages.

---

# 1️⃣3️⃣ Base Model vs Instruct Model

This was one of the trainer's exercises.

---

## Base Model

A Base Model

predicts

the next token.

It has knowledge,

but it is **not specifically trained to follow user instructions**.

---

## Instruct Model

An Instruct Model

is further trained

to understand

and follow

human instructions.

---

### Example

Prompt

```
Write a professional resignation email.
```

Base Model

↓

May simply continue text.

Instruct Model

↓

Writes a proper resignation email

because it has been trained

to follow instructions.

---

### తెలుగు

Base Model

↓

Knowledge ఉంటుంది.

Instructions Follow చేయడం

ప్రత్యేకంగా నేర్పించరు.

Instruct Model

↓

Instructions ఎలా Follow చేయాలో

ప్రత్యేకంగా Training ఇస్తారు.

---

## Comparison

| Base Model | Instruct Model |
|------------|----------------|
| General knowledge | Knowledge + Instruction Following |
| Predicts text | Understands user requests |
| Mainly for research | Best for chat applications |

---

## Memory Trick

```
Base

↓

Knowledge

Instruct

↓

Knowledge

+

Human Instructions
```

---

# 📝 Chapter Summary

Today we explored

the complete internal pipeline

of a Large Language Model.

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

We also learned

- Training Datasets
- Distillation
- Human Alignment
- Base Models
- Instruct Models

Together,

these concepts explain

how modern LLMs are built,

trained,

and used.

---

# 📚 Abbreviations

| Abbreviation | Full Form |
|--------------|-----------|
| LLM | Large Language Model |
| EOS | End Of Sequence |
| GPU | Graphics Processing Unit |
| TPU | Tensor Processing Unit |
| NLP | Natural Language Processing |
| API | Application Programming Interface |

---

# 📖 Vocabulary (English → Telugu)

| English | Telugu |
|----------|---------|
| Embedding | అర్థవంతమైన సంఖ్యా ప్రతినిధిత్వం |
| Token | చిన్న టెక్స్ట్ యూనిట్ |
| Vocabulary | పదాల/టోకెన్ల సమాహారం |
| Transformer | ట్రాన్స్‌ఫార్మర్ ఆర్కిటెక్చర్ |
| Probability | సంభావ్యత |
| Temperature | సృజనాత్మకత నియంత్రణ పరామితి |
| EOS | ముగింపు టోకెన్ |
| Dataset | శిక్షణ డేటా సమాహారం |
| Distillation | పెద్ద మోడల్ నుండి చిన్న మోడల్‌కు జ్ఞానం బదిలీ |
| Base Model | ప్రాథమిక మోడల్ |
| Instruct Model | సూచనలను అనుసరించే మోడల్ |
| Alignment | మానవ అవసరాలకు అనుగుణంగా శిక్షణ |

---

# 🎯 Quick Revision

Remember these key ideas:

✅ LLMs work with **tokens**, not words.

✅ Embeddings convert tokens into numerical vectors.

✅ The Transformer predicts a **probability distribution** for the next token.

✅ Temperature controls **creativity**, not intelligence.

✅ Generation stops when the model outputs **`<EOS>`** or reaches the **maximum token limit**.

✅ Models learn from massive datasets such as Common Crawl, Wikipedia, Books, GitHub, and arXiv.

✅ Distillation transfers knowledge from a larger model to a smaller one.

✅ Base models know language, while Instruct models are additionally trained to follow human instructions.