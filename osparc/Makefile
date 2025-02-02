
.DEFAULT_GOAL := help


.PHONY: help, clean, build, push, build-and-push

MKFILE_PATH := $(abspath $(lastword $(MAKEFILE_LIST)))
REPO_ROOT := $(realpath $(dir $(MKFILE_PATH))/..)
CALL_DIR := $(CURDIR)
OSPARC_DIR := $(REPO_ROOT)/osparc
OPENAPI_GENERATOR_CLI := $(REPO_ROOT)/modules/openapi-generator-cli/target/openapi-generator-cli.jar
CONTAINER_REPO := /tmp/openapi-generator

BUILDKIT_IMAGE := maven:3.6.3-jdk-11-slim
OPENAPI_GENERATOR_IMAGE := itisfoundation/openapi-generator-cli
OPENAPI_GENERATOR_IMAGE_VERSION := v0 # manually increment this when switching to a new version


help: ## help on rule's targets
	@awk --posix 'BEGIN {FS = ":.*?## "} /^[[:alpha:][:space:]_-]+:.*?## / {printf "%-20s %s\n", $$1, $$2}' $(MAKEFILE_LIST)

clean: # clean up build
	docker run -v ${REPO_ROOT}:${CONTAINER_REPO} -w ${CONTAINER_REPO} --rm $(BUILDKIT_IMAGE) mvn clean

build: ## build a docker image containing openapi-generator-cli and push it to dockerhub
	docker run -v ${REPO_ROOT}:${CONTAINER_REPO} -w ${CONTAINER_REPO} --rm $(BUILDKIT_IMAGE) mvn clean install
	$(eval BRANCH := $(shell git rev-parse --abbrev-ref HEAD))
	@cp $(REPO_ROOT)/modules/openapi-generator-cli/target/openapi-generator-cli.jar $(OSPARC_DIR)/build-generator; \
	cd $(OSPARC_DIR)/build-generator; \
	docker build --build-arg buildkit=$(BUILDKIT_IMAGE) -t $(OPENAPI_GENERATOR_IMAGE)-$(BRANCH):$(OPENAPI_GENERATOR_IMAGE_VERSION) .
	@rm $(REPO_ROOT)/osparc/build-generator/openapi-generator-cli.jar
	$(MAKE) clean
	@echo "Done generating docker image. Run 'docker run $(OPENAPI_GENERATOR_IMAGE)-$(BRANCH):$(OPENAPI_GENERATOR_IMAGE_VERSION)'."

push: ## build a docker image containing openapi-generator-cli and push it to dockerhub
	$(eval BRANCH := $(shell git rev-parse --abbrev-ref HEAD))
	docker push $(OPENAPI_GENERATOR_IMAGE)-$(BRANCH):$(OPENAPI_GENERATOR_IMAGE_VERSION)

build-and-push: build push # build and push a docker image container openapi-generator-cli