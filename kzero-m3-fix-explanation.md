# Kzero M3 Fix Explanation
 
## 0a
| Number | Deliverable | Accepted | Link | Evaluation Notes |
| ------ | ----------- | -------- | ---- | ---------------- |
| 0a. | License |<ul><li>[ ]</li></ul>| https://github.com/kzero-xyz/kzero-salt-enclave-service | GPLv3 file missing from the repository; please add the license as committed in the application. |
- Fix: Add new license file for GPLv3.(https://github.com/kzero-xyz/kzero-salt-enclave-service/blob/enclave-sim-mode-docker/LICENSE_GPLv3.md)


## 0b
| Number | Deliverable | Accepted | Link | Evaluation Notes |
| ------ | ----------- | -------- | ---- | ---------------- |
| 0b. | Documentation |<ul><li>[ ]</li></ul>| https://github.com/kzero-xyz/kzero-salt-enclave-service#readme | README covers Docker usage but does not document salt derivation flow or explain how to keep JWT/JWKS fixtures valid; the linked tutorial is not referenced from the codebase. |

![salt_service_workflow](https://hackmd.io/_uploads/rksMNRNTgx.png)
The salt server plays an important part in maintaining privacy and security for users' Web2 credentials when using Kzero. Using a secret master seed and the user's JWT, the salt server produces a salt value that is unique to that user for that app, but hides the connection from the user's identity to their Polkadot activity, cryptographically ensuring privacy. The salt value is required before generating a zkLogin proof and therefore before issuing transactions onchain.

When someone uses an app backed by the Kzero salt server, they enter their Web2 credentials and the application requests a JWT from the auth provider. The app then sends the JWT to the salt server to get the salt value. Each time the Polkadot address is derived from the user's identity, the salt is used to ensure that the user's address can always deterministically be computed from their token without revealing the binding between the two.
![salt_derivation](https://hackmd.io/_uploads/rJiSVRETeg.png)
This service implements a salt generation mechanism that keeps a master seed value and derives a user salt with key derivation by validating and parsing the JWT. For example, using `HKDF(ikm = seed, salt = iss || aud, info = sub)`(To know more about HKDF, please refer to the link [here](https://datatracker.ietf.org/doc/html/rfc5869)).
> For more details about the salt server, please check [here](https://github.com/kzero-xyz/kzero-grant-docs/blob/main/kzero-salt-service-spec.md)

> - JWT: Json Web Token (which is issued by OAuth2 Provider, like google. And held by user)
> - JWK: Json Web Key (which is the KeyInfo held by Oauth2 Provider, like google, to sign each JWT)

- Fix: Update the README(https://github.com/kzero-xyz/kzero-salt-enclave-service?tab=readme-ov-file#salt-generation-principles)
    - Add: salt derivation flow, and necessary description.
    - Add: necessary comment about the valid test JWK/JWT. Notice that, (using google for example) the salt service will alwary fetch the current JWT form the `https://www.googleapis.com/oauth2/v3/certs` as in the code. So, each time when trying to curl the `get_salt` api, we need to generate a real JWT, otherwise, if used the old JWT (for example, 1 week ago), it would mismatch the current JWK online. And return the `No matching key found for JWT` error.


## 0c
| Number | Deliverable | Accepted | Link | Evaluation Notes |
| ------ | ----------- | -------- | ---- | ---------------- |
| 0c. | Testing and Testing Guide |<ul><li>[ ]</li></ul>| https://github.com/kzero-xyz/kzero-salt-enclave-service#readme | `./bin/app --test` only exercises helper utilities; no automated test covers `verify_jwt_for_provider`, `process_jwt_token`, or the enclave ECALL, and no guide explains the testing gap. |

- fix: 
  - Add `test_jwt_basic_errors` for testing `verify_jwt_for_provider
        - Notice: in the salt service, the `verify_jwt_for_provider` （like `google`）will always automatically fetch the latest JWK online, so if we use the same JWT in the test, we will encounter the `No matching key found for JWT` error. So, in the `test_http_server_real_requests` we use the fixed JWT & JWK, and avoid fetching the lated JWK online.
  - Add: `test_enclave_call_error`, used to test the error senario when processing jwt token and enclave call.
- RUNNING COMMAND
    > Pulling the latest version : `docker pull kzeroxyz/kzero-salt-enclave-service:v0.1.2`

    - Now, running test with :
    ```bash
    docker run --rm --name test-enclave-test -e SGX_MODE=SIM kzeroxyz/kzero-salt-enclave-service:v0.1.2 make test-app
    ```
    - Generate Test Coverage Report with :
    ```bash
    docker run --rm --name test-enclave-test -e SGX_MODE=SIM kzeroxyz/kzero-salt-enclave-service:v0.1.2 make test-coverage-app
    ```
  You should see the following coverage report:
  ```bash
  === Test Results ===
  All tests PASSED!
  === Test Suite Complete ===
  Generating coverage report...
  File 'App/App.cpp'
  Lines executed:93.49% of 568
  ```

# 0d
| Number | Deliverable | Accepted | Link | Evaluation Notes |
| ------ | ----------- | -------- | ---- | ---------------- |
| 0d. | Docker |<ul><li>[ ]</li></ul>| https://hub.docker.com/r/kzeroxyz/kzero-salt-enclave-service | Container builds, yet POSTing the supplied JWT to `/get_salt` returns 500 (`No matching key found for JWT`) because the live Google JWKS no longer provides that `kid`. |

Notice that, the salt service will alwary fetch the current JWT form the `https://www.googleapis.com/oauth2/v3/certs` as in the code. So, each time when trying to curl the `get_salt` api, we need to generate a real JWT, otherwise, if used the old JWT (for example, 1 week ago), it would mismatch the current JWK online. And return the `No matching key found for JWT` error.
So, when using this api, we should follow the **[kzero m2 turorial](https://github.com/kzero-xyz/kzero-grant-docs/blob/main/m2-docker-tutorial.md)**, using the kzero mvp demo to generate the real-time JWT (which match the real-time JWK online), thus will pass the JWK&JWT matching process and get the salt.

Fix: You can run this command, which embeded with a fix valid JWK & JWT:
```bash
curl -s -X POST http://localhost:8080/get_salt \
  -H 'Content-Type: application/json' \
  -d '{"message":"eyJhbGciOiJSUzI1NiIsImtpZCI6IjA3ZjA3OGYyNjQ3ZThjZDAxOWM0MGRhOTU2OWU0ZjUyNDc5OTEwOTQiLCJ0eXAiOiJKV1QifQ.eyJpc3MiOiJodHRwczovL2FjY291bnRzLmdvb2dsZS5jb20iLCJhenAiOiI1NjA2MjkzNjU1MTctbXQ5ajlhcmZsY2dpMzVpOGhwb3B0cjY2cWdvMWxtZm0uYXBwcy5nb29nbGV1c2VyY29udGVudC5jb20iLCJhdWQiOiI1NjA2MjkzNjU1MTctbXQ5ajlhcmZsY2dpMzVpOGhwb3B0cjY2cWdvMWxtZm0uYXBwcy5nb29nbGV1c2VyY29udGVudC5jb20iLCJzdWIiOiIxMTExNDA0NjE1MzAyNDYxNjQ1MjYiLCJub25jZSI6InlwanZ6TXB6d09qelcycUlrVnBiQU9UTUZuVSIsIm5iZiI6MTc1Nzc1MjA2NCwiaWF0IjoxNzU3NzUyMzY0LCJleHAiOjE3NTc3NTU5NjQsImp0aSI6ImZkYzRmNTc3YWI0NWViZjhiMjU3NjkwMjQwZmUzMTYyOGFkOGI4ZmMifQ.D4NVKogzU76ZGV5HsUDTOHRwSSG1I3lgG4bUEWAeMW8G-QDnXBNY6QDFmYnVEWWx5VlejyQhvmdtJrXF2eDOMKGeOwnFlm1INQuneELbLz0sbKnDw62IKshgQGNP5jv5ij-HEKj3jkx8D1zof83duVDhFOUmDud0VZKPODfBRLbqoTJKz0cp0RwZ5k-SiT_aSeL-y_FodYcCt5VtXIZfvgWj_NbcscqPaIBMvjJ9-wFx8yD-6C5dIQDVgyhZGtLzwxRLZMr6yotBuz_49BlKquuPA6TgNdUvMRu35QRYEQYPx3RigYtKw_8GGW-LVbmZTKSBOKu8QMEweR9CCaBHvg","provider":"test_google"}'
```


## 1
| Number | Deliverable | Accepted | Link | Evaluation Notes |
| ------ | ----------- | -------- | ---- | ---------------- |
| 1. | Salt Service Code |<ul><li>[ ]</li></ul>| https://github.com/kzero-xyz/kzero-salt-enclave-service | SGX host and enclave sources are present, but the documented demo flow fails due to the stale JWKS dependency. |

As describe above, the JWT list in the previous README might be expired, because google's JWK will rorate periodically, so when using `get_salt` api, must using a real-time JWT. (can follow the kzero [m2 tutorial](https://github.com/kzero-xyz/kzero-grant-docs/blob/main/m2-docker-tutorial.md) to get a real-time JWT)
But now, you can just use the curl command above.


## 2
| Number | Deliverable | Accepted | Link | Evaluation Notes |
| ------ | ----------- | -------- | ---- | ---------------- |
| 2. | docs |<ul><li>[ ]</li></ul>| https://github.com/kzero-xyz/kzero-grant-docs/blob/main/m3-docker-tutorial.md | External tutorial not validated; repository does not make it discoverable nor address the JWT/JWKS mismatch encountered during evaluation. |

Fix: used a fixed 'test-google' as provider to test the logic, this will not fetch the latest Google JWK, but used the fixed one.
```bash
curl -s -X POST http://localhost:8080/get_salt \
  -H 'Content-Type: application/json' \
  -d '{"message":"eyJhbGciOiJSUzI1NiIsImtpZCI6IjA3ZjA3OGYyNjQ3ZThjZDAxOWM0MGRhOTU2OWU0ZjUyNDc5OTEwOTQiLCJ0eXAiOiJKV1QifQ.eyJpc3MiOiJodHRwczovL2FjY291bnRzLmdvb2dsZS5jb20iLCJhenAiOiI1NjA2MjkzNjU1MTctbXQ5ajlhcmZsY2dpMzVpOGhwb3B0cjY2cWdvMWxtZm0uYXBwcy5nb29nbGV1c2VyY29udGVudC5jb20iLCJhdWQiOiI1NjA2MjkzNjU1MTctbXQ5ajlhcmZsY2dpMzVpOGhwb3B0cjY2cWdvMWxtZm0uYXBwcy5nb29nbGV1c2VyY29udGVudC5jb20iLCJzdWIiOiIxMTExNDA0NjE1MzAyNDYxNjQ1MjYiLCJub25jZSI6InlwanZ6TXB6d09qelcycUlrVnBiQU9UTUZuVSIsIm5iZiI6MTc1Nzc1MjA2NCwiaWF0IjoxNzU3NzUyMzY0LCJleHAiOjE3NTc3NTU5NjQsImp0aSI6ImZkYzRmNTc3YWI0NWViZjhiMjU3NjkwMjQwZmUzMTYyOGFkOGI4ZmMifQ.D4NVKogzU76ZGV5HsUDTOHRwSSG1I3lgG4bUEWAeMW8G-QDnXBNY6QDFmYnVEWWx5VlejyQhvmdtJrXF2eDOMKGeOwnFlm1INQuneELbLz0sbKnDw62IKshgQGNP5jv5ij-HEKj3jkx8D1zof83duVDhFOUmDud0VZKPODfBRLbqoTJKz0cp0RwZ5k-SiT_aSeL-y_FodYcCt5VtXIZfvgWj_NbcscqPaIBMvjJ9-wFx8yD-6C5dIQDVgyhZGtLzwxRLZMr6yotBuz_49BlKquuPA6TgNdUvMRu35QRYEQYPx3RigYtKw_8GGW-LVbmZTKSBOKu8QMEweR9CCaBHvg","provider":"test_google"}'
```

## 3
| Number | Deliverable | Accepted | Link | Evaluation Notes |
| ------ | ----------- | -------- | ---- | ---------------- |
| 3. | Docker |<ul><li>[ ]</li></ul>| https://hub.docker.com/r/kzeroxyz/kzero-salt-enclave-service | Published image inherits the same failing `/get_salt` behaviour; please update the image or instructions so evaluators can observe a successful salt response. |

- as above, use the testing curl command.
