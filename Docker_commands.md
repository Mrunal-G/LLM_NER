# docker

python = 3.9.18
> pip install transformers[serving]


import torch
print(torch.__version__)
2.0.1+cu117

> transformers-cli --help

 serve -> CLI tool to run inference requests through REST and GraphQL endpoints.

> transformers-cli serve --help


usage: transformers-cli <command> [<args>] serve [-h]
                                                 [--task {audio-classification,automatic-speech-recognition,conversational,depth-estimation,document-question-answering,feature-extraction,fill-mask,image-classification,image-segmentation,image-to-text,ner,object-detection,question-answering,sentiment-analysis,summarization,table-question-answering,text-classification,text-generation,text2text-generation,token-classification,translation,video-classification,visual-question-answering,vqa,zero-shot-audio-classification,zero-shot-classification,zero-shot-image-classification,zero-shot-object-detection}]
                                                 [--host HOST] [--port PORT] [--workers WORKERS] [--model MODEL]
                                                 [--config CONFIG] [--tokenizer TOKENIZER] [--device DEVICE]

optional arguments:
  -h, --help            show this help message and exit
  --task {audio-classification,automatic-speech-recognition,conversational,depth-estimation,document-question-answering,feature-extraction,fill-mask,image-classification,image-segmentation,image-to-text,ner,object-detection,question-answering,sentiment-analysis,summarization,table-question-answering,text-classification,text-generation,text2text-generation,token-classification,translation,video-classification,visual-question-answering,vqa,zero-shot-audio-classification,zero-shot-classification,zero-shot-image-classification,zero-shot-object-detection}
                        The task to run the pipeline on
  --host HOST           Interface the server will listen on.
  --port PORT           Port the serving will listen to.
  --workers WORKERS     Number of http workers
  --model MODEL         Model's name or path to stored model.
  --config CONFIG       Model's config name or path to stored model.
  --tokenizer TOKENIZER
                        Tokenizer name to use.
  --device DEVICE       Indicate the device to run onto, -1 indicates CPU, >= 0 indicates GPU (default: -1)

> transformers-cli serve --task=ner --model=bert-base-uncased


we got the server up and running. Now the question is how to use this server. What are the endpoints.
There is not much documentation on this. So had to look at the source code to figure this out.

https://github.com/huggingface/transformers/blob/main/src/transformers/commands/serving.py

We are goint to be using the /forward APIRoute which is both the tokenization and the forward pass.


Now send the GET request to the server. you should get status 200 ok response

PS C:\Users\mruna> curl http://localhost:8888


StatusCode        : 200
StatusDescription : OK
Content           : {"infos":{"return_dict":true,"output_hidden_states":false,"output_attentions":false,"torchscript":f
                    alse,"torch_dtype":null,"use_bfloat16":false,"tf_legacy_loss":false,"pruned_heads":{},"tie_word_emb
                    ed...
RawContent        : HTTP/1.1 200 OK
                    Content-Length: 1824
                    Content-Type: application/json
                    Date: Fri, 08 Mar 2024 00:37:56 GMT
                    Server: uvicorn

                    {"infos":{"return_dict":true,"output_hidden_states":false,"output_attenti...
Forms             : {}
Headers           : {[Content-Length, 1824], [Content-Type, application/json], [Date, Fri, 08 Mar 2024 00:37:56 GMT],
                    [Server, uvicorn]}
Images            : {}
InputFields       : {}
Links             : {}
ParsedHtml        : mshtml.HTMLDocumentClass
RawContentLength  : 1824

> curl -X POST http://localhost:8888/forward -H "accept: application/json" -H "Content-Type: application/json" -d '{"inputs": "My name is Mrunal"}'


At this point you realize that bert-base-uncased is a normal transformer model which need to be finetuned for downstream NER model. So, now I would use the write token from huggingface account and access the fine-tuned NER models on huggingface models hub. 
install following 2 repositories 

> pip install git+https://github.com/huggingface/datasets.git
> pip install git+https://github.com/huggingface/transformers.git

> from huggingface_hub import notebook_login

> notebook_login()


!pip install -U transformers
!pip install -U accelerate
!pip install -U datasets


### Installations are exhausting to switch to using Docker

https://hub.docker.com/r/huggingface/transformers-pytorch-gpu

> docker pull huggingface/transformers-pytorch-gpu

> docker images | grep transformers

> docker build -t cool-api:v1 .


> docker images | grep cool-api:v1



