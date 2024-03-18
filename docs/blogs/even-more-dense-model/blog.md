# Even more dense model, Weight tying as the inverse of mixture of expert

**18 mars, 2024**

---

tl;dr: Weight tying can be seen as the inverse of mixture of expert, producing even denser model, making it crucial choice for on the edge deployment where memory is a bottleneck.

MoE (Mixture of expert) models does not use all of their weights at each forward. Allowing to train _bigger model_ (up to trillion scale) while keeping the flops utilization bounded at the cost of a worse memory footprint.

MoE are **sparse** model, in opposition to **dense** model. But what would be the _inverse_ of a sparse model ? What is a **more dense** model ? 


The quick answers is that that weight tying is basically the inverse of MoE. MoE trade flops for memory, weight tying is the other way around. 

## What is weight tying ?

Weight tying is very simple idea, decoder only model are stacking many transformer blocks, weight tying just mean to have the same weight for some of the block, basically _repeating_ the same block multiple times.

Meta has shown empirically that it helps with performance  in [MobileLLM](https://arxiv.org/abs/2402.14905).

In this post I will rather focus on the concept of density of weight rather than showing any kind of results (lazy me).

_Bigger models outperforms smaller ones_,  at least if trained on the same amount of data, that is one of the conclusion of the [scaling law](https://arxiv.org/abs/2001.08361).

What does bigger model really means ? Usually it means bigger hidden dimension and more stack layers. So you increase both the amount weight available for the model, as well as the number of operation needed to perform a forward pass. What if you could just increase the later, the amount of compute that a model can do, while keeping the amount weight constant ? This is what weight tying is doing.



It is important to decouple the two concepts, a bit of intuition here: more weight means more storage capacity, but storage is not all you need, you need to be able to preforms some kind of sequential operations over the stored/compressed data, this is what more amount of compute needed give you.

Adding more layer basically increase both concept at the same time, weight tying is only increasing the amount of compute needed to perform a forward pass while keeping the amount of weight constant.

## What is MoE ?

My goal here is not to fully explain how Mixture of expert (MoE) are trained, but rather to give a mental framework to understand why they are working. Fore more in depth MoE resources you can take a look at the [GShard paper](https://arxiv.org/abs/2006.16668), the [mixtral paper](https://arxiv.org/abs/2401.04088) or this HF blog post.

MoE are sparse model, at the feed forward layer, instead of having one block, there are 8 blocks, each token will go throw one of the block. It is sparse because not all of the weight will be used for each forward, it will depend on the input token. They are no expertise in each Block (see the [mixtral paper](https://arxiv.org/abs/2401.04088)). Mixture of Expert is actually a misleading name, you should rather see it as a gated + distributed feed forward.

So if we take 


## Memory vs compute bounded 


## 

maybe weight tying is not the solution, maybe you still want to train the model in a MoE fashion, and you will just do an hardcore quantization at the end (see the [QMOE paper](https://arxiv.org/abs/2310.16795)). But here I was trying to propose a way to understand MoE and weight tying by linking the design choice to hardware requirements.

# Reference

- [Mixture of Expert](https://huggingface.co/blog/moe)
- [GShard: Scaling Giant Models with Conditional Computation and Automatic Sharding](https://arxiv.org/abs/2006.16668)
- [MobileLLM: Optimizing Sub-billion Parameter Language Models for On-Device Use Cases](https://arxiv.org/abs/2402.14905)
- [Mixtral of Experts](https://arxiv.org/abs/2401.04088)
- [Scaling Laws for Neural Language Models](https://arxiv.org/abs/2001.08361)
- [QMoE: Practical Sub-1-Bit Compression of Trillion-Parameter Models](https://arxiv.org/abs/2310.16795)




























