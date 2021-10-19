#
# Makefile: Master Makefile for services and policies
#
# Application can be deployed as Standalone Detector Only or Complete with UI
#

# Checks required environment variables
-include env.check.mk

export ARCH ?= $(shell hzn architecture)

all: publish-inception publish-dle publish-controller publish-cme

publish-inception:
	make -C src/inception publish-service

# dle can be independently deployed. Standalobe detector only. Hence a deployment policy
publish-dle:
	make -C src/dle publish-service
	make -C src/dle deploy-policy

# controller can be independently deployed as well but for test purpose only. Hence a deployment policy
publish-controller:
	make -C src/controller publish-service

# cme deploys the complete vision-edge application. Hence a deployment policy
publish-cme:
	make -C src/cme publish-service
	make -C src/cme deploy-policy
