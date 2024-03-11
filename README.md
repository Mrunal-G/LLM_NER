# Named Entity Recognition (NER) 

Named Entity Recognition is a challenging problem in NLP. Most LLMs do not perform that well at Named Entity Recognition task out of the box. For the cost of inference, a model trained specifically for NER gives better results with predictable outputs and without hallucination. This was the approach that I had used for dealing with custom NER task in 2022 using transformer models by manaually labelling the data and then using supervised fine-tuning method. By investigating the current landscape of NER in 2024, I noticed that most of the recent approaches for creating custom NER systems are inclined towards leveraging LLM models for text-generation using prompting and instruction-tuning with zeroshot or few shot. I have investigated this recent approach for the given NER task because of custom named entity-types taxanomy and lack of availabliity of labeled data for custom NER.

## LLM text-generation for NER - How does this work ?

1. **Create instruction data** for NER using LLM like Google's gemini or gpt-3.5 for generating entity mentions and their corresponding entity-types with sample text for distilling LLM capabilities specifically for NER task. 

2. **Instruction-tuning** The data generated by LLM in above method could be used for `traditional supervised fine-tuning` like this [exmaple](). However, 2024 [UniversalNER research paper](https://universal-ner.github.io/) by Microsoft found `conversational style fine-tuning` performed beter than `NER-style tuning`. 

Note: The Universal NER Benchamark is the largest NER benchamark to date.

3. In the **zero-shot setting** the UniNER-13B model outperformed the ChatGPT and VVicunna-7B on NER task. 

Taking this into consideration, I decided to focus on zero shot instruction-tuning method.

This model performance could be improved furhter for domain-specififc fine-tuning with human labelled data.


# Starting off
Since we do not have lot of labled data with custom entities tags and lot of time for supervised learning, I started off with prompting the the Google's Gemini and OpenAI's GPT-3. Once I was happy with my prompt as seen below, I could essentially use the APIs for these models to inference on the provided demo csv. However, to save the cost of compute and inferecing, I have investigated open-source models for custom NER and their api creation using huggingface. 

One of the models I investigated for NER task using text-generation was UniversalNER LLM variations released by Microsft on the Huhhingface hub. UniversalNER returns a tuple of the entities from the entire text document for the user specified entity-type. However, the limitations of this model includes that I can only check for one entity type (eg. location) at a time in a text document. By using brute force for checking multiple entities type in a text docuemnt, I get a empty array of entities. UniversalNER has used NER datasets from various domains so it can identify the entities for entity-types like InternetActivity and MedicalRecord. But, no amount of changes in the Langchain prompt template can make the model output entity-type beside the entities in the original text because of the nature and design of the model. In conclusion, I decided to use text-generation techniques to solve the given NER task. Note that, this means when I deploy the NER pipeline with Huggingface, docker and Kubernetes I would define Task as text-generation and not NER. 


### How to use F1 score evualuation metric for zero shot instruction tuning of LLM for NER

- Additionally, I also investigated a fine-tuned versoin of Meta's open-source Code LLama model for NER task which also uses text-generation technique by zero shot instruction tuning. My solution to the problem is based on [GoLLIE, a zero shot Information Extraction](https://github.com/hitz-zentroa/GoLLIE) as seen below by defining the annotation schema for custom entities type based on given PII taxanomy. The google colab notebook for the solution can be accessed [here](). 
- This solution also allows me to set the gold standard and and use F1 evaluation score metric, including full control on the instruction template as seen in this [example](https://github.com/hitz-zentroa/GoLLIE/blob/main/notebooks/Named%20Entity%20Recognition.ipynb). 

```python



```

## Challenges I faced

- However, I am currently trying to resolve the dependency conflict with CUDA & GPU architecture. Once I resolve this problem I intend to create a Docker image for the same and solve this issue once and for all. 
I have managed to resolve the issues arising due to conflicting installation of flash-attention. Overall, resolving dependency conflicts took most amount of my time.
- Apart from the dependency conflixt issues, a major problem with using the LLM for text-generation is that these models were so huge that I have to design my experiments based on the computational constraints of my laptop, kaggle gpu and free google colab. Apart from using the LLM models huggingface API for developing an app, I could not test these LLM models with my laptop GPU. So, most of my experiments are done using Kaggle notebooks and google colab pro version.








