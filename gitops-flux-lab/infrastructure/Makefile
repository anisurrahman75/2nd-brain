BIN_DIR := ./.bin
KUSTOMIZE_VERSION := v5.4.1
KUBEVAL_VERSION := 0.16.1

KUSTOMIZE_BIN ?= $(BIN_DIR)/kustomize
KUBEVAL_BIN ?= $(BIN_DIR)/kubeval

ENV ?= staging

INFRA_OVERLAY_DIR := infrastructure/overlays/$(ENV)
APPS_OVERLAY_DIR := applications/overlays/$(ENV)

.PHONY: all validate build diff lint clean \
        validate-infra build-infra diff-infra lint-infra \
        validate-apps build-apps diff-apps lint-apps \
        install-tools install-kustomize install-kubeval ensure-bin-dir

## Default build and validate all
all: validate build

## 🧪 Validate all overlays
validate: validate-infra validate-apps

## 🔨 Build all overlays
build: build-infra build-apps

## 🔍 Diff all overlays with cluster
diff: diff-infra diff-apps

## 📋 Lint all overlays
lint: lint-infra lint-apps

## 🔥 Clean up temp artifacts
clean:
	@echo "🧹 Cleaning up..."
	rm -rf .kustomize-tmp

## ---------------- Infrastructure ----------------

validate-infra:
	@echo "🔍 Validating infrastructure for '$(ENV)'..."
	$(KUSTOMIZE_BIN) build $(INFRA_OVERLAY_DIR) >/dev/null
	@echo "✅ Infrastructure validation passed"

build-infra:
	@echo "🛠️ Building infrastructure manifest for '$(ENV)'..."
	$(KUSTOMIZE_BIN) build $(INFRA_OVERLAY_DIR)

diff-infra:
	@echo "🔎 Diffing infrastructure against cluster..."
	$(KUSTOMIZE_BIN) build $(INFRA_OVERLAY_DIR) | kubectl diff -f -

lint-infra:
	@echo "📋 Linting infrastructure YAMLs with kubeval..."
	$(KUSTOMIZE_BIN) build $(INFRA_OVERLAY_DIR) | $(KUBEVAL_BIN) --strict --ignore-missing-schemas


## ---------------- Applications ----------------

validate-apps:
	@echo "🔍 Validating applications for '$(ENV)'..."
	$(KUSTOMIZE_BIN) build $(APPS_OVERLAY_DIR) >/dev/null
	@echo "✅ Applications validation passed"

build-apps:
	@echo "🛠️ Building application manifest for '$(ENV)'..."
	$(KUSTOMIZE_BIN) build $(APPS_OVERLAY_DIR)

diff-apps:
	@echo "🔎 Diffing applications against cluster..."
	$(KUSTOMIZE_BIN) build $(APPS_OVERLAY_DIR) | kubectl diff -f -

lint-apps:
	@echo "📋 Linting application YAMLs with kubeval..."
	$(KUSTOMIZE_BIN) build $(APPS_OVERLAY_DIR) | $(KUBEVAL_BIN) --strict --ignore-missing-schemas

## ---------------- Tooling ----------------

install-tools: install-kustomize install-kubeval

ensure-bin-dir:
	@mkdir -p $(BIN_DIR)

install-kustomize: ensure-bin-dir
	@echo "⬇️  Downloading kustomize $(KUSTOMIZE_VERSION)..."
	curl -sSL https://github.com/kubernetes-sigs/kustomize/releases/download/kustomize%2F$(KUSTOMIZE_VERSION)/kustomize_$(KUSTOMIZE_VERSION)_linux_amd64.tar.gz \
		| tar -xz -C $(BIN_DIR)
	chmod +x $(KUSTOMIZE_BIN)

install-kubeval: ensure-bin-dir
	@echo "⬇️  Downloading kubeval $(KUBEVAL_VERSION)..."
	curl -sSL https://github.com/instrumenta/kubeval/releases/download/v$(KUBEVAL_VERSION)/kubeval-linux-amd64.tar.gz \
		| tar -xz -C $(BIN_DIR)
	chmod +x $(KUBEVAL_BIN)
