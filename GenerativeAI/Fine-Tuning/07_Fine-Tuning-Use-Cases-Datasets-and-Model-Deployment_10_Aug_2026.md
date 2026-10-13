# Chapter 07 - Fine-Tuning Use Cases, Datasets and Model Deployment

> **Course:** Gen-AI Developer  
> **Institute:** Quality Thought (QT)  
> **Class:** 07  
> **Date:** 10-Aug-2026

---

# 01. Why Would I Fine-Tune?

The first question before fine-tuning is:

> **Why do I need to fine-tune the model?**

The trainer gave an important rule:

```text
Fine-Tuning
    ↓
Behavior

RAG
    ↓
Facts / Knowledge
````

Fine-tuning is mainly useful when we want to change or specialize the **behavior** of a model.

RAG is mainly useful when the model needs access to **external, current, private, or frequently changing information**.

---

# 02. Fine-Tuning is Mainly for Behavior

Suppose we already have a general-purpose LLM.

The model already knows how to:

* Understand language
* Generate text
* Follow instructions
* Answer questions

But we want the model to behave in a specific way.

For example:

* Always respond like a customer-support representative
* Follow a specific response style
* Use a particular output format
* Follow company communication guidelines
* Perform a specialized task consistently

Fine-tuning can help teach these patterns.

## Simple Mental Model

```text
Existing Model

      +

Training Examples

      ↓

Specialized Behavior
```

### Telugu

Fine-Tuning అంటే

ఇప్పటికే ఉన్న Model కి

ఒక ప్రత్యేకమైన

**Behavior / ప్రవర్తన**

నేర్పించడం.

---

# 03. RAG vs Fine-Tuning

This is one of the most important concepts from this class.

```text
RAG
 ↓
"What information should I use?"

Fine-Tuning
 ↓
"How should I behave?"
```

This is a useful mental model.

---

# 04. Example 1 - Organization Policies

Suppose we are building an application that answers questions about an organization's policies.

The organization may have:

* Leave Policy
* Work From Home Policy
* Travel Policy
* Insurance Policy
* Employee Benefits
* Security Policy

These policies can change.

For example:

```text
2026 Leave Policy

        ↓

Policy Updated

        ↓

2027 Leave Policy
```

Would we want to retrain the model every time a policy changes?

Usually, no.

RAG is a better fit.

---

# 05. Why RAG is Suitable

A RAG system can retrieve the latest information at runtime.

```text
Company Documents
        ↓
Document Processing
        ↓
Chunking
        ↓
Embeddings
        ↓
Vector Database
        ↓
User Question
        ↓
Retrieve Relevant Information
        ↓
LLM
        ↓
Answer
```

For example:

```text
User:

How many casual leaves can I take?
```

The RAG system retrieves the relevant section of the latest leave policy.

The LLM then uses that retrieved information to generate the answer.

---

# 06. Telugu Explanation - RAG

Company Policies frequently change అవుతుంటే,

ప్రతి మార్పుకీ Model ని retrain చేయడం అవసరం లేదు.

Instead:

```text
Updated Documents
       ↓
RAG
       ↓
Relevant Information
       ↓
LLM
       ↓
Answer
```

అందుకే frequently changing information కోసం RAG చాలా useful.

---

# 07. Example 2 - Insurance Customer Support

Suppose we are building a customer-support chatbot for an insurance company.

We want the model to behave like a trained customer-support representative.

For example:

```text
Customer:

I want to know about my policy.


Assistant:

Certainly. I'd be happy to help you
with your policy. Could you please
provide your policy number?
```

We want the model to consistently:

* Be polite
* Follow company tone
* Ask appropriate questions
* Follow a specific response structure
* Behave like a customer-support representative

This is primarily a **behavior problem**.

Therefore, fine-tuning can be appropriate.

---

# 08. Fine-Tuning + RAG

In real applications, we don't always have to choose only one.

We can combine both.

```text
                 Insurance Chatbot
                        |
            ┌───────────┴───────────┐
            ↓                       ↓
       Fine-Tuning                  RAG
            ↓                       ↓
      Learn Behavior          Retrieve Facts
            ↓                       ↓
    Customer Support        Current Policies
            │                       │
            └───────────┬───────────┘
                        ↓
                       LLM
                        ↓
                     Response
```

Fine-tuning teaches:

> **How should I respond?**

RAG provides:

> **What information should I use right now?**

---

# 09. Important Rule

Remember:

```text
Fine-Tuning
=
Behavior / Style / Format / Task Pattern

RAG
=
External / Current / Frequently Changing Information
```

This is a very useful engineering rule.

---

# 10. Important Clarification

Do not treat this as an absolute rule:

```text
Fine-Tuning = Never for knowledge
RAG = Never for behavior
```

The real-world situation can be more nuanced.

Fine-tuning can affect what a model learns from training examples, and RAG can influence behavior through prompts and retrieved context.

A better engineering decision is:

> **Use RAG when information needs to be retrieved or updated dynamically. Use fine-tuning when you want the model to consistently learn a desired behavior, style, format, or task pattern.**

---

# 11. Datasets

Once we decide that fine-tuning is appropriate, the next question is:

> **What data do we use for fine-tuning?**

The answer is:

**Dataset.**

A dataset contains examples that teach the model the desired behavior.

For an instruction-following model:

```text
Instruction
      ↓
Expected Response
```

---

# 12. JSONL

The trainer mentioned that a popular dataset format is:

**JSONL**

## Full Form

**JSONL = JSON Lines**

JSONL is a text format where:

> **Each line contains one independent JSON object.**

Example:

```json
{"instruction":"Explain Kubernetes","response":"Kubernetes is a container orchestration platform."}
{"instruction":"Explain Docker","response":"Docker is a platform for building and running containers."}
{"instruction":"Explain Terraform","response":"Terraform is an infrastructure as code tool."}
```

Each line represents a separate training example.

---

# 13. Why JSONL?

Suppose we have:

```text
100,000 training examples
```

JSONL allows us to store:

```text
Line 1     → Example 1
Line 2     → Example 2
Line 3     → Example 3
...
Line 100000 → Example 100000
```

Each record can be processed independently.

This is convenient for large datasets and training pipelines.

---

# 14. Conversational Dataset

In many cases, we fine-tune **Instruct Models**.

Therefore, the dataset commonly contains conversational examples.

Example:

```json
{
  "messages": [
    {
      "role": "user",
      "content": "What is Kubernetes?"
    },
    {
      "role": "assistant",
      "content": "Kubernetes is a platform used to manage containerized applications."
    }
  ]
}
```

Another example:

```json
{
  "messages": [
    {
      "role": "user",
      "content": "Explain Docker simply."
    },
    {
      "role": "assistant",
      "content": "Docker allows applications to run in isolated containers."
    }
  ]
}
```

The model learns the relationship:

```text
User Instruction
       ↓
Desired Assistant Response
```

---

# 15. Dataset Structure Depends on the Model

There is no single universal dataset structure that works identically for every model.

Different models may expect different:

* Chat templates
* Special tokens
* Message structures
* Role formats
* Formatting conventions

For example, one model may use:

```text
user
assistant
system
```

while the underlying formatting may be transformed using a model-specific chat template.

Therefore:

> **Always check the model's expected dataset format and chat template before fine-tuning.**

---

# 16. Why Instruct Models are Commonly Fine-Tuned

The trainer mentioned:

> In most cases we fine-tune Instruct Models.

Why?

Because Instruct Models have already learned how to interact with users.

They understand the general pattern:

```text
User Instruction
       ↓
Assistant Response
```

Fine-tuning can then specialize this behavior.

For example:

```text
General Instruct Model

       ↓

Insurance Customer-Support Dataset

       ↓

Insurance Customer-Support Behavior
```

---

# 17. Open-Source Models

The trainer then asks:

> How do I run open-source models on my hardware?

This introduces the open-model ecosystem.

Important tools mentioned:

```text
Hugging Face
llama.cpp
Ollama
Docker
vLLM
```

Each has a different purpose.

---

# 18. Hugging Face

Hugging Face provides a large ecosystem for:

* AI models
* Datasets
* Tokenizers
* Transformers
* Training tools
* Model sharing

Think of it as a major ecosystem for discovering and working with open models.

Typical flow:

```text
Hugging Face
      ↓
Choose Model
      ↓
Download / Load Model
      ↓
Fine-Tune
      ↓
Run Model
```

---

# 19. llama.cpp

**llama.cpp** is a C/C++ based runtime for efficient local LLM inference.

The important concept is:

```text
LLM
 ↓
llama.cpp
 ↓
Local Hardware
 ↓
Inference
```

It is widely associated with efficient local inference and GGUF model workflows.

### Telugu

llama.cpp ఉపయోగించి

LLM ని

మన Local Machine లో

efficient గా run చేయవచ్చు.

---

# 20. Ollama

**Ollama** provides a simpler developer experience for running supported LLMs locally.

Conceptually:

```text
Model
  ↓
Ollama
  ↓
Local Model Runtime
  ↓
Application
```

It is useful for experimentation and local development.

---

# 21. Docker

Docker can package an application, model runtime, libraries, and dependencies into a container.

Conceptually:

```text
Application
     +
Model Runtime
     +
Dependencies
     ↓
Docker Container
```

This helps make environments reproducible and easier to deploy.

---

# 22. vLLM

The trainer mentioned:

> Production: vLLM

**vLLM** is a high-performance framework for serving LLMs.

Think of it as a model-serving layer:

```text
Client Applications
        ↓
      API
        ↓
      vLLM
        ↓
       LLM
        ↓
    Response
```

It is particularly useful for production inference workloads where throughput and efficient serving matter.

---

# 23. Local Model Journey

Now connect the tools.

```text
                 OPEN MODEL
                     ↓
                Hugging Face
                     ↓
             Download / Load
                     ↓
                Fine-Tuning
                     ↓
                LoRA / QLoRA
                     ↓
               Save Adapter
                     ↓
              Run the Model
                     ↓
        ┌────────────┼────────────┐
        ↓            ↓            ↓
    llama.cpp      Ollama       vLLM
        ↓            ↓            ↓
      Local       Easy Local   Production
```

This is the high-level ecosystem.

---

# 24. What If I Want to Fine-Tune Claude, Gemini or GPT?

The trainer asked an important question:

> What if I want to fine-tune Claude, Gemini, or GPT?

There is an important difference between open models and vendor-hosted models.

## Open Models

You can often:

```text
Download Weights
       ↓
Run Locally
       ↓
Fine-Tune
```

depending on the model's license and available tooling.

## Vendor Models

For models such as:

```text
Claude
Gemini
GPT
```

you generally do not download the underlying model weights and run LoRA on your own hardware.

Instead, you must use whatever customization or fine-tuning capability the model provider exposes.

Conceptually:

```text
Vendor Model
      ↓
Vendor API / Platform
      ↓
Supported Customization
```

The exact capabilities depend on the provider and model.

---

# 25. Open Model vs Vendor Model

| Feature             | Open Model      | Vendor Model       |
| ------------------- | --------------- | ------------------ |
| Download weights    | Often possible  | Generally not      |
| Run locally         | Often possible  | Generally not      |
| Own hardware        | Yes             | No                 |
| Local LoRA          | Often possible  | Generally not      |
| Provider API        | Sometimes       | Yes                |
| Fine-tuning options | Model-dependent | Provider-dependent |

Always check the model's license and provider documentation before using it.

---

# 26. Choosing Between RAG and Fine-Tuning

Use this decision process:

```text
Do I need new or changing facts?
              │
          ┌───┴───┐
         YES      NO
          │        │
          ▼        ▼
         RAG     Continue
```

Then:

```text
Do I need a specific behavior,
style, format or task pattern?
              │
             YES
              ↓
         Fine-Tuning
```

If you need both:

```text
Behavior
    +
Current Facts
    ↓
Fine-Tuning + RAG
```

---

# 27. Example - Insurance Chatbot

Let's combine everything.

Requirement:

> Build an insurance customer-support assistant.

### Behavior

The assistant should:

* Be polite
* Use company tone
* Ask appropriate questions
* Follow a specific response format

Possible solution:

```text
Fine-Tuning
```

### Facts

The assistant must know:

* Current policies
* Current claim procedures
* Current product details
* Updated insurance rules

Possible solution:

```text
RAG
```

### Complete Architecture

```text
                  User
                    ↓
             Insurance Bot
                    ↓
        ┌───────────┴───────────┐
        ↓                       ↓
 Fine-Tuned Model              RAG
        ↓                       ↓
   Behavior / Style       Current Information
        │                       │
        └───────────┬───────────┘
                    ↓
                  LLM
                    ↓
                Response
```

---

# 28. Next Topics

The trainer listed the next topics:

```text
PEFT
 ↓
LoRA
 ↓
QLoRA
```

These are the next important concepts in the fine-tuning journey.

---

# 29. Connection to Previous Classes

## Class 05

```text
Model
  ↓
Weights
  ↓
Fine-Tuning
  ↓
LoRA
```

## Class 06

```text
Fine-Tuning
  ↓
Dataset
  ↓
Hyperparameters
  ↓
Training
  ↓
Learning Loss
  ↓
Validation Loss
  ↓
Evaluation
```

## Class 07

```text
Why Fine-Tune?
       ↓
Behavior vs Facts
       ↓
RAG vs Fine-Tuning
       ↓
Datasets
       ↓
JSONL
       ↓
Instruct Models
       ↓
Open Models
       ↓
Local Runtimes
       ↓
Production Serving
       ↓
PEFT / LoRA / QLoRA
```

---

# 30. 🎯 Quick Revision

Remember these key points:

1. **Fine-tuning is mainly useful for specializing model behavior.**

2. **RAG is useful for providing external, current, private, or frequently changing information.**

3. **Fine-tuning and RAG can be combined.**

4. **JSONL means JSON Lines.**

5. **Each line in a JSONL file contains one JSON object.**

6. **Instruct models commonly use conversational examples for fine-tuning.**

7. **Dataset structure depends on the model and its chat template.**

8. **Hugging Face is a major ecosystem for models and datasets.**

9. **llama.cpp is a local LLM runtime.**

10. **Ollama provides an easier way to run supported LLMs locally.**

11. **Docker packages applications and dependencies into containers.**

12. **vLLM is designed for high-performance LLM serving.**

13. **Open models can often be downloaded and fine-tuned locally, depending on license and hardware.**

14. **Vendor models generally require provider-supported customization mechanisms.**

15. **PEFT, LoRA and QLoRA are important approaches for efficient fine-tuning.**

---

# 📚 Abbreviations

| Abbreviation | Full Form                                                                   |
| ------------ | --------------------------------------------------------------------------- |
| AI           | Artificial Intelligence                                                     |
| LLM          | Large Language Model                                                        |
| RAG          | Retrieval-Augmented Generation                                              |
| JSON         | JavaScript Object Notation                                                  |
| JSONL        | JSON Lines                                                                  |
| PEFT         | Parameter-Efficient Fine-Tuning                                             |
| LoRA         | Low-Rank Adaptation                                                         |
| QLoRA        | Quantized Low-Rank Adaptation                                               |
| API          | Application Programming Interface                                           |
| GPU          | Graphics Processing Unit                                                    |
| CPU          | Central Processing Unit                                                     |
| C++          | General-purpose programming language used by many high-performance runtimes |

---

# 📖 Vocabulary — English → Telugu

| English             | Telugu                                                    |
| ------------------- | --------------------------------------------------------- |
| Behavior            | ప్రవర్తన                                                  |
| Fact                | వాస్తవం / సమాచారం                                         |
| Fine-Tuning         | ప్రత్యేక ప్రవర్తన కోసం మోడల్‌ను శిక్షణ ఇవ్వడం             |
| Dataset             | డేటా సమాహారం                                              |
| Conversational      | సంభాషణకు సంబంధించిన                                       |
| Instruction         | సూచన                                                      |
| Response            | సమాధానం / ప్రతిస్పందన                                     |
| Runtime             | మోడల్‌ను అమలు చేసే వాతావరణం                               |
| Inference           | శిక్షణ పొందిన మోడల్‌తో Prediction చేయడం                   |
| Deployment          | మోడల్‌ను ఉపయోగించడానికి అందుబాటులో ఉంచడం                  |
| Production          | నిజమైన వినియోగ వాతావరణం                                   |
| Retrieve            | తిరిగి పొందడం                                             |
| Policy              | విధానం / నియమావళి                                         |
| Adapter             | మోడల్‌కు జోడించే చిన్న Trainable భాగం                     |
| Open Model          | అందుబాటులో ఉన్న / స్వయంగా అమలు చేయగల మోడల్                |
| Vendor Model        | ఒక సంస్థ నిర్వహించే మోడల్                                 |
| Serving             | మోడల్‌ను Applications కు అందించడం                         |
| Local               | స్థానిక కంప్యూటర్‌లో                                      |
| Current Information | ప్రస్తుత సమాచారం                                          |
| Chat Template       | సంభాషణను Model అర్థం చేసుకునే విధంగా Format చేసే Template |

---

# 🎓 Interview Questions

## Q1. When should you use RAG instead of fine-tuning?

When the application needs access to external, current, private, or frequently changing information.

---

## Q2. When should you consider fine-tuning?

When you want a model to consistently learn a particular behavior, style, format, or task pattern.

---

## Q3. Can RAG and fine-tuning be used together?

Yes.

Fine-tuning can specialize the model's behavior while RAG provides relevant and current information at runtime.

---

## Q4. What is JSONL?

JSONL stands for JSON Lines. Each line contains one independent JSON object.

---

## Q5. Why is JSONL commonly used for fine-tuning?

It allows individual training examples to be stored and processed conveniently, especially for large datasets.

---

## Q6. Why can dataset formats differ between models?

Different models may use different chat templates, special tokens, roles, and expected input structures.

---

## Q7. Why are Instruct Models commonly fine-tuned?

Because they are already trained to follow instructions and interact with users, making them a useful starting point for task-specific behavior.

---

## Q8. What is Hugging Face used for?

It provides an ecosystem for models, datasets, tokenizers, Transformers, and machine-learning tools.

---

## Q9. What is llama.cpp?

llama.cpp is a C/C++ based runtime used for efficient local LLM inference.

---

## Q10. What is Ollama?

Ollama is a developer-friendly tool for running supported LLMs locally.

---

## Q11. What is vLLM?

vLLM is a high-performance framework for serving LLMs, especially for production inference workloads.

---

## Q12. Can I download Claude or GPT and fine-tune it locally?

Generally, no. Vendor-hosted models do not normally provide their underlying weights for local fine-tuning. You need to use the customization or fine-tuning mechanisms provided by the vendor, if available.

---

# 🧠 Final Mental Model

The most important thing to remember from this class is:

```text
                WHAT IS MY PROBLEM?
                       │
          ┌────────────┴────────────┐
          ↓                         ↓
     Need Current              Need Behavior
       Facts?                    Change?
          │                         │
          ▼                         ▼
         RAG                   Fine-Tuning
                                    │
                                    ▼
                              PEFT / LoRA
                                    │
                                    ▼
                                  QLoRA
```

And for the model lifecycle:

```text
Open Model
    ↓
Hugging Face
    ↓
Load Model
    ↓
Dataset
    ↓
JSONL / Conversations
    ↓
Fine-Tuning
    ↓
LoRA / QLoRA
    ↓
Evaluate
    ↓
Deploy
    ↓
┌───────────────┬──────────────┬─────────────┐
↓               ↓              ↓
llama.cpp      Ollama          vLLM
Local          Easy Local      Production
```

> **Core lesson:** Don't fine-tune just because you have private data. First identify whether the problem is **knowledge retrieval** or **behavior specialization**. For changing knowledge, RAG is usually the better starting point; for consistent behavior, style, format, or task specialization, **fine-tuning may be appropriate**.


