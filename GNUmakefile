TEST?=$$(go list ./... |grep -v 'vendor')
GOFMT_FILES?=$$(find . -name '*.go' |grep -v vendor)
TARGETS=darwin linux windows
WEBSITE_REPO=github.com/hashicorp/terraform-website
PKG_NAME=shopify
export GOPATH ?= $(HOME)/go

.PHONY: default
default: build

.PHONY: build
build: fmtcheck
	go build -o terraform-provider-shopify main.go

.PHONY: targets
targets: $(TARGETS)

.PHONY: $(TARGETS)
$(TARGETS):
	GOOS=$@ GOARCH=amd64 CGO_ENABLED=0 go build -o "dist/terraform-provider-shopify_$$(git describe --tags)_$@_amd64"
	zip -j dist/terraform-provider-shopify_$$(git describe --tags)_$@_amd64.zip dist/terraform-provider-shopify_$$(git describe --tags)_$@_amd64

.PHONY: test
test: fmtcheck
	go test -i $(TEST) || exit 1
	echo $(TEST) | \
		xargs -t -n4 go test $(TESTARGS) -timeout=30s -parallel=4

.PHONY: testacc
testacc: fmtcheck
	TF_ACC=1 go test $(TEST) -v $(TESTARGS) -timeout 120m

.PHONY: vet
vet:
	@echo "go vet ."
	@go vet $$(go list ./... | grep -v vendor/) ; if [ $$? -eq 1 ]; then \
		echo ""; \
		echo "Vet found suspicious constructs. Please check the reported constructs"; \
		echo "and fix them if necessary before submitting the code for review."; \
		exit 1; \
	fi

.PHONY: fmt
fmt:
	gofmt -w $(GOFMT_FILES)

.PHONY: fmtcheck
fmtcheck:
	@sh -c "'$(CURDIR)/scripts/gofmtcheck.sh'"

docs:
	tfplugindocs generate

.PHONY: vendor-status
vendor-status:
	@govendor status

.PHONY: test-compile
test-compile:
	@if [ "$(TEST)" = "./..." ]; then \
		echo "ERROR: Set TEST to a specific package. For example,"; \
		echo "  make test-compile TEST=./$(PKG_NAME)"; \
		exit 1; \
	fi
	go test -c $(TEST) $(TESTARGS)
