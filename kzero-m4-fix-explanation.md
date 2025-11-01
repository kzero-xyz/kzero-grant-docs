# Kzero M4 fix explanation

## 0a (Already Done)
| Number | Deliverable | Accepted | Link | Evaluation Notes |
| ------ | ----------- | -------- | ---- | ---------------- |
| 0a. | License | [x] | https://github.com/kzero-xyz/kzero-wallet | GPLv3 license is present at the repository root. |

## 0b 
| Number | Deliverable | Accepted | Link | Evaluation Notes |
| ------ | ----------- | -------- | ---- | ---------------- |
| 0b. | Documentation | [ ] | n/a | SDK packages only ship stub README files; no technical documentation or integration guide is provided as promised. |

Fix: Add `KZero Wallet SDK - Technical Documentation` https://github.com/kzero-xyz/kzero-grant-docs/blob/main/kzero-wallet-sdk.md


## 0c
| Number | Deliverable | Accepted | Link | Evaluation Notes |
| ------ | ----------- | -------- | ---- | ---------------- |
| 0c. | Testing and Testing Guide | [ ] | n/a | No automated tests are included and `pnpm test` fails because no projects declare a `test` task. No testing guide found. |

Fix: Add: `Testing Guide for KZero Wallet SDK`:  https://github.com/kzero-xyz/kzero-grant-docs/blob/main/kzero-wallet-test-guide.md


## 0d
| Number | Deliverable | Accepted | Link | Evaluation Notes |
| ------ | ----------- | -------- | ---- | ---------------- |
| 0d. | Docker | [ ] | n/a | No Dockerfile or container resources are present in the repository. |


Fix: You can follow the docker tutorial to build and run the docker locally.
- m4-docker-tutorial: [https://github.com/kzero-xyz/kzero-grant-docs/blob/main/m4-docker-tutorial.md](https://github.com/kzero-xyz/kzero-grant-docs/blob/main/m4-docker-tutorial.md)

The local Docker setup consists of two parts:
1. **Starting the pre-packaged kzero-service Docker**: Launch the ready-to-use Docker setup for kzero-service which includes PostgreSQL, auth-server, proof-server, and proof-worker. See [Step 1: Starting kzero-service] in the m4-docker-tutorial.
2. **Starting kzero-wallet**: Launch the wallet application and example application locally. See [Step 2: Starting kzero-wallet] in the m4-docker-tutorial.

## 0e
| Number | Deliverable | Accepted | Link | Evaluation Notes |
| ------ | ----------- | -------- | ---- | ---------------- |
| 0e. | Article | [ ] | n/a | No article or publication reference provided in the repository or documentation. |

Fix: The document shows the overall arch of Kzero: https://github.com/kzero-xyz/kzero-grant-docs/blob/main/kzero-article.md


## 1
| Number | Deliverable | Accepted | Link | Evaluation Notes |
| ------ | ----------- | -------- | ---- | ---------------- |
| 1. | KZero-SDK | [ ] | https://github.com/kzero-xyz/kzero-wallet | Packages build, but the example wallet renders a blank page locally even after running `pnpm install && pnpm dev:wallet`; SDK functionality cannot be verified in isolation. |

- Fix: follow the docker tutorial, at 0d, and run the command below open the kzero page at `localhost:5175`.

```bash
pnpm dev:example --port 5175
```

```bash
# Another terminal
pnpm dev:wallet --port 5176
```

And, then open the http://localhost:5175/

## 2
| Number | Deliverable | Accepted | Link | Evaluation Notes |
| ------ | ----------- | -------- | ---- | ---------------- |
| 2. | KZero-Website | [ ] | https://github.com/kzero-xyz/kzero-wallet/tree/master/example-wallet | `pnpm dev:wallet` launches Vite dev server but the app serves a blank screen, so the demo flow is not demonstrably functional. |

Fix: Run the docker above(at part `0d`), and open the http://localhost:5175/, you should be able to go through the whole process locally.

