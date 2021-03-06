## Deploying Testnet to GCP

We use Google Cloud Platform's Kubernetes Engine (GKE) to easily bootstrap
and deploy Parity against Ropsten with an attached Chainlink node.

#### Requirements:

1. Valid GCP account.
2. Create project chainlink-testnet on GCP and enable the kubernetes engine by visiting [the web portal](https://console.cloud.google.com/kubernetes/list).
3. [`gcloud` CLI tool](https://github.com/Homebrew/homebrew-core/issues/583#issuecomment-214024802) installed.
4. [`kubectl` CLI tool](https://kubernetes.io/docs/tasks/tools/install-kubectl/#install-with-homebrew-on-macos) installed.

#### Steps:

```
brew tap caskroom/cask
brew cask install google-cloud-sdk
brew install kubectl

# Create from scratch
PROJECT_ID=your-project-name make bootstrap-cluster

# which is the same as
PROJECT_ID=your-project-name make configure cluster testnet info

# Redeploy after updating images
make patch
```

#### Logs

You can see logs of containers by finding the corresponding pod
id with `make info` and using the commands below:

```
kubectl logs po/testnet-deploy-84c6b4686c-kkp5x chainlink
kubectl logs po/testnet-deploy-84c6b4686c-kkp5x ethereum
```

#### Copying Files
```
kubectl cp $POD_ID:data/chainlink/logs.jsonl ./your/local/directory
```

#### Restarting a Deployment
```
kubectl scale --replicas=0 deployment.extensions/ethereum-deploy
kubectl scale --replicas=1 deployment.extensions/ethereum-deploy
```
