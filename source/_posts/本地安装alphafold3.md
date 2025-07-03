---
title: 本地安装alphafold3
date: 2025-06-23 15:41:18
categories:
  - MD
tags:
  - MD
---

# 本地安装 alphafold3

关于这个 docker 的安装教程我无法按照教程实现 rootless docker，因此我只能用 docker 组来实现。原教程在: [安装 alphafold3](https://github.com/google-deepmind/alphafold3/blob/main/docs/installation.md)

## 添加 sudo 权限

给某个用户添加 sudo 权限, 需要 sudo 权限来安装软件。

```bash
sudo usermod -aG sudo username
```

## 添加 docker 的官方权限

### Ubuntu 22.04

```bash
sudo apt-get update
sudo apt-get install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc
```

### openSUSE

openSUSE 不需要设置什么权限

<!--more-->

## 安装 docker

### Ubuntu 22.04

这是在添加源并且安装 docker

```bash
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update
sudo apt-get install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
sudo docker run hello-world
```

### openSUSE

```bash
sudo zypper ref
sudo zypper install docker

# openSUSE需要手动启动docker
sudo systemctl enable docker # 设置开机启动
sudo systemctl start docker # 启动docker
```

## 将用户添加进 docker 组

```bash
sudo usermod -aG docker username
```

添加入 docker 组后要重启下 terminal 使其生效。

接下来我们就能不用 sudo 使用 docker 命令了。

可以用以下命令来测试 docker 是否安装完成。

```bash
docker run hello-world
```

这个命令会从 docker hub 拉下一个 hello-world 的 docker 镜像。

## 安装 NVIDIA 驱动

### Ubuntu 22.04

如果还没安装驱动则用以下命令安装 (ubuntu 安装 NVIDIA 驱动好简单啊)。

```bash
sudo apt-get -y install alsa-utils ubuntu-drivers-common
sudo ubuntu-drivers install

sudo nvidia-smi --gpu-reset

nvidia-smi  # Check that the drivers are installed.
```

### openSUSE

添加 NVIDIA 的官方源

```bash
sudo zypper addrepo --refresh 'https://download.nvidia.com/opensuse/leap/$releasever' NVIDIA
sudo rpm --import https://www.nvidia.com/en-us/drivers/unix/nvidia-public.key
```

安装编译工具

```bash
sudo zypper install -t pattern devel_kernel
```

安装 NVIDIA 驱动

```bash
sudo zypper install x11-video-nvidiaG06
sudo reboot
nvidia-smi
```

## 为 docker 添加 NVIDIA 支持

### Ubuntu 22.02

```bash
curl -fsSL https://nvidia.github.io/libnvidia-container/gpgkey | sudo gpg --dearmor -o /usr/share/keyrings/nvidia-container-toolkit-keyring.gpg \
  && curl -s -L https://nvidia.github.io/libnvidia-container/stable/deb/nvidia-container-toolkit.list | \
    sed 's#deb https://#deb [signed-by=/usr/share/keyrings/nvidia-container-toolkit-keyring.gpg] https://#g' | \
    sudo tee /etc/apt/sources.list.d/nvidia-container-toolkit.list
sudo apt-get update
sudo apt-get install -y nvidia-container-toolkit
sudo nvidia-ctk runtime configure --runtime=docker --config=/etc/docker/daemon.json
```

### openSUSE

添加库

```bash
https://nvidia.github.io/libnvidia-container/stable/rpm/nvidia-container-toolkit.repo
sudo rpm --import https://nvidia.github.io/libnvidia-container/gpgkey
```

安装 nvidia-container-toolkit

```bash
sudo zypper install -y nvidia-container-toolkit
```

## 获得 AlphaFold3 的代码

```bash
git clone https://github.com/google-deepmind/alphafold3.git
```

## 下载通用数据

```bash
cd alphafold3  # Navigate to the directory with cloned AlphaFold 3 repository.
./fetch_databases.sh [<DB_DIR>]
```

如果直接只写`./fetch_databases.sh`的话就会下载到家目录下的`public_databases`下。

## 获得模型参数

需要填这个[表](https://docs.google.com/forms/d/e/1FAIpQLSfWZAgo1aYk0O4MuAXZj8xRQ8DafeFJnldNOnh_13qAx2ceZw/viewform)来获得模型参数，这需要 2-3 天时间。我至今也没得到这个参数文件。

## 构建 AlphaFold3 的 docker 镜像

```bash
docker build -t alphafold3 -f docker/Dockerfile .
```

## 运行测试

在家目录下新建`af_input`, `af_output`, 通用数据在`~/public_databases`下，模型参数在`~/models`下。

```bash
cd ~
mkdir af_input
mkdir af_output
```

这是官网给的[example 文件](https://github.com/google-deepmind/alphafold3?tab=readme-ov-file#installation-and-running-your-first-prediction)

```json
{
  "name": "2PV7",
  "sequences": [
    {
      "protein": {
        "id": ["A", "B"],
        "sequence": "GMRESYANENQFGFKTINSDIHKIVIVGGYGKLGGLFARYLRASGYPISILDREDWAVAESILANADVVIVSVPINLTLETIERLKPYLTENMLLADLTSVKREPLAKMLEVHTGAVLGLHPMFGADIASMAKQVVVRCDGRFPERYEWLLEQIQIWGAKIYQTNATEHDHNMTYIQALRHFSTFANGLHLSKQPINLANLLALSSPIYRLELAMIGRLFAQDAELYADIIMDKSENLAVIETLKQTYDEALTFFENNDRQGFIDAFHKVRDWFGDYSEQFLKESRQLLQQANDLKQG"
      }
    }
  ],
  "modelSeeds": [1],
  "dialect": "alphafold3",
  "version": 1
}
```

将他保存到`~/af_input/fold_input.json`里，然后运行以下的文件进行测试。

```bash
docker run -it \
    --volume $HOME/af_input:/root/af_input \
    --volume $HOME/af_output:/root/af_output \
    --volume $HOME/models:/root/models \
    --volume $HOME/public_databases:/root/public_databases \
    --gpus all \
    alphafold3 \
    python run_alphafold.py \
    --json_path=/root/af_input/fold_input.json \
    --model_dir=/root/models \
    --output_dir=/root/af_output
```

## 安装 singularity

因为要运行 docker 命令的话需要加入 docker 组，为了更方便使用，我们生成 singularity 镜像。

### Ubuntu 22.04

```bash
wget https://github.com/sylabs/singularity/releases/download/v4.2.1/singularity-ce_4.2.1-jammy_amd64.deb
sudo dpkg --install singularity-ce_4.2.1-jammy_amd64.deb
sudo apt-get install -f
```

这三个命令首先下载`.deb`包，然后用`dpkg`去安装 (可能会一些错误)，最后用`apt-get`解决依赖问题。

## 由 docker 镜像生成 singularity 镜像

先生成一个私有仓库：

```bash
docker run -d -p 5000:5000 --restart=always --name registry registry:2
docker tag alphafold3 localhost:5000/alphafold3
docker push localhost:5000/alphafold3
```

然后使用 singularity 生成容器：

```bash
SINGULARITY_NOHTTPS=1 singularity build alphafold3.sif docker://localhost:5000/alphafold3:latest
```

## 运行测试

可以运行以下的命令进行测试：

```bash
singularity exec \
      --nv
      --bind $HOME/af_input:/root/af_input \
      --bind $HOME/af_output:/root/af_output \
      --bind $HOME/models:/root/models \
      --bind $HOME/public_databases:/root/public_databases \
      alphafold3.sif \
      python /app/alphafold/run_alphafold.py \
      --json_path=/root/af_input/fold_input.json \
      --model_dir=/root/models \
      --db_dir=/root/public_databases \
      --output_dir=/root/af_output
```

> 教程里也没告诉`run_alphafold.py`在`/ap/alphafold/`

对应地，qsub 中的提交脚本为：

```bash
#!/bin/bash
#$ -S /bin/bash
#$ -cwd
#$ -V
#$ -pe mpi 32
#$ -q all.q@helix.local
#$ -N alphafold3
#$ -e stdout
#$ -o stdout

model_dir="<YOUR_MODEL_DIRECTORY>"
af_input_dir="<YOUR_AF_INPUT_DIRECTORY>"
af_output_dir="<YOUR_AF_OUTPUT_DIRECTORY>"
input_json="<INPUT_JSON>"

for i in $model_dir $af_input_dir $af_output_dir $input_json
do
    if [[ "$i" =~ \<.*\> ]]; then
        echo "Please set ${i} directory!" 1>&2
        exit -1
    fi
done

singularity exec \
    --nv \
    --bind ${af_input_dir}:/root/af_input \
    --bind ${af_output_dir}:/root/af_output \
    --bind ${model_dir}:/root/models \
    --bind /home/shared/public_databases:/root/public_databases \
    /home/shared/alphafold3.sif \
    python /app/alphafold/run_alphafold.py \
    --json_path=/root/af_input/${input_json} \
    --model_dir=/root/models \
    --db_dir=/root/public_databases \
    --output_dir=/root/af_output
```
