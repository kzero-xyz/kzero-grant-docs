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
- m4-docker-tutorial:
https://github.com/kzero-xyz/kzero-grant-docs/blob/main/m4-docker-tutorial.md

The whole process includes 6 components working together:
- POSTGRES_DB - PostgreSQL database for storing authentication and user data
- auth-server - OAuth2 authentication server with zkLogin support (from kzero-service)
- proof-server - WebSocket proof generation task server (from kzero-service)
- proof-worker - Zero-knowledge proof generation worker (from kzero-service)
- wallet - KZero wallet application (from kzero-wallet)
- example - Example application demonstrating KZero integration (from kzero-wallet)

> The whole process docker tutorial includes the following docker instruction related to kzero-service(at the [Part 3. Step 1: Setting Up kzero-service](https://github.com/kzero-xyz/kzero-grant-docs/blob/main/m4-docker-tutorial.md#3-step-1-setting-up-kzero-service) in the m4-docker-tutorial)
> - For running kzero-service(running the first 4 components):
> https://github.com/kzero-xyz/kzero-service/blob/feature/auth-server/DOCKER.md

## 0e
| Number | Deliverable | Accepted | Link | Evaluation Notes |
| ------ | ----------- | -------- | ---- | ---------------- |
| 0e. | Article | [ ] | n/a | No article or publication reference provided in the repository or documentation. |

Fix: The document shows the overall arch of Kzero: https://github.com/kzero-xyz/kzero-grant-docs/blob/main/kzero-article.md


## 1
| Number | Deliverable | Accepted | Link | Evaluation Notes |
| ------ | ----------- | -------- | ---- | ---------------- |
| 1. | KZero-SDK | [ ] | https://github.com/kzero-xyz/kzero-wallet | Packages build, but the example wallet renders a blank page locally even after running `pnpm install && pnpm dev:wallet`; SDK functionality cannot be verified in isolation. |

- Fix: follow the docker tutorial, at 0d, and run the command below open the kzero page.

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

