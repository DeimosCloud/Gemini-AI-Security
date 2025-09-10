# 🔀 Gemini Dispatch (GitHub Actions)

**Purpose:** Route PRs, issues, and `@gemini-cli` comments to the right AI workflow (review, triage, or general invoke) and acknowledge the request with a run link.  
**Workflow file:** `.github/workflows/gemini-dispatch.yml`

---

## How it works

**Listens to:**  
`pull_request.opened` · `pull_request_review.submitted` · `pull_request_review_comment.created` · `issues.opened|reopened` · `issue_comment.created`

**Router logic (auto & commands):**
- **PR opened** → `gemini-review.yml`
- **Issue opened/reopened** → `gemini-triage.yml`
- **Comment starts with `@gemini-cli /review …`** → `gemini-review.yml` (passes trailing text as `additional_context`)
- **Comment starts with `@gemini-cli /triage`** → `gemini-triage.yml`
- **Comment starts with `@gemini-cli …` (anything else)** → `gemini-invoke.yml`
- Otherwise → posts a helpful fallthrough comment

**Guards & QoL:**
- Command comments only run for `OWNER`, `MEMBER`, or `COLLABORATOR`.
- Auto-review is skipped for **fork** PRs.
- Adds a brief delay on PRs so OSV scanning can spin up.

---

## Repo setup

```text
.github/
  workflows/
    gemini-dispatch.yml   # router (this file)
    gemini-review.yml     # reusable: review
    gemini-triage.yml     # reusable: triage
    gemini-invoke.yml     # reusable: general invoke
```

**Required configuration**
- **Variables**
  - `APP_ID` — GitHub App ID used for commenting
- **Secrets**
  - `APP_PRIVATE_KEY` — PEM for the GitHub App

**Optional**
- `DEBUG=true` (Repository Variable) to print event context.

**Permissions**
- Uses least-privilege writes to issues/PRs and contents; comments via short-lived GitHub App token.
- Reusable workflows are invoked with `uses: ./.github/workflows/<file>.yml` and `secrets: inherit`.

---

## Usage

**Automatic**
- Open a **PR** → bot queues a **review**
- Open/reopen an **issue** → bot runs **triage**
- **On Merge to default branch** or **scheduled run** → scans may create **issues** for vulnerabilities/dependency updates

### Create your own specific workflows

You can add new reusable workflows in `.github/workflows/` (e.g., a PR builder or vulnerability fixer) and route new commands to them from the dispatcher. These workflows are examples to start from—**fork and shape them to your team’s needs**.


**Command comments (trusted users)**
```text
@gemini-cli /review Focus on API breaking changes
@gemini-cli /triage
@gemini-cli Generate a concise changelog from the diff

The dispatcher acknowledges your request with a link to the run and forwards it to the selected reusable workflow.