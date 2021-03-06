.PHONY: configure cluster apply sshchainlink sshethereum info proxy
.PHONY: patch bootstrap-cluster secrets help

DEPLOYMENT ?= testnet
CLUSTER_NAME := $(DEPLOYMENT)
ROLLOUT_PATCH := '{"spec":{"template":{"spec":{"containers":[{"name":"CONTNAME","image":"CONTSHA"}]}}}}'
CHAINLINK_IMAGE ?= smartcontract/chainlink:latest
ETHEREUM_IMAGE ?= smartcontract/devnet:latest

configure: ## Configure google cloud credentials for PROJECT_ID (must be set)
	gcloud config set project $(PROJECT_ID)
	gcloud config set compute/zone us-central1-a
	gcloud auth login
	gcloud config list
	gcloud container clusters get-credentials $(CLUSTER_NAME)
	kubectl config current-context

cluster: ## Create a google cloud k8s cluster
	gcloud container clusters create $(CLUSTER_NAME) \
		--cluster-version 1.10

apply: ## Apply k8s configuration changes to cluster
	kubectl apply -f shared/ethereum-svc.yaml
	kubectl apply -f shared/ethereum-pv-claim.yaml
	kubectl apply -f $(DEPLOYMENT)/ethereum-deploy.yaml
	kubectl apply -f shared/chainlink-svc.yaml
	kubectl apply -f shared/chainlink-pv-claim.yaml
	kubectl apply -f $(DEPLOYMENT)/chainlink-deploy.yaml
	kubectl apply -f shared/ingress.yaml

sshchainlink: ## SSH into the chainlink container at POD env variable
	kubectl exec -it $(POD) -c chainlink -- /bin/sh

sshethereum: ## SSH into the ethereum container at POD env variable
	kubectl exec -it $(POD) -c ethereum -- /bin/bash

info: ## Use kubectl to display information about the running cluster
	kubectl config current-context
	kubectl cluster-info
	kubectl get secrets,pvc,pv,nodes,ingress,svc,deploy,po -o wide

proxy: ## Open up k8s dashboard by using kubectl to setup a proxy with gcloud
	open 'http://127.0.0.1:8001/ui'
	kubectl proxy

patch: ## Update the cluster's container SHAs, effectively forcing them to bump to the latest docker image
	-kubectl patch deployment ethereum-deploy -p \
		$(subst CONTSHA,$(shell ./docker-repodigest $(ETHEREUM_IMAGE)),$(subst CONTNAME,ethereum,$(ROLLOUT_PATCH)))
	-kubectl patch deployment chainlink-deploy -p \
		$(subst CONTSHA,$(shell ./docker-repodigest $(CHAINLINK_IMAGE)),$(subst CONTNAME,chainlink,$(ROLLOUT_PATCH)))

bootstrap-cluster: configure cluster secrets apply info ## One command to bootstrap a whole cluster from nothing. Make sure dotfiles are in place for secrets.

secrets: ## Update the cluster's secrets. Must have a populated secrets/ folder with ENVIRONMENT.* files. i.e. secrets/staging.env
	kubectl create secret generic cl-environment --from-env-file=secrets/$(DEPLOYMENT).env --dry-run=true -o yaml | kubectl apply -f -
	kubectl create secret generic cl-password --from-file=password=secrets/$(DEPLOYMENT).password --dry-run=true -o yaml | kubectl apply -f -
	kubectl create secret generic cl-api --from-file=api=secrets/$(DEPLOYMENT).api --dry-run=true -o yaml | kubectl apply -f -
	-kubectl delete secret cl-sslcert
	-kubectl create secret tls cl-sslcert --cert=secrets/$(DEPLOYMENT).crt --key=secrets/$(DEPLOYMENT).key

certificate-openssl: ## Generates a self signed certificate with openssl to tmp/
	./generate-openssl-cert

certificate-letsencrypt: ## Uses certbot to manually generate a certificate with Let's Encrypt at DOMAIN
	./generate-letsencrypt-cert

help:
	@echo ""
	@echo "Chainlink $(DEPLOYMENT) Cluster Commands"
	@echo ""
	@grep -E '^[a-zA-Z_-]+:.*?## .*$$' $(MAKEFILE_LIST) | sort | awk 'BEGIN {FS = ":.*?## "}; {printf "\033[36m%-30s\033[0m %s\n", $$1, $$2}'
