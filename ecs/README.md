# ECS DEPRECATED

We now use Kubernetes. This information is out of date.

## Deploying Testnet

We use AWS ECS to easily bootstrap and deploy a cluster of docker containers
running Parity against Ropsten with an attached Chainlink node.

#### Requirements:

1. Valid AWS credentials with access id and secret key.
  - Set as ENV vars to `AWS_ACCESS_KEY_ID` and `AWS_SECRET_ACCESS_KEY`
2. [`aws` CLI tool](https://docs.aws.amazon.com/cli/latest/userguide/cli-install-macos.html) installed and configured.
3. [`ecs-cli` CLI tool](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/ECS_CLI_installation.html) installed.

#### Steps:

```
brew install awscli amazon-ecs-cli

# Configure awscli and set AWS_ACCESS_KEY_ID and AWS_SECRET_ACCESS_KEY

cd ecs
./configure-ecs-cli
./testnet-cluster-up
# After completion wait for a minute for resources to be available

ecs-cli compose up
ecs-cli compose ps
ecs-cli logs --task-id 101754fe-2e3f-49a3-9e90-c97d14d505c2 # Replace with your task id
```

#### Splitting logs by container

Chainlink

```
ecs-cli logs --task-id 101754fe-2e3f-49a3-9e90-c97d14d505c2 --container-name chainlink
```

Parity

```
ecs-cli logs --task-id 101754fe-2e3f-49a3-9e90-c97d14d505c2 --container-name parity
```
