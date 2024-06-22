# The bitter lesson of training LLM.

**28 may, 2024**

I worked for the last 6 months on training a 7b multilingual model from scratch at [nyonic.ai](https://nyonic.ai/) and the paper has just been published,
so I feel it is time to share a bit more insight of going throw the process.

## 0 - the results 

Se we trained a 7b multi lingual model, that is fluent in, english, french, german, spanish, italian, chinese, japanese and koreans (that is many, too many ? languages).

Raw performance are not nescerally impressive. It falls behind sota 7b model like mistral 7b or llama3. Spoiling alert, this is the bitter lesson of this post, if you want a better model you need more compute. Model was undertrained (1T), but take the same codebase we used, the same data mix, add a zero at the end of the check of our cloud provider and you get between mistral 7b (8T tokens for training) and a llama8b (15T tokens).

That being said, there is still a lot of useful learning of going throw the process of training a model, so I thought I would just share it.


## 1- not reinventing the wheel  

Training a 7b llm is more a engineering endeavor that a research one. It is about repeating the process describe in many papers on how to scale transformers. It is about
rigouros engineering, clean, maintanable and performant code, proper infrastructure work and good evaluation pipeline.



The model was trained for around 1T tokens on douzens of interconnected nodes of 8 times a100.

### tokenizer 


### monitoring for instability

### preventing instability

### datamix 

### multi plexer


### evaluation piplein


## What I would do differently

As any engineering process, even more when it is research driven, it was not perfect and we did mistake when training our LLM. 

### Evaluation pipeline.

Evaluation pipeline is crucial to train a good model, don't wait until your model is trained to evaluate it.
Make the evaluation pipeline, fast. You cannot wait one week for your model evaluation to come back.

Make it fast. Implement an okay fast inference pipeline on which you can plug your evaluation. Distil your evaluation dataset into a smaller one that give you a good proxy for the real performance. Include in training model evaluation. I am not talking about tracking the loss or the perplexity. Include model inference evaluation during training on out of distrubton dataset. 


need short spike of compute, cite prime intelect. 


### PS not beeing cited as author and nyonic drama
























