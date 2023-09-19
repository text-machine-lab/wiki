> How to use TextMachine servers
---

# Compute Resources

## Servers

We currently have 5 servers with GPUs installed and 1 server with a lot of memory but no GPU.
All servers run Ubuntu, so you should be comfortable constantly typing commands in a terminal.
A quick reference of the most used bash commands can be found here.
To connect to a server, you need to be within the UML network: this can be done using a PulseSecure VPN client,
or by first logging into “cs” server using your cs credentials and then jumping to one of our servers. 
For establishing an ssh connection, you can either use IPs or server names as follows:

GPU-enabled:

| IP           | Domain            | GPUs                     | CUDA version |
|--------------|-------------------|--------------------------|--------------|
| 172.16.33.17 | inanna.cs.uml.edu | 2x RTX 3090              | 11.8         |
| 172.16.33.13 | enki.cs.uml.edu   | 1x RTX 3090              | 12.1         |
| 172.16.33.15 | shala.cs.uml.edu  | 2x RTX 3090              | 12.0         |
| 172.16.33.14 | ishkur.cs.uml.edu | DOWN                     | N/A          |
| 172.16.33.9  | marduk.cs.uml.edu | 2x Titan X               | 11.0         |
| <ask Vlad>   | ml1 <ask Vlad>    | 7x A6000 Ada             | 12.2         |

CPU-only:

* dumuzi.cs.uml.edu

If you have problems connecting to a server or wish to create an account, please contact the corresponding administrator.
If you don’t currently have an account, you can use the teaching lab’s workstations in DAN417 (each one of them has a GPU).

### Some special things about ML1 server (the 7x A6000 Ada)

* **Do not** use your cs account home directory to store data or models
* Use local server storage instead, it is located in `/home/public`
* Create your user directory `mkdir /home/public/$(whoami)`
* Use local conda. To activate it, you can add this to your bashrc `source /home/public/source_conda.sh`
   * Double-check that it works and your `which conda` shows you `/home/public/miniconda3/bin/conda`
   * If it doesn't work, ask Vlad how to fix it
* Do `export NCCL_P2P_DISABLE=1` before starting a distributed run. This is a known bug in [A6000](https://discuss.pytorch.org/t/single-machine-ddp-issue-on-a6000-gpu/134869/15)
* (Optionally) change your Huggingface cache directory by adding this to your `.bashrc`:
```
TRANSFORMERS_CACHE="/home/public/$(whoami)/transformers_cache"
HF_DATASETS_CACHE="/home/public/$(whoami)/datasets_cache"
```
* Please notice that `/home/public` storage is very limited. It is less than 4 TB for all ML1 users. Do not save unnecessary checkpoints, clean up your experiment directories regularly, and occasionally delete all contents of your Huggingface cache directories. Also, do not store terabyte-sized datasets there without consulting with Vlad or Anna.
* Currently, the server only has 7GPUs, which significantly limits your batch sizes. If using distributed, I recommend no more than 4 GPUs
* These GPUs are super cool, but 7 of them is not a lot, so
  * Make sure your runs use 100% of the GPU and as much memory as possible
  * This is important, because while you are using this GPU, other people can't (never run more than one job on the same GPU)
* I expect several people may want to use ML1, please make sure you are correctly specifying GPUs in your  CUDA_VISIBLE_DEVICES when starting a run (e.g, if the GPUs 0 and 1 are occupied and you need two GPUs do `export CUDA_VISIBLE_DEVICES=2,3`


### Detailed Shala spec

| thing           | thing spec        |
|-----------------|-------------------|
| CPU	            | Intel Boxed Core i7-6850K |
| Motherboard    	| ASRock ATX DDR4 Motherboard | 
| RAM.           	| 64GB DDR4 2400 |
| Full Tower Case	| Corsair Obsidian Series 750D |
| SSD	            | Samsung 850 EVO 500GB |
| HDD	            | WD Red Pro 3TB |
| Power Supply   	| EVGA Supernova G2 1300W |
| CPU Cooler	     | Cooler Master Hyper 212 EVO |
| GPU      	      | EVGA GeForce GTX 1080 |

### Detailed Inanna spec

https://pcpartpicker.com/list/NQ8gD2


## How to select a GPU on a server

To select a GPU on a server, you need to set the CUDA_VISIBLE_DEVICES environment variable.
For example, to use the first GPU on the server, you can run:

```bash
export CUDA_VISIBLE_DEVICES=0
```

Notice that the order of the GPUs might not be the same as in `nvidia-smi`.
Basically `nvidia-smi` shows the order of the GPUs as they are physically connected to the server
while `CUDA_VISIBLE_DEVICES` ranks them by how amazing they are (not really, but this is the simplest way to look at it).
You can change this behavior by setting the `CUDA_DEVICE_ORDER` environment variable to `PCI_BUS_ID` in your `.bashrc` or `.zshrc` which will make
`CUDA_VISIBLE_DEVICES` use the same order as `nvidia-smi`.

You can also use the `CUDA_VISIBLE_DEVICES` variable to select multiple GPUs.
```bash
export CUDA_VISIBLE_DEVICES=0,1
```

## Other resources

* If you need to host a servise you can use Digital Ocean Droplets and university infrastructure. Ask Anna about it.
* How to use multiple GPUs for training:
    * [official guide](https://pytorch.org/tutorials/intermediate/dist_tuto.html) (which we do not recommend)
    * [related medium post](https://medium.com/huggingface/training-larger-batches-practical-tips-on-1-gpu-multi-gpu-distributed-setups-ec88c3e51255)
    * [lightning](https://towardsdatascience.com/how-to-refactor-your-pytorch-code-to-get-these-42-benefits-of-pytorch-lighting-6fdd0dc97538) module which makes it very easy to use
    * You can also use PyTorch DataParallel (and it is easier), but it is less efficient
    * **Note:** always test that your multi-GPU setup if actually faster than a single GPU on a small subset of your data
* [TensorFlow Research Cloud](https://sites.research.google/trc/about/) allows to access free TPUs for research purposes, consult with somebody from the lab before applying.

## If you need help

You can ask any hardware-related questions in Slack #hardware channel
