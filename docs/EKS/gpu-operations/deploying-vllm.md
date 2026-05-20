# Deploying the model using vLLM

!!! Note

    The following contents are the abbreviated version of the [Generative AI on Amazon EKS](https://catalog.workshops.aws/genai-on-eks/en-US) workshop.

---
## Why vLLM?

[vLLM](https://github.com/vllm-project/vllm) is one of several popular, open-source inference and serving engines specifically designed to optimize the performance of generative AI applications through more efficient GPU memory utilization.

### Key Benefits of vLLM

#### Superior Performance

- Up to 24x higher throughput compared to standard PyTorch implementations
- Continuous batching for optimal GPU utilization
- Optimized CUDA kernels for faster inference speeds

#### Efficient Memory Management with PagedAttention

- Reduces GPU memory usage by up to 60%
- Dynamic memory allocation for KV cache
- Enables larger batch sizes and longer sequences

#### Production-Ready Architecture

- OpenAI-compatible API server for easy integration
- Distributed inference with tensor parallelism
- Built-in streaming responses and request scheduling

While vLLM is not mandatory for running generative AI applications, these advantages make it a compelling choice for production deployments. Alternative inference engines like TensorRT-LLM can also be used for running generative AI models. 

---
## Model and Weights

For this session, we will use the [Ministral-3-8B-Instruct-2512 model](https://huggingface.co/mistralai/Ministral-3-8B-Instruct-2512).

First, let's set up the environment variables and explore the model files directly in S3:
``` bash
# Set up environment variables for your deployment
# AWS_REGION is already set in the workshop environment
# Get the account ID and construct the S3 bucket name
export AWS_ACCOUNT_ID=$(aws sts get-caller-identity --query Account --output text)
export S3_BUCKET_NAME="genai-models-${AWS_ACCOUNT_ID}"

echo "AWS Account ID: $AWS_ACCOUNT_ID"
echo "AWS Region: $AWS_REGION"
echo "S3 Bucket: $S3_BUCKET_NAME"

AWS Account ID: 080403789922
AWS Region: us-east-2
S3 Bucket: genai-models-080403789922
```

Now let's explore the model files stored in S3:
``` bash hl_lines="8 11 14"
# List the model files in the S3 bucket
aws s3 ls s3://${S3_BUCKET_NAME}/Ministral-3-8B-Instruct-2512/ --recursive

2026-05-18 18:33:14          0 Ministral-3-8B-Instruct-2512/
2026-05-18 18:55:23      20311 Ministral-3-8B-Instruct-2512/README.md
2026-05-18 18:55:23       2361 Ministral-3-8B-Instruct-2512/SYSTEM_PROMPT.txt
2026-05-18 18:55:23       1903 Ministral-3-8B-Instruct-2512/config.json
2026-05-18 18:55:24 10420633176 Ministral-3-8B-Instruct-2512/consolidated.safetensors  # (1)!
2026-05-18 18:55:23        131 Ministral-3-8B-Instruct-2512/generation_config.json
2026-05-18 18:55:23     103195 Ministral-3-8B-Instruct-2512/model.safetensors.index.json
2026-05-18 18:55:23       1185 Ministral-3-8B-Instruct-2512/params.json # (2)!
2026-05-18 18:55:23        976 Ministral-3-8B-Instruct-2512/processor_config.json
2026-05-18 18:55:23   16753777 Ministral-3-8B-Instruct-2512/tekken.json
2026-05-18 18:55:24   17077402 Ministral-3-8B-Instruct-2512/tokenizer.json # (3)!
2026-05-18 18:55:23      21168 Ministral-3-8B-Instruct-2512/tokenizer_config.json
```

1.  **consolidated.safetensors:**
        - This is the main model weights file using the SafeTensors format
        - SafeTensors is a safe and efficient format for storing tensors, designed as a safer alternative to Python's pickle format
        - It offers better security and faster loading times compared to traditional .bin or .pt (PyTorch) files
        - The 'consolidated' prefix indicates that all model weights are combined into a single file
2.  **params.json**
        - Contains the model's configuration parameters
        - Includes architectural details like number of layers, attention heads, and other hyperparameters
        - Essential for reconstructing the model's architecture during loading
3.  **tokenizer.json**
        - The tokenizer file that handles text preprocessing
        - Converts raw text into tokens that the model can understand
        - Contains the vocabulary and rules for text tokenization
        - The v3 suffix indicates the tokenizer version

---
## Deploying the Model

To deploy the model, we will use a vanilla Kubernetes service and deployment. This deployment leverages `Run:ai streamer` for efficient model loading directly from S3, eliminating the need for Persistent Volumes (PV) or Persistent Volume Claims (PVC).

### Benefits of `Run:ai` Streamer

- **Direct S3 Access**: Streams model weights directly from S3 without requiring local storage
- **Faster Startup**: Eliminates the need to copy entire models to local storage before starting inference
- **Cost Efficiency**: Reduces storage costs by avoiding duplicate model copies
- **Scalability**: Enables rapid scaling without storage provisioning delays
- **Concurrent Loading**: Supports concurrent model loading with configurable concurrency levels

Now let's download and customize the deployment manifest:
``` bash
mkdir -p manifests/200-inference

# Download the vLLM deployment manifest with Run:AI streamer
curl -o manifests/200-inference/vllm-s3-deployment.yml https://raw.githubusercontent.com/aws-samples/sample-genai-on-eks/refs/tags/v2.3.1/manifests/100-vllm/vllm-deployment.yml

# Use envsubst to replace the S3 bucket placeholder with your actual bucket name
envsubst < manifests/200-inference/vllm-s3-deployment.yml > /tmp/vllm-temp.yml && mv /tmp/vllm-temp.yml manifests/200-inference/vllm-s3-deployment.yml

# Verify the substitution worked
cat manifests/200-inference/vllm-s3-deployment.yml | grep -A 2 "model=s3://"

# Apply the deployment
kubectl apply -f manifests/200-inference/vllm-s3-deployment.yml
```

### Observing Auto Mode Node Scaling

Since this deployment requires GPU nodes, EKS Auto Mode will automatically provision a new GPU-enabled node if one doesn't already exist from the previous section. You can check if GPU nodes are available:

``` bash
# Check for existing GPU nodes
kubectl wait node --for=condition=Ready -l karpenter.sh/nodepool=gpu

node/i-0e2c27f8b63f89296 condition met
```

### Monitoring the Deployment

First, monitor the pod status until it's running:
``` bash
# Watch pod status until it transitions to Running
kubectl wait pods --for=jsonpath='{.status.phase}'=Running -l model=mistral --timeout=300s

pod/mistral-6cfb59d6cd-xbwgd condition met
```

Once the pod is running, you can check the logs to follow the model loading process:
``` bash
kubectl logs -l model=mistral -f

(APIServer pid=10) INFO 05-18 19:01:56 [utils.py:325] 
(APIServer pid=10) INFO 05-18 19:01:56 [utils.py:325]        █     █     █▄   ▄█
(APIServer pid=10) INFO 05-18 19:01:56 [utils.py:325]  ▄▄ ▄█ █     █     █ ▀▄▀ █  version 0.15.1
(APIServer pid=10) INFO 05-18 19:01:56 [utils.py:325]   █▄█▀ █     █     █     █  model   s3://genai-models-080403789922/Ministral-3-8B-Instruct-2512/
(APIServer pid=10) INFO 05-18 19:01:56 [utils.py:325]    ▀▀  ▀▀▀▀▀ ▀▀▀▀▀ ▀     ▀
(APIServer pid=10) INFO 05-18 19:01:56 [utils.py:325] 
(APIServer pid=10) INFO 05-18 19:01:56 [utils.py:261] non-default args: {'enable_auto_tool_choice': True, 'tool_call_parser': 'mistral', 'model': 's3://genai-models-080403789922/Ministral-3-8B-Instruct-2512/', 'trust_remote_code': True, 'max_model_len': 2048, 'enforce_eager': True, 'config_format': 'mistral', 'load_format': 'runai_streamer', 'model_loader_extra_config': {'concurrency': 16}, 'disable_custom_all_reduce': True, 'block_size': 16, 'swap_space': 16.0, 'max_num_batched_tokens': 8192, 'max_num_seqs': 256}
...
(APIServer pid=10) INFO:     Started server process [10]
(APIServer pid=10) INFO:     Waiting for application startup.
(APIServer pid=10) INFO:     Application startup complete.
(APIServer pid=10) INFO:     10.0.18.57:50880 - "GET /health HTTP/1.1" 200 OK
```



