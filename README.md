<h2 align="center">Starting Kit for Edge-Device LLM Competition, NeurIPS 2024</h2>

This is the starting kit for the Edge-Device LLM Competition, a NeurIPS 2024 competition. To learn more about the competition, please see the [competition website](https://edge-llms-challenge.github.io/edge-llm-challenge.github.io/).  This starting kit provides instructions on downloading data, running evaluations, and generating submissions.

### Submission Requirements

- Model definition file (.py) and its configuration file: **Must be huggingface format**. (https://huggingface.co/docs/transformers/en/custom_models)

- The saved weights: **Must be huggingface format**, i.e., saved via save_pretrained() function that inherated from transformers.PreTrainedModel(https://huggingface.co/docs/transformers/v4.41.3/en/main_classes/model#transformers.PreTrainedModel)

- The compiled model via MLC-MiniCPM tool. (https://github.com/OpenBMB/mlc-MiniCPM)

- The evaluated results including sores for CommonsenseQA, BIG-Bench Hard, GSM8K, LongBench, HumanEval, CHID, TruthfulQA, Throughput and Memory Usage.

### Evalution for CommonsenseQA, BIG-Bench Hard, GSM8K, LongBench, HumanEval, CHID, TruthfulQA Tasks

The evaluation for CommonsenseQA, BIG-Bench Hard, GSM8K, LongBench, HumanEval, CHID, and TruthfulQA tasks is conducted using the Opencompass tool.

**Environment Setup**

```bash
  conda create --name opencompass python=3.10 
  conda activate opencompass
  conda install pytorch torchvision torchaudio pytorch-cuda=12.1 -c pytorch -c nvidia
  Pip install Faiss-gpu
  cd opencompass && pip install -e .
  cd opencompass/human-eval && pip install -e .
```

**Data Preparation**

```bash
# Download dataset to data/ folder
wget https://github.com/open-compass/opencompass/releases/download/0.2.2.rc1/OpenCompassData-core-20240207.zip
unzip OpenCompassData-core-20240207.zip
```

**Evaluation Huggingface models**

```bash 
CUDA_VISIBLE_DEVICES=0 python run.py --datasets commonseqa_gen longbench bbh_gen gsm8k_gen humaneval_gen FewCLUE_chid_gen truthfulqa_gen --hf-num-gpus 1 --hf-type base --hf-path meta-llama/Meta-Llama-3-8B --debug --model-kwargs device_map='auto' trust_remote_code=True
## --dataset: specify datasets
```
**Evaluate local own models**

  - Our own model must be wraped to opencompass format. An example can be found in opencompass/opencompass/models/custom_llama.py Refer to (https://opencompass.readthedocs.io/en/latest/advanced_guides/new_model.html).
  - Prepare the corresponding configure file. An example can  be found in opencompass/configs/example/example.py 

```bash
CUDA_VISIBLE_DEVICES=0 python run.py --datasets commonseqa_gen longbench bbh_gen gsm8k_gen humaneval_gen FewCLUE_chid_gen truthfulqa_gen --hf-num-gpus 1 --hf-type base --models example --debug --model-kwargs device_map='auto' trust_remote_code=True
# --models: specify the local model
```

> \[!TIP\]
>
> -- The wrapped model file (.py) must be under folder: opencompass/opencompass/models.
>
> -- The prepared configure file must be under folder: /opencompass/configs



### Evalution for Memeory Usage+Throughput

```bash
# Replace the model/tokenizer loader code with your own code. DO NOT CHANGE THE HYPER-PARAMETER SETTING.
python EvaluateThroughputAndMemory.py --model_name MODEL_NAME
```

### Compile model via MLC-MiniCPM
Refer to (https://github.com/OpenBMB/mlc-MiniCPM)

**Prepare Enviroment**

Follow https://llm.mlc.ai/docs/deploy/android.html to prepare requirements.

For the **Compile PyTorch Models from HuggingFace**,  conduct the following instructions to install mlc_chat.

```bash
mkdir -p build && cd build
# generate build configuration
python3 ../cmake/gen_cmake_config.py && cd ..
# build `mlc_chat_cli`
cd build && cmake .. && cmake --build . --parallel $(nproc) && cd ..
# install
cd python && pip install -e . && cd ..
```

**Compile Model**

put huggingface downloaded model checkpoint into `dist/models`.

```bash
MODEL_NAME=MiniCPM
MODEL_TYPE=minicpm
mlc_chat convert_weight --model-type ${MODEL_TYPE} ./dist/models/${MODEL_NAME}-hf/  -o dist/$MODEL_NAME/
mlc_chat gen_config --model-type ${MODEL_TYPE} ./dist/models/${MODEL_NAME}-hf/ --conv-template LM --sliding-window-size 768 -o dist/${MODEL_NAME}/
mlc_chat compile --model-type ${MODEL_TYPE} dist/${MODEL_NAME}/mlc-chat-config.json --device android -o ./dist/libs/${MODEL_NAME}-android.tar
cd ./android/library
./prepare_libs.sh
cd -
```

### Submissions

- The shared link for the saved model in huggingface format and the compiled model via MLC-MiniCPM. (Make Sure that the saved model and compiled model can be downloaded via this shared link)

- The model definition file (.py) and its configur file. 

- The evaluated results containing scores of CommonsenseQA, BIG-Bench Hard, GSM8K, LongBench, HumanEval, CHID, TruthfulQA , Throughput and GPU memory usage. (.csv). Please generate CSV file via Generate_CSV.py


<span style="color:red">**Please join us on Discord for discussions and up-to-date announcements:**</span>

[https://discord.gg/SsyY2s2k](https://discord.gg/SsyY2s2k)
