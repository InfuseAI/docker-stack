VERSION := ngc-tensorflow2-22.10-py3
IMAGE_NAME_BASE=docker-stacks
IMAGE_BASE=infuseai

build:
	docker build . -t ${IMAGE_BASE}/${IMAGE_NAME_BASE}:${VERSION}

push:
	docker push ${IMAGE_BASE}/${IMAGE_NAME_BASE}:${VERSION}
