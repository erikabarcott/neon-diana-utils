# Install and Deploy ChatGPT to NeonAI Diana 

[ChatGPT](https://chat.openai.com/) is a large-language-model-based AI chatbot developed and maintained by OpenAI. This tutorial demonstrates using [NeonAI Diana](https://github.com/NeonGeckoCom/neon-diana-utils) to configure and deploy ChatGPT in NeonAI.

## Prerequisites

This tutorial uses a local installation of NeonAI Diana running on Microk8s, and a local installation of NeonCore running in Docker containers.

* A [ChatGPT API key](https://help.openai.com/en/articles/4936850-where-do-i-find-my-secret-api-key).
* [Diana 1.0](https://github.com/NeonGeckoCom/neon-diana-utils/blob/dev/README.md). 
    * Follow the instructions in the [NeonAI Diana README](https://github.com/NeonGeckoCom/neon-diana-utils/blob/dev/README.md) to set up Diana with Microk8s.
    * Use the subnet `10.10.10.10-10.10.10.10`.
    * Add `10.10.10.10 chatgpt.example.com` to the `/etc/hosts` file.
* [NeonCore](https://github.com/NeonGeckoCom/NeonCore) 27 or later.
    * Enable a communication method for NeonCore. For example, set up audio/voice communication, or install the [Neon CLI Client](https://pypi.org/project/neon-cli-client/).
    * Follow the instructions in the [NeonCore README](https://github.com/NeonGeckoCom/NeonCore) to launch NeonCore in Docker containers.
    * Add an MQ server to `DOCKER_DIR/xdg/config/neon/neon.yaml` with a configuration block like:
    ```
    MQ:
      server: mq.2021.us
      port: 5672
    ```
    
    
## 1. Use a Python virtual environment

We recommend you use a [Python virtual environment](https://docs.python.org/3/library/venv.html) for installing and testing Diana. In addition to the usual benefits, using a virtual environment makes it easier to ensure you are using the correct versions of Python and Diana.

a. Create a Python virtual environment:

```
python3.10 -m venv venv
```

b. Open a new terminal window. Activate the Python virtual environment in this new window:

```
. venv/bin/activate
```

Using this new window, proceed with the instructions.

## 2. Deploy NeonAI Diana

a. Start Microk8s.

```
microk8s start
```

b. Configure the Diana backend.

```
diana configure-mq-backend OUTPUT_PATH
```

Replace `OUTPUT_PATH` with the directory where you want Diana to store its Helm charts and configurations. For example:

```
diana configure-mq-backend ~/neon_diana
```

c. Diana prompts you to enable or disable various services. Answer `y` or `n` as appropriate. 

    1. Answer `y` to the prompt, "Configure ChatGPT Service?"
    2. Paste in your ChatGPT API Key at the prompt.
    3. Select the ChatGPT Model you wish to use. To use the default version, hit `Enter`.
    4. Enter your ChatGPT Role. This should be a short description of your account, like "Admin" or "Researcher."
    5. Select your ChatGPT Context Depth. The default level is 3.
    6. Select the maximum tokens in responses. The default is 100.
    7. Verify the ChatGPT configuration is correct. Answer `y` to continue, or `n` to go back and change your responses.

d. Deploy the NGINX ingress.

```
helm install ingress-nginx OUTPUT_PATH/ingress-common --namespace ingress-nginx --create-namespace
```

e. Edit `OUTPUT_PATH/diana-backend/values.yaml`. Change `domain` to `example.com`.

```
# This Chart will deploy Diana backend services, including RabbitMQ,
# HTTP Services with associated ingress and certificates, MQ Services,
# and configuration including secrets with keys and passwords.

# TODO: Update `domain`
domain: &domain example.com
```

## 3. Deploy the Diana Backend

a. Update the Helm dependency:

```
helm dependency update OUTPUT_PATH/diana-backend
```

b. Use Helm to launch Diana:

```
helm install diana-backend OUTPUT_PATH/diana-backend --namespace backend --create-namespace
```

This creates the `backend` namespace and launches Diana into that namespace. You can change this to any namespace name you prefer. You may want to use separate namespaces for test versus production deployments, to separate the Diana backend from other deployments, or both.

## 4. Launch NeonCore

a. Go to the directory where you installed NeonCore.

```
cd DOCKER_DIR/docker
```

b. Launch NeonCore.

```
docker-compose up -d
```

c. Connect to NeonCore, either by using a wake word or by launching the Neon CLI.

## 5. Access ChatGPT in NeonAI

a. Use `Talk to ChatGPT` to start using ChatGPT in NeonAI.

IMAGE: SCREENSHOT



