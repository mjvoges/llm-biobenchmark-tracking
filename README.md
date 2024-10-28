# An evaluation of general purpose LLMs on biological benchmarks to extract trendlines.

## Description

LLM performance on traditional benchmarks is [saturating](https://contextual.ai/news/plotting-progress-in-ai/). We wanted to find out to what degree this trend holds true for benchmarks in biology. To this end, we performed a sprint-like evaluation of SOA LLMs against an "easy" benchmark ([GPQA](https://arxiv.org/abs/2311.12022)'s "biology" subset) and one of the more recently published, challenging benchmarks ([ProtocolQA](https://arxiv.org/pdf/2407.10362) from FutureHouse). 

Evaluations were performed using API's for models hosted through [OpenRouter](https://openrouter.ai/) or models hosted through [HuggingFace Inference Endpoints.](https://huggingface.co/inference-endpoints/dedicated)

## Approach

- Follow the provided instructions to `git clone` and `pip install -e .` FutureHouse's [LAB-bench](https://github.com/Future-House/LAB-Bench) and EleutherAI's [lm-evaluation-harness](https://github.com/EleutherAI/lm-evaluation-harness).

- Gather your API's URL and API key. Set these to be environment variables. As an example for a model on OpenRouter, use `export BASE_URL=https://openrouter.ai/api/v1` and `export OPENROUTER_API_KEY=####` for LAB bench and `export BASE_URL=https://openrouter.ai/api/v1/chat/completions` for lm-evaluation-harness.

- Some small changes to the existing code to call the relevant APIs. This assumes that your API is compatible with the OpenAI API "standard".
  
  - **ProtocolQA evaluation with LAB-bench**:
    - Navigate to labbench/openai.py and replace `self.client = openai.AsyncOpenAI()` with `self.client = openai.AsyncOpenAI(base_url=os.environ.get("BASE_URL"), api_key=os.environ.get("OPENROUTER_API_KEY")`.
    - Example use: to run an eval with Llama 3.2, call the evalution using
      ```
      ./score_baseline.py --eval ProtocolQA --provider openai --model meta-llama/llama-3.2-3b-instruct --n_threads 8 --output path/to/outputs/ProtocolQA-meta-llama/llama-3.2-3b-instruct.json
      ```
      
  - **GPQA evaluation with lm-evaluation-harness**:
    - Navigate to lm_eval/models/openai_completions.py. Under `_create_payload` of the `LocalChatCompletion` class change the format of the request from `"messages": messages` to `"messages": [{"role": "user", "content":messages}]`.
    - Under the `OpenAIChatCompletion` class, replace with `base_url=os.environ.get("BASE_URL")`, and under `api_key`, replace with `key = os.environ.get("OPENROUTER_API_KEY", None)`
    - To evaluate only the biology-related questions from the GPQA benchmark, navigate to lm_eval/tasks/gpqa/cot_zeroshot/utils.py and replace `return dataset.map(_process_doc)` with `return dataset.map(_process_doc).filter(lambda example: example["Subdomain"] in ["Molecular Biology", "Genetics"])`
    - Example use: to run an eval with Llama 3.2, call
      ```
      lm_eval --model openai-chat-completions --tasks gpqa_main_cot_zeroshot --model_args model=meta-llama/llama-3.2-3b-instruct --output_path path/to/outputs/
      ```

Happy evaluating!
