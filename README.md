# Install SUSE Observability Agent

[![Action Status](https://img.shields.io/badge/action-ready-brightgreen.svg)]()

A GitHub Action that installs or upgrades the **SUSE Observability Agent** on any Kubernetes cluster using **Helm**. It ensures the namespace exists and installs or upgrades the chart with configurable options.

---

## 🚀 Features

* ✅ Automatic namespace creation
* ✅ Install or upgrade SUSE Observability Agent via Helm
* ✅ Secret-friendly API key handling
* ✅ Optional chart version pinning (use latest if omitted)
* ✅ Configurable TLS skip options

---

## 📥 Inputs

| Input name                | Required |                               Default                               | Description                                                                     |
| ------------------------- | :------: | :-----------------------------------------------------------------: | ------------------------------------------------------------------------------- |
| `stackstate_API_Key`      |    Yes   |                                  —                                  | StackState API key (mark as secret).                                            |
| `stackstateClusterName`   |    Yes   |                                  —                                  | Logical name of the observed Kubernetes cluster.                                |
| `stackstate_URL`          |    Yes   |                                  —                                  | SUSE Observability receiver endpoint URL (eg. `https://.../receiver/stsAgent`). |
| `Namespace`               |    No    |                         `suse-observability`                        | Namespace to install into.                                                      |
| `HelmRepo`                |    No    | `https://charts.rancher.com/server-charts/prime/suse-observability` | Helm repo URL (or repo name if already configured).                             |
| `ChartName`               |    No    |            `suse-observability/suse-observability-agent`            | Helm chart reference (repo/chart or chart if repo is added).                    |
| `ChartVersion`            |    No    |                               *empty*                               | Pin a specific chart version. If empty, the latest chart is used.               |
| `NodeagentskipTLSVerify`  |    No    |                                `true`                               | Set `nodeAgent.skipKubeletTLSVerify` (boolean as string: `true`/`false`).       |
| `GlobalskipSslValidation` |    No    |                               `false`                               | Set `global.skipSslValidation` (boolean as string: `true`/`false`).             |

---

## 🛠️ How it works

1. Ensures the target namespace exists (creates it if missing).
2. Adds the Helm repo (if provided) and updates local index.
3. Builds a `--version` argument only if `ChartVersion` is provided; otherwise installs the latest.
4. Runs `helm upgrade --install` with the supplied configuration.

---

## ✅ Example: workflow (.github/workflows/deploy-suse-agent.yml)

```yaml
name: Install SUSE Observability Agent

on:
  push:

jobs:
  deploy-suse-observability-agent:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Setup Kubeconfig
        uses: azure/k8s-set-context@v4
        with:
          method: kubeconfig
          kubeconfig: ${{ secrets.KUBECONFIG }}

      - name: Install SUSE Observability Agent
        uses: ./
        with:
          stackstate_API_Key: ${{ secrets.stackstate_API_Key }}
          stackstateClusterName: 'testing-amolk'
          stackstate_URL: 'https://stackstate.io/receiver/stsAgent'
          Namespace: 'suse-observability'
```

---

## ⚙️ Example: optional inputs

Install a specific chart version:

```yaml
with:
  ChartVersion: "1.4.0"
```

Install into a different namespace:

```yaml
with:
  Namespace: "monitoring"
```

Change TLS settings:

```yaml
with:
  NodeagentskipTLSVerify: "true"
  GlobalskipSslValidation: "true"
```

---

## Local testing tips

To manually replicate the action logic for debugging:

```bash
# Add and update helm repo
helm repo add suse-observability https://charts.rancher.com/server-charts/prime/suse-observability || true
helm repo update

# Example helm upgrade (without version, uses latest)
helm upgrade --install suse-observability-agent suse-observability/suse-observability-agent \
  --namespace "suse-observability" --create-namespace \
  --set-string 'stackstate.apiKey'="<YOUR_API_KEY>" \
  --set-string 'stackstate.cluster.name'="cluster_name" \
  --set-string 'stackstate.url'="https://example.stackstate.io/receiver/stsAgent" \
  --set "nodeAgent.skipKubeletTLSVerify"="true" \
  --set "global.skipSslValidation"="false"
```

---

## 🛟 Troubleshooting

* **`Error: flag needs an argument: --version`**

  * Cause: `--version` was appended without a value.
  * Fix: Provide `ChartVersion` input or leave it empty so the action omits `--version` and uses the latest.

* **`error: name must be specified`**

  * Cause: `Namespace` was empty when calling `kubectl`.
  * Fix: Provide `Namespace` input or rely on default `suse-observability`.

* **`repository not found`**

  * Cause: Wrong HelmRepo URL or chart name.
  * Fix: Verify `HelmRepo` and `ChartName` inputs. Try adding the repo locally and search for the chart.

* **`unauthorized` / `401`**

  * Cause: Invalid API key.
  * Fix: Verify `stackstate_API_Key` stored in GitHub Secrets.

---

## 🧑‍💻 Maintainer

**amolk** — contributions welcome. Open issues or PRs to improve the action.

---
