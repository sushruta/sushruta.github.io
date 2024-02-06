---
title: "Command Line Inference on Llama2 models"
description: My attempt to run inference on llama2 model on a GPU machine
slug: llama2-simple-inference
date: 2024-02-05T18:38:12-07:00
categories:
  - machine learning
  - llm
  - llama2
  - gpu
tags:
  - machine learning
  - llm
  - llama2
  - gpu
---

This post follows the instructions given in llama2 repository hosted by Meta. I have added some observations as I went about running these examples. This will also set the stage for later posts around LLama2 models on this blog.

## Downloading the Model

Go to [llama2 download center](https://llama.meta.com/llama-downloads/) to download the model. You may have to provide some details and a valid email where you'll receive the links for the models.

## Model Size on Disk and GPU

When downloaded to disk, the size of these models are the following -

|     Model Name     |    Model Size   |
| ------------------ | --------------- |
|  llama-2-7b        |   13 GB         |
|  llama-2-7b-chat   |   13 GB         |
|  llama-2-13b       |   25 GB         |
|  llama-2-13b-chat  |   25 GB         |
|  llama-2-70b       |  130 GB         |
|  llama-2-70b-chat  |  130 GB         |


These models weights are compressed and when loaded in raw form on GPU, they might take up some more space. For example, the smallest model - `7B` tokens - takes up more than `20 GB` GPU while performing inference. This gives a fair idea of what GPUs to use for our inference.

Nvidia A10 GPU has `24 GB` of VRAM which means this is enough for our `7b` model. On the other hand, we would require multiple A10 GPUs hosting the model in a sharded fashion (a.k.a tensor parallelism) for `13b` and `70b` models.

Nvidia A100 has a `40 GB` and an `80 GB` version. If we assume `80 GB` for our discussions, we will see that we can use one A100 for `7b` and `13b` models but need more than one A100 GPUs for hosting the `70b` model.

Also, we can see that these numbers are higher than the VRAM in consumer level GPUs like RTX 3080, 3090, 4080, 4090, etc. Therefore we will not be able to run these models on those cards. However, the quantized versions of these models can run on these GPUs when the quantized model size is much smaller.

Given all this information, let us use A10 GPU for performing inference using the `7b` model.

## GPU Instances Quota Increase

**Note:** We are requesting quota increase and not creating the instances. You will not be billed by following this step.

I will assume that you are using AWS. Similar things apply in Azure, GCP, OCI, etc. Initially, you will not be able to provision these machines because you don't have the quota. You will have to create a ticket to increase your quota. We are specifically interested in the `g5.__` instances. All of these instances have A10 GPUs and differ by the CPU and sytem RAM offered. To increase quota for `g5` class machines, go to 

```
Service Quotas > AWS services > Amazon Elastic Compute Cloud (Amazon EC2) > Running Dedicated g5 Hosts
```

Click on `Request Increase at Account Level`. Hit `2` and press submit. Wait for a few hours after which you should get the quota. 

## Instantiate the GPU instance

I use `g5.8xlarge` instance because I would like to have more than `100 GB` of system ram. I am billed about about `$2.50` per hour for this instance. Please check all the g5 instance types and choose the right one. You should be fine with either of `g5.4xlarge` or `g5.8xlarge`. Anything smaller would have less RAM and anything larger would be an overkill for this example.

You can start the launching part by going to -

```
EC2 > Instances > Launch Instances
```

Some properties that I usually choose are -

1. name of the host to be something descriptive like - `llama2-inference-host-1`
2. for AMI, I search for `pytorch` and chose the `pytorch 2.1` AMI with all the drivers and python installed
3. Instance type - I chose `g5.8xlarge`
4. For keypair, choose the right keypair. Please choose the keypair you have otherwise you'll not be able to SSH into the machine.
5. Leave the Network Settings as they are. They are not recommended for production settings! We will destroy our VM very soon and therefore we need not be worrying about this.
6. Make sure you ask for atleast 64 GB of storage. I usually do 72 GB.

That's it! Hit `Luanch` and wait for the instance to come up.

## Logging Through SSH

Just do 

```
ssh -i <YOUR_KEYPAIR_PVT_FILE.pem> ubuntu@<PUBLIC_IP_INSTANCE>
```

## Clone Llama2 Repository

Clone the llama2 repository

```bash
# clone the repo
git clone https://github.com/facebookresearch/llama.git
cd llama
# install all the relevant packages
pip install -e .
# download the model
./download.sh
```

When prompted for the link, paste the link you have received in your email. Paste the exact link that you got in the email. Please don't paste it in the browser and then get some redirected link and then paste it here.

Choose `7B` as the model to download.

## Running Inference

Let us run the simple text completion code given in the repository.

```bash
time torchrun --nproc_per_node 1 example_text_completion.py \
    --ckpt_dir llama-2-7b/ \
    --tokenizer_path tokenizer.model \
    --max_seq_len 128 \
    --max_batch_size 4
```

The model will load into GPU memory and then run the text completion on the inputs presented in the code file.

## Conclusion

That wraps our simple run through of computing inference on `llama-2-7b` model using nvidia A10 GPU. You can modify the code or run the chat completion code as well.
