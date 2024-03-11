# Anote NER 

## Named Entity Recognition (NER) 
In 2024, the NER task mainly uses some form of text-generation techiques. 
- finetuning on dataset begin and offset  cnnllp distillbert





A major problem with using the LLM for text-generation is that these models were so huge that I have to design my experiments based on the computational constraints of my laptop, kaggle gpu and free google colab. Apart from using the LLM models huggingface API for developing an app, I could not test these LLM models with my laptop GPU. So, most of my testing is done using Kaggle notebooks and google colab pro version. 

## NER Task for Custom Entities 
### How to use LLM for NER task with custom entities 
Since we do not have lot of labled data with custom entities tags and lot of time for supervised learning, I started off with prompting the the Google's Gemini and OpenAI's GPT-3. Once I was happy with my prompt as seen below. I realized that instruction-tuning instead of fine-tuning is more suitable for my task at hand. I then investigated the current landscape of state-of-the-art models for NER which has significantly changed since 2022 when I last worked on this problem. In 2024, all the models use prompting and instruction-tuning with zeroshot or few shot for NER tasks. One of the models I investigated for NER task using text-generation was UniversalNER LLM variations released by Microsft on the Huhhingface hub. UniversalNER returns a tuple of the entities form the entire text document for the user specified entity-type. However, the limitations of this model includes that I can only check for one entity type (eg. location) at a time in a text document. For checking multiple entities type in a text docuemnt, I get a empty array of entities. UniversalNER has used NER datasets from various domains so can identify thte entities for internetactivity and medicalrecord. But, no amount of changes in the print statements and Langchain prompt template can make the model output entity-type beside the entities in the original text because of the way the model is designed. In conclusion, I decided to use text-generation techniques to solve the given NER task. Note that, this means when I deploy the NER pipeline on Huggingface, docker and Kubernetes I would define Task as text-generation and not NER.


### How to use evualuation metric for zero shot instruction tuning of LLM for NER

Additionally, I also investigated Meta's open-source LLama model for NER task which also uses text-generation technique by zero shot instruction tuning. My solution to the problem is based on [GoLLIE, a zero shot Information Extraction](https://github.com/hitz-zentroa/GoLLIE) as seen below by defining the annotation schema for custom entities type based on given PII taxanomy. The google colab notebook for the solution can be accessed here. This solution also allows me to set the gold standard and and use F1 evaluation score metric, including full control on the instruction template.  However, I am currently trying to resolve the dependency conflict with CUDA & GPU architecture. Once I resolve this problem I intend to create a Docker image for the same once and for all. 
I have mamanged to resolve the issues arising due to conflicting installation of flash-attention. Overall, resolving dependency conflicts took most amount of my time. 


```python



```



Apart from the dependency conflixt issues, another major problem with using the LLM for text-generation was they were huge 

