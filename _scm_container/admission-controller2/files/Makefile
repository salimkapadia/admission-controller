GOFILES=$(shell find . -type f -name '*.go' -not -path "./vendor/*")
GOPACKAGES=$(shell go list ./... | grep -v /vendor/ | grep -v test/ | grep -v pkg/apis/)

dep:
	@go get -u github.com/golang/dep/cmd/dep

build-deps: dep
	@dep ensure -v -vendor-only

test-deps: build-deps
	@go get github.com/stretchr/testify/assert
	@go get golang.org/x/lint/golint
	@go get github.com/pierrre/gotestcover
	@go get github.com/onsi/ginkgo/ginkgo
	@go get github.com/onsi/gomega/...

push:
	docker build -t $(HUB)/admission-controller2:$(TAG) .
	docker push $(HUB)/admission-controller2:$(TAG)

fmt:
	@if [ -n "$$(gofmt -l ${GOFILES})" ]; then echo 'Please run gofmt -l -w on your code.' && exit 1; fi

lint:
	@set -e; for LINE in ${GOPACKAGES}; do golint -set_exit_status=true $${LINE} ; done

vet:
	@set -e; for LINE in ${GOPACKAGES}; do go vet $${LINE} ; done

helm.install.local: push
	-rm $$(pwd)/admission-controller2-2.0.0.tgz
	helm package helm/admission-controller2
	helm install -n admission-controller2 $$(pwd)/admission-controller2-2.0.0.tgz --set image.host=$(HUB) --set image.tag=$(TAG)

helm.install:
	-rm $$(pwd)/admission-controller2-2.0.0.tgz
	helm package helm/admission-controller2
	helm install -n admission-controller2 $$(pwd)/admission-controller2-2.0.0.tgz

helm.clean:
	-helm/cleanup.sh admission-controller2
