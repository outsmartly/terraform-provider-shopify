TEST?=$$(go list ./... |grep -v 'vendor')
GOFMT_FILES?=$$(find . -name '*.go' |grep -v vendor)
TARGETS=darwin linux windows
ARCHS=amd64 arm64 arm
WEBSITE_REPO=github.com/hashicorp/terraform-website
PKG_NAME=shopify
export GOPATH ?= $(HOME)/go

.PHONY: default
default: build

.PHONY: build
build: fmtcheck
	go build -o terraform-provider-shopify main.go

.PHONY: clean
clean:
	rm -rf dist

.PHONY: targets
targets:
	parallel --tagstring "[{1}-{2}]" '[[ "{1}" == "darwin" && "{2}" == "arm" ]] && echo "Skipping arm build for darwin" || \
	(GOOS={1} GOARCH={2} CGO_ENABLED=0 go build -o "dist/terraform-provider-shopify_$$(git describe --tags)_{1}_{2}" && \
	zip -j "dist/terraform-provider-shopify_$$(git describe --tags)_{1}_{2}.zip" "dist/terraform-provider-shopify_$$(git describe --tags)_{1}_{2}")' ::: $(TARGETS) ::: $(ARCHS)

.PHONY: release
release:
	$(MAKE) clean
	$(MAKE) targets
	@echo "Hashing binaries..."
	shasum -a 256 dist/*.zip > dist/terraform-provider-shopify_$$(git describe --tags)_SHA256SUMS
	@echo "Signing binaries..."
	(cd dist && gpg --local-user 1BE907FC25CA01E6 --detach-sign terraform-provider-shopify_$$(git describe --tags)_SHA256SUMS)
	@echo "Copying manifest..."
	cp terraform-registry-manifest.json dist/terraform-provider-shopify_$$(git describe --tags)_manifest.json
	gh release create $$(git describe --tags) dist/*.zip dist/terraform-provider-shopify_$$(git describe --tags)_SHA256SUMS.sig dist/terraform-provider-shopify_$$(git describe --tags)_SHA256SUMS dist/terraform-provider-shopify_$$(git describe --tags)_manifest.json --repo outsmartly/terraform-provider-shopify

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
