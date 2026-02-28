# 6. Use Cases

## Use Case 1: Privileged Command Execution

### Scenario

An AI agent operating on a server needs to execute a command that requires elevated privileges — for example, installing a package, modifying system configuration, or restarting a service.

### Flow

1. The agent determines it needs to run `systemctl restart nginx`.
2. The agent authenticates with the Grants Server via DDISA.
3. The agent submits a grant request:
   - **command:** `systemctl restart nginx`
   - **reason:** "Configuration updated for project X, restart required to apply changes"
   - **cmd_hash:** `sha256("systemctl restart nginx")`
4. The human approver reviews the command and reason, then approves with `allow_once`.
5. The agent receives an AuthZ-JWT.
6. The agent presents the JWT to the execution proxy (e.g., `apes` — the OpenApe sudo equivalent).
7. `apes` verifies the JWT signature, checks `cmd_hash` against the actual command, and executes.

### What Grants Protect

Without grants, the agent would need persistent sudo access. With grants, each privileged command requires explicit approval, the command is cryptographically bound to the approval, and there is a full audit trail of who requested and who approved.

## Use Case 2: HTTP Proxy with Approval

### Scenario

An AI agent needs to make HTTP requests to external services (APIs, websites) but operates in a restricted network environment where outbound access requires authorization.

### Flow

1. The agent needs to call `POST https://api.cloud.example.com/v1/instances` to provision a resource.
2. The agent submits a grant request:
   - **command:** `POST https://api.cloud.example.com/v1/instances {"type":"small","region":"eu"}`
   - **reason:** "Provisioning compute instance for load testing"
   - **request_hash:** `sha256("POST https://api.cloud.example.com/v1/instances\n{\"type\":\"small\",\"region\":\"eu\"}")`
3. The human approves with `allow_once`.
4. The agent receives an AuthZ-JWT containing the `request_hash`.
5. The agent sends the HTTP request through an authorized proxy, attaching the JWT.
6. The proxy verifies the JWT, recomputes the `request_hash` from the actual request, confirms it matches, and forwards the request.

### What Grants Protect

The proxy ensures agents cannot make arbitrary HTTP requests. Each request is individually approved and bound by hash, preventing the agent from substituting a different API call after approval.

## Use Case 3: Service Account Operations

### Scenario

An AI agent manages cloud infrastructure using service account credentials. Operations such as creating resources, modifying IAM policies, or deleting instances require authorization.

### Flow

1. The agent needs to update an IAM policy to grant a new service read access to a storage bucket.
2. The agent submits a grant request:
   - **command:** `gcloud iam policies add-binding storage-bucket-x --member=sa:new-service --role=reader`
   - **reason:** "New analytics service needs read access to ingest data from bucket X"
   - **cmd_hash:** computed from the exact command
3. The human approver reviews. The UI highlights this as a permission-changing operation (high risk).
4. The approver grants `allow_once`.
5. The agent receives the AuthZ-JWT and presents it to the operations gateway.
6. The gateway verifies the token and executes the operation using the service account credentials.

### What Grants Protect

Service account credentials are powerful and long-lived. Grants add a per-operation approval layer on top, ensuring that no IAM change or resource modification happens without human review. The `decided_by` claim creates accountability — if an IAM change causes an incident, the audit trail shows both the requesting agent and the approving human.
