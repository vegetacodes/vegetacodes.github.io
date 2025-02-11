---
title: "Running DeepSeek-R1 Locally with Ollama"
date: 2025-02-10T11:15:02-04:00
draft: false
---

## Introduction
DeepSeek-R1 is an advanced AI model that can be run locally for various natural language processing tasks. DeepSeek has suspended its usage top-up feature recently, and I had to find a way to use it locally in order to work on a side project. I have documented the steps, which resulted in this blog post.

![image](/deepseek-topup-error.png#center)

In this post, we'll walk through the process of setting up and running DeepSeek-R1 using Ollama, a tool designed for easily managing AI models locally. We will also test the setup using a `curl` command to verify its functionality.

## Prerequisites
Before proceeding, ensure you have the following:

- A system with sufficient computing power (GPU recommended but not mandatory)
  - For reference, I ran this on my good old MacBook Pro 2017 with a 2.3 GHz Dual-Core Intel Core i5 and 8GB of RAM. Not the fastest machine, but it got the job done! 😂
- Docker installed (Optional, if using Ollama with containerized deployment)
- Ollama installed ([Download here](https://ollama.com))

## Step 1: Install Ollama
First, download and install Ollama from its official website:

```sh
curl -fsSL https://ollama.com/install.sh | sh
```

Verify the installation by running:

```sh
ollama --version
```

## Step 2: Run the DeepSeek-R1 Model
Once Ollama is installed, fetch and run the DeepSeek-R1 model using(I used the 1.5B parameter model which was ~1.1Gb):

```sh
ollama run deepseek-r1:1.5b
```

This command downloads the necessary files, initializes the model and starts listening for API requests.

## Step 4: Test with a cURL Request
To ensure everything is working correctly, test the model with a `curl` request:

```sh
❯ curl http://localhost:11434/api/chat -d '{"model": "deepseek-r1:1.5b", "messages":[{"role": "user", "content": "Solve: 25x25"}], "stream": false }'
```

If the setup is correct, the response should include the answer:

```json
 {
   "model":"deepseek-r1:1.5b",
   "created_at":"2025-02-11T19:46:50.991731Z",
   "message":{
      "role":"assistant",
      "content":"\u003cthink\u003e\nFirst, I identify the two numbers to be multiplied: 25 and 25.\n\nNext, I multiply each pair of digits from right to left. Starting with the units place: 5 multiplied by 5 equals 25. I write down the 5 in the units place and carry over the 2.\n\nThen, I move to the tens place: 5 multiplied by 5 equals 25 again, plus the carried-over 2 makes 27. I write down the 7 in the tens place and carry over the 2.\n\nSince both numbers have their digits multiplied entirely, I add any remaining carries to get the final product of 625.\n\u003c/think\u003e\n\nTo solve \\( 25 \\times 25 \\), follow these steps:\n\n1. **Multiply the units digit:**\n   \\[\n   5 \\times 5 = 25\n   \\]\n   Write down **5** and carry over **2**.\n\n2. **Multiply the tens digit:**\n   \\[\n   5 \\times 5 = 25\n   \\]\n   Add the carried-over **2**: \n   \\[\n   25 + 2 = 27\n   \\]\n   Write down **7** and carry over **2**.\n\n3. **Add any remaining carries:**\n   Since there are no more digits to multiply, add the carried-over **2**:\n   \\[\n   2\n   \\]\n\n4. **Combine all results:**\n   \\[\n   625\n   \\]\n\n\\(\\boxed{625}\\)"
   },
   "done_reason":"stop",
   "done":true,
   "total_duration":21819647737,
   "load_duration":30788930,
   "prompt_eval_count":12,
   "prompt_eval_duration":281000000,
   "eval_count":335,
   "eval_duration":21506000000
}

```

## Conclusion
By following these steps, you have successfully set up and tested the DeepSeek-R1 model locally using Ollama. This setup allows you to use AI models without relying on external APIs, offering improved privacy and control over your workflows.

Happy coding! 🧑‍💻👩‍💻🚀

