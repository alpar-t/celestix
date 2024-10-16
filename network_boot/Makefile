DOCKER_IMAGE := pxe-tftp

.PHONY: all build docker clean

all: build

build: docker

# Target to build the Docker image
docker:
	docker build -t $(DOCKER_IMAGE) .

run:
	docker run --rm -it $(DOCKER_IMAGE)

clean:
	docker rmi $(DOCKER_IMAGE)

