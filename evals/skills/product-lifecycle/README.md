# product-lifecycle eval

Tests that the agent can query the Red Hat Product Life Cycle API for product support status, EOL dates, and OCP version compatibility using the product-lifecycle skill.

## Prerequisites

### Cluster

A live OpenShift cluster is not required to run these evals. The test cases embed operator metadata in the query, and the agent queries the public [Red Hat Product Life Cycle API](https://access.redhat.com/product-life-cycles/api/v1/products) directly. Internet access is required.

The operator metadata in each test case was collected from a real OCP 4.21.5 cluster on GCP (6 nodes: 3 master + 3 worker) with these OLM operators installed:

| Operator | Package | Version | Channel |
|---|---|---|---|
| Red Hat OpenShift Logging | `cluster-logging` | 6.5.1 | stable-6.5 |
| Compliance Operator | `compliance-operator` | 1.9.0 | stable |
| Red Hat OpenShift Pipelines | `openshift-pipelines-operator-rh` | 1.22.0 | latest |
| Web Terminal | `web-terminal` | 1.16.0 | fast |
| DevWorkspace Operator | `devworkspace-operator` | 0.41.0 | fast |

## Ground truth

Expected values come from the live PLC API. Verify with:

```bash
# OCP 4.21 — should show "Full Support"
curl -s "https://access.redhat.com/product-life-cycles/api/v1/products?name=Red+Hat+OpenShift+Container+Platform" \
  | jq -r '.data[0].versions[] | select(.name == "4.21") | "\(.name) - \(.type)"'

# OCP 4.14 — should show "Extended Support"
curl -s "https://access.redhat.com/product-life-cycles/api/v1/products?name=Red+Hat+OpenShift+Container+Platform" \
  | jq -r '.data[0].versions[] | select(.name == "4.14") | "\(.name) - \(.type)"'

# cluster-logging — should show "Full Support", compatible with OCP 4.21
curl -s "https://access.redhat.com/product-life-cycles/api/v1/products?name=logging+for+Red+Hat+OpenShift" \
  | jq -r '.data[] | select(.package == "cluster-logging") |
    .name as $prod | .versions[] |
    "\($prod) \(.name) - \(.type) (OCP: \(.openshift_compatibility // "N/A"))"'

# compliance-operator — should show "Full Support" for v1.9
curl -s "https://access.redhat.com/product-life-cycles/api/v1/products?name=compliance+operator" \
  | jq -r '.data[] | .name as $prod | .versions[] |
    "\($prod) \(.name) - \(.type)"'

# Batch check — find operators by package name in the "OpenShift" product set
curl -s "https://access.redhat.com/product-life-cycles/api/v1/products?name=OpenShift" \
  | jq -r '.data[] | select(.is_operator) |
    "\(.package // "N/A"): \(.name)"'
```

## Running

```bash
bash evals/run.sh -k "product-lifecycle"
```
