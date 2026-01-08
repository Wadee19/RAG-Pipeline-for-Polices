# RAG Privacy Policy Simplifier

> **Important note**  
> this project is a proof of concept only. i am not providing legal advice and this output should not be treated as a replacement for the original privacy policy.


## 1. what this project does

i built this pipeline to take a very long and complex privacy policy and rewrite it in very simple english while keeping the full legal meaning.

the main goal is not to create a short summary, but to make the full policy readable by normal users who are not lawyers.

the pipeline supports two paths:

- a lightweight RAG-based path for quick risk highlights  
- a full chunk-by-chunk rewrite path using openai models for full coverage  


## 2. input data

i tested this poc using the tiktok privacy policy (other regions).  
the policy is large, legal-heavy, and realistic for testing hallucination and coverage issues.

**policy size before processing:**

- about **8237 words**
- about **50k characters**

this size is important because it is large enough to break one-shot generation and force chunking.


## 3. chunking strategy (why these numbers)

i split the policy into chunks before any generation.

**values used:**

- chunk size: **500 words**
- overlap: **80 words**
- minimum chunk size: **120 words**

**why i chose these numbers:**

- 500 words is small enough to stay stable inside model context limits  
- 80 words overlap prevents meaning loss at chunk borders  
- 120 words minimum avoids tiny tail chunks that confuse the model  

with these settings, the policy produced around **20 chunks** with consistent sizes.


## 4. retrieval (RAG) setup

for the RAG path, i used semantic retrieval before generation.

**embedding model:**

- `sentence-transformers/all-MiniLM-L6-v2`

**retrieval values:**

- `top_k = 12`

**why top_k is 12:**  
i tested smaller values and they missed important legal areas like user rights, data transfers, and enforcement.  
12 was the lowest number that consistently covered all major policy sections without adding too much noise.

the RAG path is intentionally conservative and only meant for quick inspection, not full rewriting.


## 5. generation paths

### 5.1 RAG small rewrite (baseline)

**generator:**

- `mistralai/Mistral-7B-Instruct-v0.2`
- loaded with 4-bit quantization to fit colab gpu limits

**generation settings:**

- `max_new_tokens: 1400`
- `do_sample: false`
- `repetition_penalty: 1.12`
- `no_repeat_ngram_size: 8`

**why these values:**

- sampling disabled to reduce hallucination  
- repetition penalty avoids looping and 1.12 worked best for this use case
- ngram blocking avoids repeated legal phrases  

this path produces a short, readable output but does not cover the full policy.


### 5.2 openai full chunk-by-chunk rewrite (main result)

this is the main extension added in the latest code.

**model selection logic:**  
i query available openai models and pick the first available from this list:

- `gpt-5.2-pro`
- `gpt-5.2`
- `gpt-5`
- `gpt-4.1`
- `gpt-4o`
- `gpt-4o-mini`

**model used in the actual run:**

- `gpt-5.2-pro`

**generation settings:**

- `max_output_tokens per chunk: 1400`

**why 1400:**  
this is enough to fully rewrite a 500-word legal chunk into simpler english without truncation.  
lower values caused silent cut-offs in testing.

**prompt design constraints:**

- forces a1-level english  
- forbids summaries  
- forbids bullets and numbering  
- forbids adding or guessing information  
- keeps original order as much as possible  
- explains legal terms only when needed  

each chunk is rewritten independently, then stitched back together in the original order.


## 6. evaluation method

i evaluated results using both qualitative and quantitative checks.

**quantitative:**

- fkgl (flesch-kincaid grade level)
- gunning fog
- smog index
- word and character counts

**qualitative:**

- head / middle / tail comparison  
- manual eye check for flow consistency  
- checking that no new obligations or claims were invented  


## 7. results summary

**original policy:**

- ~8237 words  
- fkgl ~14.8  
- gunning fog ~17.6  
- very hard for normal users to read  

**rag small output:**

- ~374 words  
- slightly better readability  
- acts as a short highlight, not a real simplification  

**openai full rewrite:**

- ~13318 words  
- fkgl ~8.1  
- gunning fog ~10.3  
- smog ~10.7  

the word count increased on purpose.  
legal sentences were split and complex terms were explained inline.  
this is expected behavior when simplifying without losing meaning.

![results_image](images/readability_scores.png)

## 8. runtime and cost notes

**runtime for one full policy:**

- about **24 minutes** end-to-end in colab

**reason:**

- one openai call per chunk  
- sequential processing  

**cost note:**  
this approach is more expensive than RAG-only methods.  
scaling to many policies will require batching, parallelization, or budget planning.


## 9. saved outputs

the pipeline saves:

- original policy text  
- rag small output  
- openai full rewritten policy  

all files are saved as plain txt for traceability.


## 10. limitations

- js-rendered websites may return partial text  
- not legal advice  
- no automatic legal validation  
- high cost if scaled blindly  


## 11. future improvements

- add coverage checks per policy section  
- add chunk-to-output trace ids  
- parallelize openai calls  
- test stronger embedding models  
- add optional human review stage  


## 12. final note

i consider this poc successful for its intended goal.  
the chunk-by-chunk rewrite is the only method that achieved full coverage and real readability improvement without turning the policy into marketing text.

## 13. License

The MIT License (MIT)

```xml
Copyright (c) 2026 Ahmed Wadee Moustafa

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in
all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN
THE SOFTWARE.