VERSION := ngc-pytorch-22.11-py3
IMAGE_NAME_BASE=docker-stacks
IMAGE_BASE=infuseai

build:
	docker build . -t ${IMAGE_BASE}/${IMAGE_NAME_BASE}:${VERSION}

push:
	docker push ${IMAGE_BASE}/${IMAGE_NAME_BASE}:${VERSION}
