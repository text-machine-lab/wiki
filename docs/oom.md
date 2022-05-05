# How to resolve OOM errors

The error looks like this
```
RuntimeError: CUDA error: out of memory
```

**If this happens during training**

1. Reduce batch size by 2 and increase gradient accumulation by 2.
1. Repeat step 2 until your batch size is 1.
1. If this doesn't work, use a beefier GPU, like 3090 or A100 if it is available to you.
1. Parallelize your model across multiple GPUs. In Transformers it can easily (but inefficiently) done via `.parallelize()`.

If you are still having issues, **advanced techniques** come in place.

**Use mixed precision**. Actually, you might want to use mixed precision even if you don't have OOM to allow for larger batch sizes and speed up training.
Note that some pre-trained models might not work well with mixed precision. Try to google if it is alright to use this parcitular model in fp16 mode (or bf16 mode).

On 1080 GPUs mixed precision technically works, but it only stores model parameters in this format, computations are still performed in fp32.
Meaning, it can reduce memory consumption, but it won't speed up the training.
Mixed precision works way better on 3090 GPUs, where tensor cores can do computations in fp16 too.
This both reduces memory consumption and speeds up training up to 2x.
Mixed precision is already included in recent versions of PyTorch and with Transformers Trainer,
but if you have an old version of PyTorch, you might need to install Nvidia Apex.

Other advanced techniques include **gradient checkpointing**, **weights offloding**, and using a different optimizer. Instead of ADAM you can try to use ADAM8 or AdaFactor. Note that AdaFactor requires a particular learning rate scheduling and seems to work well only with sequence-to-sequence models for some reason.

If all of this didn't work and even batch size 1 causes OOM, google for something like "training large models" or "training large models on a single GPU".
You might find DeepSpeed useful. Also, contact Vlad when you start trying these exotic methods.

### Summary:

1. Reduce batch size, increase gradient accumulation
2. Use larger GPUs
3. Use mixed precision
4. Google search exotic stuff

**During inference** steps, 1 and 2 still apply, but also you can turn off gradients in PyTorch via `torch.set_grad_enabled(False)`.
This should especially help with models that have a lot of layers.
If all of this didn't work, try mixed precision, but there's a good chance (I would say 50%) this may not work. Then you look for exotic stuff.

## Useful links

1. [ZeRO-Infinity and DeepSpeed: Unlocking unprecedented model scale for deep learning training](https://www.microsoft.com/en-us/research/blog/zero-infinity-and-deepspeed-unlocking-unprecedented-model-scale-for-deep-learning-training/)
2. [Getting Started with DeepSpeed for Inferencing Transformer based Models](https://www.deepspeed.ai/tutorials/inference-tutorial/)
3. [Training Neural Nets on Larger Batches: Practical Tips for 1-GPU, Multi-GPU & Distributed setups](https://medium.com/huggingface/training-larger-batches-practical-tips-on-1-gpu-multi-gpu-distributed-setups-ec88c3e51255)
4. [PyTorch Performance Tuning Guide](https://pytorch.org/tutorials/recipes/recipes/tuning_guide.html)
