> How to create/restore an account, who has sudo, soft- and hardware setups, ...
---

# Compute Resources

## Some of administrator responsibilities

* All servers must run Ubuntu release that is a) supported b) LTS (long-term). Best, if it is the latest LTS.
* CUDA should support the latest version of PyTorch.
* When adding a new user, the administrator needs to add them to `hf_cache_users` group.

## HuggingFace Cache management

We're currently introducing a centralized cache management directory to our servers.
It is implemented via environment variables `TRANSFORMERS_CACHE` and `HF_DATASETS_CACHE` that are set in `/etc/environment`.

```bash
TRANSFORMERS_CACHE="/home/hf_cache/transformers_cache"
HF_DATASETS_CACHE="/home/hf_cache/datasets_cache"
```

> The location of `hf_cache` might be different on different servers. We try to put it on the largerst drive.

The directory `hf_cache` should have the group `hf_cache_users` assigned. This is how we set it up:

```bash
sudo groupadd hf_cache_users  # create the group
sudo chgrp -R hf_cache_users hf_cache  # assign the directory to the group
sudo chmod -R 2775 hf_cache  # allow all group users to read, write and execute files in it
for user in user names separated by space;
    do sudo usermod -a -G hf_cache_users $user;  # add users to the group
    done
```

## Installing drivers and CUDA

### Ubuntu >= 20.04

**EXTREMELY IMPORTANT: Use tmux** or screen to run the installation or you can corrupt the whole server if you accidentally close the terminal or lose connection.

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

## User management

In general, all user accounts should have the name following the template <first letter of first name><last name>.
For example, for a user Alexey Romanov, the username would be `aromanov`.

When creating a user do the following:

```bash
sudo adduser <username>  # create a user
sudo passwd -e <username>  # require the user to change their password on the first login
sudo usermod -a -G hf_cache_users <username>  # add the user to the group that has access to the Huggingface cache
```

## Useful commands

* View all users: `getent passwd`
* View sudo users: `getent group sudo | cut -d: -f4`
* Delete user from sudo: `sudo deluser USERNAME sudo`
* Add user to sudo: `sudo usermod -aG sudo USERNAME`
* Add user: `sudo adduser USERNAME`
* Add a group: `sudo addgroup GROUPNAME`
* Set primary group: `usermod -g GROUPNAME USERNAME`
* Expire password: `sudo passwd -e USERNAME`
