
# Weight sharing is the inverse of MoE.  

**22 june, 2024**

MoE/sparse models are less dense than traditional architectures where density is defined as the ratio of total active parameter divided by the number of parameters. What is usually called a "dense model" has a density of 1.0.  
  
MoEs usually reduce their density by routing tokens to different circuits within the model, increasing the total amount of trainable parameters while keeping the number of active ones constant.  
  

### Different levels of density 


If MoEs are less dense models, what would be a more dense model? (with density >1)  

It's a model where the weights are shared. For instance, a model where a transformer block would be repeated. Each parameter would be used several times, making them more active than in a traditional "dense" model.  
  
I put dense into a bracket here because it is actually abusing the word, there is no such thing as a dense and non-dense model, just a different density flavor.  
  
Why does it matter? **Because introducing density as a scale allows us to consider a whole new class of model architectures**.  
  
We need to move away from the idea that number of parameters should scale on the same path as the number of flops we allow a model to consume. There should be two axes to bigger models, more activations and more weights (more flops and more VRAM).

## Disentangle storage and computation 

Conceptually it actually makes sense to disentangle the storage (weights) from the computation (activations). The former dictates how much data can be compressed, and the latter controls the complexity of the algorithms that can be learned by increasing the number of flops that can be spent at each forward.  
  
Of course, in practice things are more complex, knowledge / fact and reasoning ability / algorithm are two sides of the same coins.  However, it can be handy to **visualize all of the functions learned by an LLM in a range between *purely retrieval* and *purely algorithmic*.** It would actually make sense that the former leverages more weights while the latter leverages more depth or more flops per forward.  
  
Let's take an example, for a model with an almost infinite storage capacity the easiest way to achieve low perplexity over, let's say a math textbook distribution, is to learn everything by heart. On the other hand, a model with limited storage but infinite flops at runtime will have to learn to predict everything from first principles.
  
**Having these two asymptotes in mind helps reason about the usefulness of different densities of models**.  
  
It is a known fact that small LLMs (sorry for the oxymoron) have a hard time following instructions. Many concluded that it is because of its too small amount of weight - it's missing a neuron or two -  lol I am funny. What if the real problem was just that the forward pass is too compute limited to be actually able to do anything "smart"?  


### MoE are shaped by hardware constraint 
  
Okay, enough for the "theoretical" thinking. MoE has emerged because of hardware constraints. They are simply cheaper to train and cheaper to do inference, *on our available hardware*. 

Sparse models have comparable capability to "dense" models at equal weight size, but with an order of magnitude less active parameters, and therefore, less time and less cost to do inference and training. 

Let's briefly take a look at the [Mixtral 8x7b](https://mistral.ai/news/mixtral-of-experts/) architecture to understand how hardware constraints shaped its design. (Feel free to skip directly to the conclusion)

Let's take a look to understand how it is hardware and cost that shape the design of MoE more than any other factor. Despite its name suggesting 8x7b = 56b parameters, the model is actually 46.7B parameters, with only 12.9B active parameters. Only the feed-forward layer of the transformer blocks are sparsified. Basically, instead of having one feedforward per transformer, there are 8 of them and each token will be directed to 2 feedforwards. Why 8 feedforwards and not 10 or 6? Simply because H100 nodes usually come with 8 GPUs. Each of the GPUs actually hold the equivalent of a 7b model (thus the naming of 8x7b) but with each of them using a different feedforward. 

Funny how this 8x7b magically fits into one node of 8x(H100 - 80gig) trained with optimizer and gradient shardings. Well, you would have guessed this is not a coincidence. 

TLDR; MoE designs are shaped by hardware constraints, they are cheaper to train than dense models only because some MoE architectures work nicely with high bandwidth inter and intra connect.



### Super dense model, any use case?

  
Okay, that was for the hardware constraint that led to the popularization of MoE. So what about the super dense model, a.k.a weight-shared model? Is there some hardware constraint that would make them relevant?  
  
Yes, local inference (from laptop to mobile). Basically, in local inference, you work most of the time with a batch size 1. Contrary to the model behind an API that receives a constant flow of thousands of users, locally there is only you using your model. Unfortunately, this is totally [inefficient for GPU-like chips](https://timdettmers.com/), and you end up being memory bandwidth bottlenecked most of the time. It is like having to move a pile of books from the library to your table, only to respond to one question of an exam. You would spend most of your time moving books away.

That means that your compute is barely used it is always waiting for the next weights to be loaded from ram/vram to local memory (SRAM).

![image](img.png){: style="height:60%;width:60%"}

GPU being idle means that you have room to do more operations for the same cost/time as long as you can reuse the loaded weight while waiting for the next batch of weight to be loaded. Meta actually published a paper about this, [MobileLLM](https://arxiv.org/abs/2402.14905). 

Nice explanation from the paper 

```
The SRAM for computing is typically limited to around 20MB. 
This capacity is usually only sufficient to hold a  single 
transformer block. Therefore, placing shared weights in the
cache and computing twice immediately can avoid the need to 
transfer weights between the SRAM and DRAM, resulting in improved 
overall execution speed for auto-regressive inference
```
  

Immediate weight sharing between two blocks can basically double the amount of active parameters while keeping inference time the same. 


### Conclusion:  
* MoEs are used for an economical reason, driven by hardware constraints.
* Let's stop talking about dense and not dense models, density is rather a scale. Different hardware constraints would lead to different density requirements.
* Super dense models with shared weights (and reasoning tokens?) could help create a very good small model that would still fit in ram/vram but rival the bigger models in term of performance.
* When a 4x1b or 4x8b with some block repeated 4 times for edge?
