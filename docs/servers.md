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
| 172.16.33.17 | inanna.cs.uml.edu | 2x RTX 3090              | 11.4         |
| 172.16.33.13 | enki.cs.uml.edu   | 2x Titan X               | 11.2         |
| 172.16.33.15 | shala.cs.uml.edu  | 1x RTX 3090, 1x GTX 1080 | 11.0         |
| 172.16.33.14 | ishkur.cs.uml.edu | 2x RTX 3090              | 11.0         |
| 172.16.33.9  | marduk.cs.uml.edu | 2x GTX 1080, 1x Titan X  | 11.0         |

CPU-only:

* dumuzi.cs.uml.edu

If you have problems connecting to a server or wish to create an account, please contact the corresponding administrator.
If you don’t currently have an account, you can use the teaching lab’s workstations in DAN417 (each one of them has a GPU).

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
