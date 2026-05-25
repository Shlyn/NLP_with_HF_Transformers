<h1 align="center"> Natural Language Processing  with Hugging Face Transformers </h1>
<p align="center"> Tugas 14</p>

<div align="center">

<img src="https://img.shields.io/badge/python-3670A0?style=for-the-badge&logo=python&logoColor=ffdd54">
<img src="https://img.shields.io/badge/PyTorch-%23EE4C2C.svg?style=for-the-badge&logo=PyTorch&logoColor=white">

</div>

## Name : Shelina

## My Exercise : 

### 1. Exercise 1 - Sentiment Analysis

```
# TODO :
specific_model = pipeline(model="cardiffnlp/twitter-roberta-base-sentiment")
data = "AI tools are exciting, but many workers are still worried about how fast jobs will change."
specific_model(data)

original_model = pipeline("sentiment-analysis")
original_model(data)
```

Result : 

```
Specific model  : [{'label': 'LABEL_2', 'score': 0.6355943083763123}]
Default model   : compared in notebook to show different sentiment interpretation
```

Analysis on exercise 1 : 

This exercise compares a tweet-specific sentiment model with the default sentiment classifier. The notebook shows that the tweet-specific model is more suitable for social-media style text, because it can interpret mixed or nuanced opinion more carefully than the default model .

### 2. Exercise 2 - Topic Classification

```
# TODO :
classifier = pipeline("zero-shot-classification", model="facebook/bart-large-mnli")
classifier(
    "I love travelling and learning about new cultures and food.",
    candidate_labels=["art", "education", "travel"]
)
```

Result : 

```
{'sequence': 'I love travelling and learning about new cultures and food.',
 'labels': ['travel', 'education', 'art'],
 'scores': [0.9853582978248596, 0.010175268165767193, 0.004466455429792404]}
```

Analysis on exercise 2 : 

The classifier correctly ranks **travel** as the best label for the sentence, with a very large margin over the other options. This shows that zero-shot classification works well when the candidate labels are meaningful and closely related to the sentence context .

### 3. Exercise 3 - Text Generation

```
# TODO :
generator = pipeline("text-generation", model="gpt2")
generator("Hello, I'm a language model", max_length=30, num_return_sequences=3)
```

Result : 

```
[{'generated_text': "Hello, I'm a language model ..."},
 {'generated_text': "Hello, I'm a language model ..."},
 {'generated_text': "Hello, I'm a language model ..."}]
```

Analysis on exercise 3 : 

The model generates several continuations from the same prompt, showing that text generation is flexible and non-deterministic. The output is generally fluent, but each continuation can vary in meaning and quality because the model predicts likely next words rather than a single fixed answer .

### 4. Exercise 4 - Name Entity Recognition (NER)

```
# TODO :
nlp = pipeline("ner", model="dslim/bert-base-NER", aggregation_strategy="simple")
example = "Shelina lives in Batam and works with the Berlin Governance."
ner_results = nlp(example)
print(ner_results)
```

Result : 

```
entity_group: PER, word: She
entity_group: PER, word: lina
entity_group: LOC, word: Batam
entity_group: ORG, word: Berlin Governance
```

Analysis on exercise 4 : 

The NER model detects the important entities in the sentence, especially the location and organization. The person name is split into two parts, which shows that entity recognition can still work well overall even when tokenization causes slightly imperfect grouping .

### 5. Exercise 5 - Question Answering

```
# TODO :
question_answerer = pipeline("question-answering", model="distilbert-base-cased-distilled-squad")
question_answerer(
    question="Which planet is known as the Red Planet?",
    context="Mars is often called the Red Planet because of the iron oxide on its surface."
)
```

Result : 

```
{'score': 0.9993314146995544, 'start': 0, 'end': 4, 'answer': 'Mars'}
```

Analysis on exercise 5 : 

The question-answering model extracts the correct answer with extremely high confidence. This demonstrates that extractive QA works very well when the answer appears clearly and directly inside the given context .

### 6. Exercise 6 - Text Summarization

```
# TODO :
summarizer = pipeline("summarization", model="sshleifer/distilbart-cnn-12-6")
summarizer(
    "Renewable energy is becoming more important as countries try to reduce pollution and slow climate change. Solar panels and wind turbines are now used in many parts of the world to produce electricity without burning fossil fuels. Although the initial cost of building renewable energy systems can be high, the long-term benefits are significant. These technologies reduce greenhouse gas emissions, improve air quality, and create new jobs in engineering, maintenance, and manufacturing. Governments are also supporting clean energy through incentives and public investment. As battery storage improves, renewable energy will become even more reliable and practical for daily use.",
    max_length=59
)
```

Result : 

```
[{'summary_text': 'Renewable energy is becoming more important as countries try to reduce pollution and slow climate change. Solar panels and wind turbines are now used in many parts of the world to produce electricity without burning fossil fuels. These technologies reduce greenhouse gas emissions, improve air quality, and create new jobs in ...'}]
```

Analysis on exercise 6 : 

The summarization model keeps the core message of the paragraph while reducing supporting detail. It preserves the main ideas about clean energy, environmental benefits, and practical future use, which makes the summary informative and concise .

### 7. Exercise 7 - Translation

```
# TODO :
translator = pipeline("translation_en_to_de", model="t5-small")
translator("New York is my favourite city", max_length=40)
```

Result : 

```
translation_text: New York ist meine Lieblingsstadt
```

Analysis on exercise 7 : 

The translation model produces a correct German output for the English sentence. The result is short, natural, and grammatically appropriate, showing that the exercise successfully demonstrates basic machine translation with a Hugging Face pipeline .

---

## Analysis on this project

This project gives hands-on practice with seven different NLP exercises using Hugging Face Transformers. The exercises show how transformer pipelines can handle classification, generation, extraction, summarization, and translation tasks in a simple and practical way .
