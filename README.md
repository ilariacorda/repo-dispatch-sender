# Repository Dispatch Sender

This repository sends a `repository_dispatch` event to another GitHub repository when a Git tag is pushed or when manually triggered from the Actions tab.

This is useful for triggering workflows in downstream repos like `repo-dispatch-receiver`.

---

## How It Works

1. A new new tag (e.g. `v1.2.3`) is pushed
2. A GitHub Action triggers and sends a `repository_dispatch` event to another repository
3. The receiver repository listens and runs its workflow

---

## Workflow File

**Location:** `.github/workflows/send-dispatch.yaml`

```yaml
name: Send Dispatch on Tag

on:
  push:
    tags:
      - 'v*'
      - '!v*-*'  # Exclude pre-releases like v1.2.3-beta
  workflow_dispatch:

jobs:
  dispatch:
    runs-on: ubuntu-latest
    steps:
      - name: Send repository_dispatch to receiver repo
        uses: peter-evans/repository-dispatch@v2
        with:
          token: ${{ secrets.TEST_DISPATCH_TOKEN }}
          repository: ilariacorda/repo-dispatch-receiver
          event-type: test-dispatch
          client-payload: |
            {
              "version": "${{ github.ref_name || 'manual-test' }}",
              "repo": "${{ github.repository }}"
            }
```

---

## Token Setup (REQUIRED)

It is **required** to create a classic Personal Access Token (PAT)** and store it in this repository’s secrets.

> Fine-grained tokens currently do **not** work reliably for `repository_dispatch` and ultimately the Class PAT is the recommend approach. 

### How to Generate a Classic PAT

1. Go to: [GitHub → Settings → Developer Settings → Personal Access Tokens → Tokens (classic)](https://github.com/settings/tokens)
2. Click **Generate new token (classic)**
3. Select:
   - ✅ `repo` (full repo access)
   - ✅ `workflow` (trigger Actions)
4. Set an expiration (e.g. 30 days) - this usually the recommend duration
5. Click **Generate Token**
6. **Copy the token immediately** — 

### Add the Token as a Secret

In this repository:

1. Go to `Settings → Secrets and variables → Actions`
2. Click **New repository secret**
3. Enter:
   - **Name:** `TEST_DISPATCH_TOKEN`
   - **Value:** (paste your token)

---

## How to Test It

### Option 1: Manual Trigger

1. Go to the **Actions** tab
2. Select **Send Dispatch on Tag**
3. Click **Run workflow**
4. This should be seen:
   ```
   Response: 204 No Content
   ```

### Option 2: Push a Version Tag

```bash
git tag v1.2.3
git push origin v1.2.3
```

This will trigger the workflow and dispatch the event.

---

## Payload Sent

Example payload:

```json
{
  "version": "v1.2.3",
  "repo": "<gitusername>/repo-dispatch-sender"
}
```

This will be received by `repo-dispatch-receiver`.
