> How to create/restore an account, who has sudo, soft- and hardware setups, ...
---

# Compute Resources

## Some of administrator responsibilities

* All servers must run Ubuntu release that is a) supported b) LTS (long-term). Best, if it is the latest LTS.
* CUDA should support the latest version of PyTorch.

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
| 172.16.33.17 | inanna.cs.uml.edu | 3x GTX 1080              | 10.2         |
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

## Installing drivers and CUDA

### Ubuntu 20.04

1. Remove stuff installed via apt-get
    ```bash
    sudo apt-get purge cuda
    sudo apt-get purge nvidia-cuda-toolkit
    sudo apt-get purge "cuda*"
    sudo apt autoremove
    ```
2. Go to [Nvidia website](https://developer.nvidia.com/cuda-downloads) select Linux, x86, Ubuntu, 20.04, deb (network). It will show you commands like these, execute them.

```bash
wget https://developer.download.nvidia.com/compute/cuda/repos/ubuntu2004/x86_64/cuda-ubuntu2004.pin
sudo mv cuda-ubuntu2004.pin /etc/apt/preferences.d/cuda-repository-pin-600
sudo apt-key adv --fetch-keys https://developer.download.nvidia.com/compute/cuda/repos/ubuntu2004/x86_64/7fa2af80.pub
sudo add-apt-repository "deb https://developer.download.nvidia.com/compute/cuda/repos/ubuntu2004/x86_64/ /"
sudo apt-get update
sudo apt-get -y install cuda
```
3. Reload the computer `sudo reboot`
4. Follow official Nvidia tutorial to install CUDNN. To download cudnn directly to the server using wget use [this hack](https://stackoverflow.com/questions/31279494/how-to-install-cudnn-from-command-line)
5. Check that `torch.cuda.is_available()` **and** that something like `torch.zeros(10).cuda()` works. Fix all warnings or errors if they appear.


#### Ubuntu < 20.04

The easiest way to install everything:
1. **Do not install via apt get**. It will cause a lot pain to the next person who will be updating the drivers.
2. **Do not install drivers manually**, let CUDA installer do it
3. **Use tmux** during the installation process. If you accidentally close your ssh session while intaller works, it's going to be a huge mess.
4. **(Important)** Delete previously installed drivers
    1. Remove stuff installed via apt-get
        ```bash
        sudo apt-get purge cuda
        sudo apt-get purge nvidia-cuda-toolkit
        sudo apt-get purge "cuda*"
        sudo apt autoremove
        ```
    1. Remove manually installed CUDA versions. Go to `/usr/local/cuda-XX.X/bin` and run something like `sudo chmod + cuda-uninstall; sudo sh cuda-uninstall`
    2. Remove CUDNN (`sudo rm -rf /usr/local/cuda`)
5. **Install/upgrade dependencies** `sudo apt-get install build-essential dkms`
6. Go to the pytorch website and look up the latest CUDA version they support.
7. Go to the NVIDIA website and download CUDA installer
8. **Allow file execution** `chmod + cuda_installer_name.run`
9. **Run it as root** `sudo sh cuda_installer_name.run`
10. Accept EULA and deselect "examples" and "developer" stuff. We only need the driver and CUDA itself
11. Follow the wizard instructions
12. At the end of the installation process, you going to see something like this

  ```bash
  ===========
  = Summary =
  ===========

  Driver:   Installed
  Toolkit:  Installed in /usr/local/cuda-11.2/
  Samples:  Not Selected

  Please make sure that
   -   PATH includes /usr/local/cuda-11.2/bin
   -   LD_LIBRARY_PATH includes /usr/local/cuda-11.2/lib64, or, add /usr/local/cuda-11.2/lib64 to /etc/ld.so.conf and run ldconfig as root

  To uninstall the CUDA Toolkit, run cuda-uninstaller in /usr/local/cuda-11.2/bin
  To uninstall the NVIDIA Driver, run nvidia-uninstall
  Logfile is /var/log/cuda-installer.log
  ```

13. To add CUDA to PATH, open `sudo vim /etc/environment` and replace `:/usr/local/cuda-VERSION.NUMBER/bin` with a new link. **Replace**, do not just add.
14. To update LD_LIBRARY_PATH, create file `sudo vim /etc/ld.so.conf.d/cuda.conf` with content `/usr/local/cuda-11.2/lib64`. After that execute `sudo ldconfig`
15. Install CUDNN (read recommendations below). Download it and unpack via `sudo tar zxvf cudnn-VERSION.tgz`
16. Reboot the server and check that `torch.matmul(torch.ones(3, 4, device="cuda"), torch.ones(4, 5, device="cuda"))` works
17. Check that `nvidia-smi` and `nvcc --version` work and show the same driver version (if they show different driver versions you fogtot to delete something and it is going to hurt you in the future)
18. **!!Update this guide!!** CUDA installation is always a pain and the details change from time to time. You will thank yourself in ~3 months.

If what you tried does not work, follow [this guide](https://gist.github.com/wangruohui/df039f0dc434d6486f5d4d098aa52d07)

Recommendations: 
  * Remember to `sudo apt-get install build-essential dkms` and `chmod +x` the installation file. Read the guide carefuly!
  * Do everything in tmux session in case ssh connection is interrupted (it will!)
  * Call `sudo ./cuda_10.2.89_440.33.01_linux.run` instead of `./cuda_10.2.89_440.33.01_linux.run`
  * Remember to install [cuDNN](https://developer.nvidia.com/cudnn)! Select "cuDNN Library for Linux" instead of "cuDNN Runtime Library for Ubuntu18.04 (Deb)" because you don't want to suffer deleteing .deb packages when reinstalling the CUDA next time. You cannot use `wget` to download cuDNN (because NVIDIA sucks) so download it to your computer and `scp` to the server.
  * To download cudnn directly to the server use [this hack](https://stackoverflow.com/questions/31279494/how-to-install-cudnn-from-command-line)
  * Untar cuDNN as `sudo` to create symlinks

Possible issues:
  * "Existing package manager installation of the driver found" - the problem is that somebody installed CUDA via package manager (you sick bastard!). You need to find what exactly is installed and delete is using `sudo apt-get purge <PACKAGE_NAME>` and to execute `sudo apt autoremove` after that. You can also try to do this (but the result and safety of these commands are not guaranteeed):
  * Unmet dependencies: look at what is exectly unmet and delete the packages that require something.

```bash
sudo apt-get purge cuda
sudo apt-get purge nvidia-cuda-toolkit
sudo apt-get purge "cuda*"
sudo apt autoremove
```

  * "Installation failed. See log at /tmp/cuda-installer.log for details" - see the log first. If this is driver error, just remove [x] from driver installation when installing cuda (we installed the driver at the previous step, remember?)

## Installing Apex

[Nvidia Apex](https://github.com/NVIDIA/apex) is a library that allows you to speed up training / reduce memory requirements in just a few lines of code (or no code at all!). It is used in many externali libraries (like fairSeq and Megatron) and just makes things better. Use it.

Follow installation instructions in the official README.md

Possible issues:
  * If your CUDA version does not match Apex version completely, installation will fail on purpose ("Cuda extensions are being compiled with a version of Cuda that does not match the version used to compile Pytorch binaries"). To get around this you can comment out this check in setup.py file and hope for the best.

## User management

In general, all user accounts should have the name following the template <first letter of first name><last name>.
For example, for a user Alexey Romanov, the username would be `aromanov`.

## Useful commands

* View all users: `getent passwd`
* View sudo users: `getent group sudo | cut -d: -f4`
* Delete user from sudo: `sudo deluser USERNAME sudo`
* Add user to sudo: `sudo usermod -aG sudo USERNAME`
* Add user: `sudo adduser USERNAME`
* Add a group: `sudo addgroup GROUPNAME`
* Set primary group: `usermod -g GROUPNAME USERNAME`

## Other resources

* Digital Ocean Droplets and university infrastructure (if you need to host a servise)
* Multiple GPUs (days of compute): just learn PyTorch Distributed to utilize multiple GPUs
    * [official guide](https://pytorch.org/tutorials/intermediate/dist_tuto.html) (which we do not recommend)
    * [related medium post](https://medium.com/huggingface/training-larger-batches-practical-tips-on-1-gpu-multi-gpu-distributed-setups-ec88c3e51255)
    * [lightning](https://towardsdatascience.com/how-to-refactor-your-pytorch-code-to-get-these-42-benefits-of-pytorch-lighting-6fdd0dc97538) module which makes it very easy to use
    * You can also use PyTorch DataParallel (and it is easier), but it is less efficient
    * **Note:** always test that your multi-GPU setup if actually faster than a single GPU on a small subset of your data
* [TensorFlow Research Cloud](https://www.tensorflow.org/tfrc) (months of compute): TFRC allows to access free TPUs for research purposes, consult with somebody from the lab before applying

## If you need help

You can ask any hardware-related questions in Slack #hardware channel
