# Building a deep learning rig, part-1

**03 february, 2024**

I just got my hands on a mining rig with 3 rtx 3090 founder edition for the modest sum of 1.7k euros.

My plan is to transform it into a deep learning ring, to finetune and serve LLM, play with torch distributed with some MoE as well as doing a bit of independent research.


![Minning rig](rig_0.jpg)


### Detailed:

* rtx 3090 (x3)
* Ryzen 5 1600
* b450 steel legend
* RAM 4gig (lol)
* Cooler master 750w silver (x2) 

Bit of an insane deal, if it was only for the cards it would have cost 560 euro (=1700/3) per cards. In 2024 the price for a second hand 3090 is around 600 / 700. Plus I got all of the other spare part, that I might need to replace. Fun fact two years ago this rig probably would have cost like 7k even with spare part.

## Deep learning and inter card bandwidth.

The current rig is a mining rig. The three 3090s are connected to the mobo via pci 1X extender. This is totally inefficient for deep learning. 

PCI lines are doing the bridge between the the different part of the computer so that they can communicate, a normal gamer pc has usually 24x lines and the gpu is usually using 16x lines. Using 1x mean dividing the normal bandwidth by 16x. It apparently does not matter for crypto mining. Probably because is that in crypto gpu are used to compute the "proof of work" which is basically some brut force algorithm and the bandwidth does not matter, everything stay within the card. But deep learning model take data in (by batch), there is a lot of communication needed between the CPU that pre-process the data and the GPU, thus PCI lines matter.


# Inter GPU bandwidth, choosing the number of PCI lines  

By looking in the the [bible](https://timdettmers.com/2023/01/30/which-gpu-for-deep-learning/) of consumer grade deep learning, we can see that PCI lines x4 "should be enough". 

```
Operating GPUs on 4x lanes is fine, especially if you only have 2 GPUs. For a 4 GPU setup, I would prefer 8x lanes per GPU, but running them at 4x lanes will probably only decrease performance by around 5-10% if you parallelize across all 4 GPUs.
```


The CPU on the rig support up to 24 PCI lines and the mobo support [bifurcation](https://timdettmers.com/2023/01/30/which-gpu-for-deep-learning/#Do_I_need_PCIe_40_or_PCIe_50), aka you can split the main x16 lines into 4x4 pci ones. Meaning I could plug my for cards on my main mobo using a PCI riser like this [one](https://riser.maxcloudon.com/en/bifurcated-risers/22-bifurcated-riser-x16-to-4x4-set.html) that I found recommended by this excellent deep learning rip [blog post](https://nonint.com/2022/05/30/my-deep-learning-rig/).

It would mean that I only need to add 150 euros more (actually 200 with the shipping cost)  and got my deep learning rig ready. Would have been the cheapest deep learning rig of history.

The alternative is to go with a CPU which has many more cpu lines (the ryzen 5 1600 has only 16) as well as a mobo with at least 3 gpu slots. Problem is even the high end ryzen 9 or intel i9 have only 24 cpu lines ... So I would have to go with a AMD Epyc or Threadripper which are not cheap.

In an ideal word the first option would work out.

## Is x4 lines really okay for deep learning with LLM ?

This might depend on the type of GPU parallelism  I want to use. 

### DDP

Few years ago the GPU parallelism was mainly about DDP: distributed data parallelism. The model is replicated on each gpu device, the data is split per gpu, each GPU do a normal forward backward pass on its data, compute the gradient. Then Each GPU shared their gradient via an [All reduce](https://pytorch.org/docs/stable/distributed.html) communication (using [nccl](https://developer.nvidia.com/nccl)) and each GPU update its internal weights. 


### FSDP

Large language model are, as they name suggest, larger that their non generative counter parts. GPT3 is 175B parameters, some model even go up to the trillion scale, though usually using some sparse setup (Mixture of Expert) so not really relevant for our calculation.

Nowadays good and large LLM like [llama2](https://arxiv.org/abs/2307.09288) is around 70b.

It means that even in int 8 precision the model weight are still 70 GB. The 3090 only have 24gb, so one model does not even fit, not even talking about training. 

In this case we need to split the model in chunk. They are multiple way to do this:

* **Pipeline parallelism**: The model is split in chunks, each GPU hold part of the layers. Communication between GPU happened during forward and backward each time that it need to go to the next chunk. Let's say that we split the model on 4 gpus.

During forward you need to use the [send](https://pytorch.org/docs/stable/distributed.html) operation 3 times because you have 4 chunks. Each send is sending an enter activation.

* **Tensor parallelism**:  Tensor parallelism split the weight of each layer on each gpu. If Pipeline parallelism is splitting the model horizontally tensor parallelism is splitting it vertically. The communication scheme is slightly more complex, I don't fully get, but basically at each layer you need to do a mix of [All-gather](https://pytorch.org/docs/stable/distributed.html) and [All-reduce](https://pytorch.org/docs/stable/distributed.html) operation. 


At large scale this two strategy are used alongside data parallelism, that is called 3d parallelism. Checkout [this blog post](https://github.com/stas00/ml-engineering/tree/master/model-parallelism#dppptp) for more inf. 

In my use case only one of this two strategy will be used alongside DDP.

The conclusion is that such parallelism need more inter gpu communication that pure DDP. **So while PCI lines x4 per gpu might be fine for pure DDP, it might be a huge bottleneck for finetune 70b model**, even worse for local inference that is memory bounded. 

Additionally I want to play with Mixture Of Expert, which are sparse model, a.k.a not all weight are used during each forward.  Each "expert" is host on a different GPU and a router dispatch each token into the expert. This of course means a lot more communication that normal DDP.


## Conclusion

So I am a bit puzzled, using the 4x pcie Lanes **should work**, but I will be limited for anything that is not DDP like finetune LLM or MoE.

I will investigate the threadripper direction, if it is cheap enough it is probably the best solution, especially if I plan to add a 4 gpus later.

# Reference

* Tim Dettmers - [Which GPU(s) to Get for Deep Learning: My Experience and Advice for Using GPUs in Deep Learning](https://timdettmers.com/2023/01/30/which-gpu-for-deep-learning/)

* [PCIe Bifurcation – What is it? How to enable?](https://shuttletitan.com/miscellaneous/pcie-bifurcation-what-is-it-how-to-enable-optimal-configurations-and-use-cases-for-nvme-sdds-gpus/)

* [TORCH.DISTRIBUTED](https://pytorch.org/docs/stable/distributed.html)


* [nccl](https://developer.nvidia.com/nccl)

* [What Every User Should Know About Mixed Precision Training in PyTorch](https://pytorch.org/blog/what-every-user-should-know-about-mixed-precision-training-in-pytorch/) 

* [Llama 2](https://arxiv.org/abs/2307.09288)

