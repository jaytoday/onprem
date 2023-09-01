# OnPrem

<!-- WARNING: THIS FILE WAS AUTOGENERATED! DO NOT EDIT! -->

> A tool for running large language models on-premises using non-public
> data

**OnPrem** is a simple Python package that makes it easier to run large
language models (LLMs) on non-public or sensitive data and on machines
with no internet connectivity (e.g., behind corporate firewalls).
Inspired by the [privateGPT](https://github.com/imartinez/privateGPT)
GitHub repo and Simon Willison’s [LLM](https://pypi.org/project/llm/)
command-line utility, **OnPrem** is designed to help integrate local
LLMs into practical applications.

## Install

Once [installing PyTorch](https://pytorch.org/get-started/locally/), you
can install **OnPrem** with:

``` sh
pip install onprem
```

For GPU support, see additional instructions below.

## How to use

### Setup

``` python
import os.path
from onprem import LLM

url = 'https://huggingface.co/TheBloke/Wizard-Vicuna-7B-Uncensored-GGML/resolve/main/Wizard-Vicuna-7B-Uncensored.ggmlv3.q4_0.bin'

llm = LLM(model_name=os.path.basename(url))
llm.download_model(url, ssl_verify=True ) # set to False if corporate firewall gives you problems
```

    There is already a file Wizard-Vicuna-7B-Uncensored.ggmlv3.q4_0.bin in /home/amaiya/onprem_data.
     Do you want to still download it? (Y/n) Y
    [██████████████████████████████████████████████████]

### Send Prompts to the LLM to Solve Problems

This is an example of few-shot prompting, where we provide an example of
what we want the LLM to do.

``` python
prompt = """Extract the names of people in the supplied sentences. Here is an example:
Sentence: James Gandolfini and Paul Newman were great actors.
People:
James Gandolfini, Paul Newman
Sentence:
I like Cillian Murphy's acting. Florence Pugh is great, too.
People:"""

saved_output = llm.prompt(prompt)
```


    Cillian Murphy, Florence Pugh

### Talk to Your Documents

Answers are generated from the content of your documents.

#### Step 1: Ingest the Documents into a Vector Database

``` python
llm.ingest('./sample_data')
```

    Creating new vectorstore
    Loading documents from ./sample_data
    Loaded 11 new documents from ./sample_data
    Split into 62 chunks of text (max. 500 tokens each)
    Creating embeddings. May take some minutes...
    Ingestion complete! You can now query your documents using the prompt method

    Loading new documents: 100%|██████████████████████| 2/2 [00:00<00:00, 11.58it/s]

#### Step 3: Answer Questions About the Documents

``` python
question = """Please answer the following question in a single sentence using only the provided context: What is  ktrain?""" 
answer, docs = llm.ask(question)
print('\n\nReferences:\n\n')
for i, document in enumerate(docs):
    print(f"\n{i+1}.> " + document.metadata["source"] + ":")
    print(document.page_content)
```

     K-Train is an automation tool that augments and complements human engineers during machine learning workow.

    References:



    1.> ./sample_data/ktrain_paper.pdf:
    lection (He et al., 2019). By contrast, ktrain places less emphasis on this aspect of au-
    tomation and instead focuses on either partially or fully automating other aspects of the
    machine learning (ML) workﬂow. For these reasons, ktrain is less of a traditional Au-
    2

    2.> ./sample_data/ktrain_paper.pdf:
    possible, ktrain automates (either algorithmically or through setting well-performing de-
    faults), but also allows users to make choices that best ﬁt their unique application require-
    ments. In this way, ktrain uses automation to augment and complement human engineers
    rather than attempting to entirely replace them. In doing so, the strengths of both are
    better exploited. Following inspiration from a blog post1 by Rachel Thomas of fast.ai

    3.> ./sample_data/ktrain_paper.pdf:
    tifying examples that the model is getting the most wrong, and Explainable AI methods to
    understand why mistakes were made.
    3) Model-Application.
    Both the model and the potentially complex set of steps re-
    quired to preprocess raw data into the format expected by the model must be easily saved,
    transferred to, and executed on new data in a production environment.
    ktrain is a Python library for machine learning with the goal of presenting a simple,

    4.> ./sample_data/ktrain_paper.pdf:
    this may involve language-speciﬁc preprocessing (e.g., tokenization). In the case of images,
    this may involve auto-normalizing pixel values in a way that a chosen model expects. In
    the case of graphs, this may involve compiling attributes of nodes and links in the network
    (Data61, 2018). All preprocessing methods in ktrain return a Preprocessor instance
    that encapsulates all the preprocessing steps for a particular task, which can be employed

### Speeding Up Inference Using a GPU

The above example employed the use of a CPU.  
If you have a GPU (even an older one with less VRAM), you can speed up
responses.

#### Step 1: Install `llama-cpp-python` with CUDABLAS support

``` shell
CMAKE_ARGS="-DLLAMA_CUBLAS=on" FORCE_CMAKE=1 pip install --upgrade --force-reinstall llama-cpp-python==0.1.69 --no-cache-dir
```

It is important to use the specific version shown above due to library
incompatibilities.

#### Step 2: Use the `n_gpu_layers` argument with [`LLM`](https://amaiya.github.io/onprem/core.html#llm)

llm = LLM(model_name=os.path.basename(url), n_gpu_layers=128)

With the steps above, calls to methods like `llm.prompt` will offload
computation to your GPU and speed up responses from the LLM.
