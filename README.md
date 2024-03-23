Realizing that finetuning and using NER approach is not suitable for PII detection at all, since the lot of the entity types need context awareness to understand the PII type. Prompting various open source models and using Langchain or something similar is going to be the way to solve this problem.

I inferenced Microsoft Presidio on demo data PII text csv and the results are still not satisfactory.


# Current Approach

## API development
Currently working on developing API using FASTAPI for NER. 
Exploring LangchainJs, HuggingFaceJs & TransformersJs.
Langchain code from few months ago does not work anymore due to rapid development & changes in this framework so exploring app development using Langchain framework.
Llamaindex calls a NER model for PII detection.

**Predefined or custom PII recognizers** leveraging Named Entity Recognition, regular expressions, rule based logic and checksum with relevant context in multiple language
- https://github.com/microsoft/presidio

Customizing presidio analyzer
- https://github.com/microsoft/presidio/blob/main/docs/samples//python/customizing_presidio_analyzer.ipynb
  
Existing NER approaches lack many entity types and are sometimes not accurate enough to completely remove the PII. `Presidio` is already integrated into `LangChain`:

- https://python.langchain.com/docs/guides/privacy/presidio_data_anonymization/
- Presidio Demo --> https://huggingface.co/spaces/presidio/presidio_demo
- https://microsoft.github.io/presidio/samples/

  ![](https://github.com/Mrunal-G/LLM_NER/blob/main/images/presidio_PII.png)

  ![](https://github.com/Mrunal-G/LLM_NER/blob/main/images/streamlit_PII.png)



## Web app development
- https://huggingface.co/spaces/Mrunal09/NER
- https://huggingface.co/spaces/Mrunal09/Multilingual-NER

  ![](https://github.com/Mrunal-G/LLM_NER/blob/main/images/ner1.png)

  ![](https://github.com/Mrunal-G/LLM_NER/blob/main/images/ner2.png)


============================================================================================
   Following approach discarded as it is not suitable to solve the PII detection problem.
   
==================================================================================

# Named Entity Recognition (NER) for PII detection ?

Most LLMs do not perform that well at Named Entity Recognition task out of the box. For the cost of inference, a model trained specifically for NER gives better results with predictable outputs and without hallucination. This was the approach that I had used for dealing with custom NER task in 2022 using transformer models by manaually labelling the data and then using supervised fine-tuning method. By investigating the current landscape of NER in 2024, I noticed that most of the recent approaches for creating custom NER systems are inclined towards leveraging LLM models for text-generation using prompting and instruction-tuning with zeroshot or few shot. I have investigated this recent approach for the given NER task because of custom named entity-types taxanomy and lack of availabliity of labeled data for custom NER.

## LLM text-generation for NER - How does this work ?

1. **Create instruction data** for NER using LLM like Google's gemini or gpt-3.5 for generating entity mentions and their corresponding entity-types with sample text for distilling LLM capabilities specifically for NER task. 

2. **Instruction-tuning** The data generated by LLM in above method could be used for `traditional supervised fine-tuning` like this [exmaple](https://github.com/Mrunal-G/anote_NER/blob/main/Fine_tuning_BERT_NER.ipynb). However, 2024 [UniversalNER research paper](https://universal-ner.github.io/) by Microsoft found `conversational style fine-tuning` performed beter than `NER-style tuning`. 

Note: The Universal NER Benchamark is the largest NER benchamark to date.

3. In the **zero-shot setting** the UniNER-13B model outperformed the ChatGPT and VVicunna-7B on NER task. 

Taking this into consideration, I decided to focus on zero shot instruction-tuning method.

This model performance could be improved furhter for domain-specififc fine-tuning with human labelled data.



# Starting off
Since we do not have lot of labled data with custom entities tags and lot of time for supervised learning, I started off with prompting the the Google's Gemini and OpenAI's GPT-3. Once I was happy with my prompt as seen below, I could essentially use the APIs for these models to inference on the provided demo csv. However, to save the cost of compute and inferencing, I have investigated open-source models for custom NER and their api creation using huggingface. 

```
Rewrite the text by identifying the entities and writing the entity type (the one-word PII) from the following 15 entity types described below in front of the entities in square brackets within original text.

[Name] - The complete name of an individual.
[Address] - The residential address of an individual.
[Email] - Personal email addresses.
[SSN] - A unique number assigned to individuals in the the United States for identification purposes.
[Passport] - A unique number assigned to a passport document.
[License] - A unique number assigned to a driver's license.
[CreditCard] - Numbers associated with personal credit cards.
[Birthdate] - The birth date of an individual.
[Phone] - Personal landline or mobile phone numbers.
[MedicalRecord] - Unique identifiers for personal medical records.
[Biometric] - Fingerprints, facial recognition patterns, DNA, etc.
[Vehicle] - License plate numbers, VIN Vehicle Identification Number, etc.
[InternetActivity] - IP addresses, cookie IDs, device identifiers, etc.
[Employment] - Employee ID number, work email, work phone number, etc.
[Education] - Student ID number, transcripts, etc.

**Example:** My IP address is 192.0.2.0 [InternetActivity], my device identifier is DI12345 [InternetActivity], and my cookie ID is CI12345 [InternetActivity].
```

One of the models I investigated for NER task using text-generation was UniversalNER LLM variations released by Microsft on the Huhhingface hub. UniversalNER returns a tuple of the entities from the entire text document for the user specified entity-type. However, the limitations of this model includes that I can only check for one entity type (eg. location) at a time in a text document. By using brute force for checking multiple entities type in a text docuemnt, I get a empty array of entities. UniversalNER has used NER datasets from various domains so it can identify the entities for entity-types like InternetActivity and MedicalRecord. But, no amount of changes in the Langchain prompt template can make the model output entity-type beside the entities in the original text because of the nature and design of the model. In conclusion, I decided to use text-generation techniques to solve the given NER task.  


### How to use F1 score evualuation metric for zero shot instruction tuning of LLM for NER

- Additionally, I also investigated a fine-tuned versoin of Meta's open-source Code LLama model for NER task which also uses text-generation technique by zero shot instruction tuning. My solution to the problem is based on [GoLLIE, a zero shot Information Extraction](https://github.com/hitz-zentroa/GoLLIE) as seen below by defining the annotation schema for custom entities type based on given PII taxanomy. The google colab notebook for the solution can be accessed [here](https://github.com/Mrunal-G/anote_NER/blob/main/NER.ipynb). 
- This solution also allows me to set the gold standard and and use F1 evaluation score metric, including full control on the instruction template as seen in this [example](https://github.com/hitz-zentroa/GoLLIE/blob/main/notebooks/Named%20Entity%20Recognition.ipynb). 

Example: 
Supervised Fine-tuning of UniNER-7B achieved average F1 score of 84.78% on 20 datasets surpasing BERT-base and InstructUIE-11B by 4.69% and 3.62% repsectively on the 20 datasets.

```python
from typing import List

from src.tasks.utils_typing import Entity, dataclass

"""
Entity definitions
"""


@dataclass
class Name(Entity):
    """Refers to The complete name of an individual with or without initials."""

    span: str  # Such as: "John H. Doe", "Jane Kim", "C.V. Raman", "Ashley"


@dataclass
class Address(Entity):
    """Refers to The residential address of an individual."""

    span: str  # Such as "456 Elm Avenue, Apartment 3B, Springfield, USA", "15 Rue de la Paix, Paris, France"


@dataclass
class Email(Entity):
    """Refers to Personal email addresses."""

    span: str  # Such as: "jane@gmail.com", "john.doe@hotmail.com", "", "Mercury", "Saturn"


@dataclass
class SSN(Entity):
    """Refers to the social security number - a unique number assigned to individuals in the United States for identification purposes."""

    span: str  # Such as: "987654321", "123-45-6789", "1234567890"

@dataclass
class Passport(Entity):
    """Refers to Passport Number - A unique number assigned to a passport document."""

    span: str  # Such as: "AB123456", "XY-123456789 (Issued: 2022-01-01, Expires: 2032-01-01)", "KL 876543"


@dataclass
class License(Entity):
    """Refers to Driver's License Number - A unique number assigned to a driver's license. """

    span: str  # Such as: "123456789", "DL-654321 (Issued: 2020-01-01, Expires: 2025-01-01)", "DL12-3456CD", "DL 987654"

@dataclass
class CreditCard(Entity):
    """Refers to Credit Card Numbers - Numbers associated with personal credit cards."""

    span: str  # Such as: "1234 5678 9012 3456", "9876 5432 1010 2020 (Expires: 12/25)", "4321-0987-6543-2109"

@dataclass
class Birthdate(Entity):
    """Refers to Date of Birth - The birth date of an individual."""

    span: str  # Such as: "05/25/1990", "25-May-1990", "May 25, 1990"


@dataclass
class Phone(Entity):
    """Refers to Telephone Number - Personal landline or mobile phone numbers."""

    span: str  # Such as: "Blue origin", "Boeing", "Northrop Grumman", "Arianespace"


@dataclass
class MedicalRecord(Entity):
    """Refers to Medical Record Numbers - Unique identifiers for personal medical records."""

    span: str  # Such as: "MRN123456", "MRN-98765-A", "MRN-456789 (Last updated: 2023-01-15)"


@dataclass
class Biometric(Entity):
    """Refers to Biometric Identifiers - Fingerprints, facial recognition patterns, DNA, etc."""

    span: str  # Such as: "BD12345", "ATCGATCGATCGATCG", "0123456789ABCDEF", "0x9876543210fedcba"



@dataclass
class Vehicle(Entity):
    """Refers to Vehicle Identifiers - License plate numbers, VIN (Vehicle Identification Number), etc."""

    span: str  # Such as: "ABC123", "1G1-BL52P0-TR123-456", "DEF-456", "1G1 BL52P0 TR123"


@dataclass
class InternetActivity(Entity):
    """Refers to Internet Activity - IP addresses, cookie IDs, device identifiers, etc."""

    span: str  # Such as: "192.0.2.1", "DI12345", "CI12345"

@dataclass
class Employment(Entity):
    """Refers to Employment Information - Employee ID number, work email, work phone number, etc."""

    span: str  # Such as: "E12345", "johndoe@company.com", "555-987-6543"

@dataclass
class Education(Entity):
    """Refers to Educational Records - Student ID number, transcripts, etc.."""

    span: str  # Such as: "S12345678", "DP-MSCS-2022", "CS108"


ENTITY_DEFINITIONS: List[Entity] = [
    Name,
    Address,
    Email,
    SSN,
    Passport,
    License,
    CreditCard,
    Birthdate,
    Phone,
    MedicalRecord,
    Biometric,
    Vehicle,
    InternetActivity,
    Employment,
    Education,
]

if __name__ == "__main__":
    cell_txt = In[-1]



```

### Inferencing

Explored huggingface text generation inferencing and api deployment methods.


- https://huggingface.co/docs/text-generation-inference/index
- https://www.youtube.com/watch?v=u-GE9_411g4

 ![hf inference ](https://github.com/Mrunal-G/LLM_NER/blob/main/images/pic.png)


### Challenges I faced

- However, I am currently trying to resolve the dependency conflict with CUDA & GPU architecture. Once I resolve this problem I intend to create a Docker image for the same and solve this issue once and for all. 
I have managed to resolve the issues arising due to conflicting installation of flash-attention. Overall, resolving dependency conflicts took most amount of my time.
- Apart from the dependency conflixt issues, a major problem with using the LLM for text-generation is that these models were so huge that I have to design my experiments based on the computational constraints of my laptop, kaggle gpu and free google colab. Apart from using the LLM models huggingface API for developing an app, I could not test these LLM models with my laptop GPU. So, most of my experiments are done using Kaggle notebooks and google colab pro version.
