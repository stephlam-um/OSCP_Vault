**Summary:**  run-gemini-cli  /setup-github creates insecure github actions

***The vulnerability is known to third parties!***

**Program:** OSS VRP

**URL:** https://github.com/google-gemini/gemini-cli

**Vulnerability type:** Permissions Bypass

### Details

Summary: The /setup-github command in gemini-cli downloads 5 example github acitons workflows from the run-gemini-cli github repository (https://github.com/google-github-actions/run-gemini-cli). However, the `gemini-dispatch.yml` workflow is insecurely implemented, allowing all users to route to a priviledged `gemini-invoke.yml` workflow by creating an issue starting with `@gemini-cli <prompt>`, despite the comment in `gemini-dispatch.yml` suggesting that only trusted users (owner, member, or collaborator) can invoke `@gemini-cli` this way.

Exact line for downloading the workflows is in:
https://github.com/google-gemini/gemini-cli/blob/1202dced7339ef05e0638442853ee614bab0be03/packages/cli/src/ui/commands/setupGithubCommand.ts#L144
```js
    // Download each workflow in parallel - there aren't enough files to warrant
    // a full workerpool model here.
    const downloads = [];
    for (const workflow of GITHUB_WORKFLOW_PATHS) {
      downloads.push(
        (async () => {
          const endpoint = `https://raw.githubusercontent.com/google-github-actions/run-gemini-cli/refs/tags/${releaseTag}/examples/workflows/${workflow}`;
```

By default it adds 5 workflow files:
```
export const GITHUB_WORKFLOW_PATHS = [
  'gemini-dispatch/gemini-dispatch.yml',
  'gemini-assistant/gemini-invoke.yml',
  'issue-triage/gemini-triage.yml',
  'issue-triage/gemini-scheduled-triage.yml',
  'pr-review/gemini-review.yml',
];
```

All workflows except "issue-triage/gemini-scheduled-triage.yml" for are meant to be dispatched by `gemini-dispatch/gemini-dispatch.yml`

https://github.com/google-github-actions/run-gemini-cli/blob/8a300991f26b2958bf7da6e100f99af84a2faf24/examples/workflows/gemini-dispatch/gemini-dispatch.yml
The main parts are copied below. (I will elaborate below the code block)
```
on:
  pull_request_review_comment:
    types:
      - 'created'
  pull_request_review:
    types:
      - 'submitted'
  pull_request:
    types:
      - 'opened'
  issues:
    types:
      - 'opened'
      - 'reopened'
  issue_comment:
    types:
      - 'created'

jobs:
  dispatch:
    # For PRs: only if not from a fork
    # For comments: only if user types @gemini-cli and is OWNER/MEMBER/COLLABORATOR
    # For issues: only on open/reopen
    if: |-
      (
        github.event_name == 'pull_request' &&
        github.event.pull_request.head.repo.fork == false
      ) || (
        github.event.sender.type == 'User' &&
        startsWith(github.event.comment.body || github.event.review.body || github.event.issue.body, '@gemini-cli') &&
        contains(fromJSON('["OWNER", "MEMBER", "COLLABORATOR"]'), github.event.comment.author_association || github.event.review.author_association || github.event.issue.author_association)
      ) || (
        github.event_name == 'issues' &&
        contains(fromJSON('["opened", "reopened"]'), github.event.action)
      )
    runs-on: 'ubuntu-latest'
    permissions:
      contents: 'read'
      issues: 'write'
      pull-requests: 'write'
    outputs:
      command: '${{ steps.extract_command.outputs.command }}'
      request: '${{ steps.extract_command.outputs.request }}'
      additional_context: '${{ steps.extract_command.outputs.additional_context }}'
      issue_number: '${{ github.event.pull_request.number || github.event.issue.number }}'
    steps:
      - name: 'Mint identity token'
        id: 'mint_identity_token'
        if: |-
          ${{ vars.APP_ID }}
        uses: 'actions/create-github-app-token@a8d616148505b5069dccd32f177bb87d7f39123b' # ratchet:actions/create-github-app-token@v2
        with:
          app-id: '${{ vars.APP_ID }}'
          private-key: '${{ secrets.APP_PRIVATE_KEY }}'
          permission-contents: 'read'
          permission-issues: 'write'
          permission-pull-requests: 'write'

      - name: 'Extract command'
        id: 'extract_command'
        uses: 'actions/github-script@60a0d83039c74a4aee543508d2ffcb1c3799cdea' # ratchet:actions/github-script@v7
        env:
          EVENT_TYPE: '${{ github.event_name }}.${{ github.event.action }}'
          REQUEST: '${{ github.event.comment.body || github.event.review.body || github.event.issue.body }}'
        with:
          script: |
            const request = process.env.REQUEST;
            const eventType = process.env.EVENT_TYPE
            core.setOutput('request', request);

            if (request.startsWith("@gemini-cli /review")) {
              core.setOutput('command', 'review');
              const additionalContext = request.replace(/^@gemini-cli \/review/, '').trim();
              core.setOutput('additional_context', additionalContext);
            } else if (request.startsWith("@gemini-cli /triage")) {
              core.setOutput('command', 'triage');
            } else if (request.startsWith("@gemini-cli")) {
              core.setOutput('command', 'invoke');
              const additionalContext = request.replace(/^@gemini-cli/, '').trim();
              core.setOutput('additional_context', additionalContext);
            } else if (eventType === 'pull_request.opened') {
              core.setOutput('command', 'review');
            } else if (['issues.opened', 'issues.reopened'].includes(eventType)) {
              core.setOutput('command', 'triage');
            } else {
              core.setOutput('command', 'fallthrough');
            }
  review:
    needs: 'dispatch'
    if: |-
      ${{ needs.dispatch.outputs.command == 'review' }}
    # ...

  triage:
    needs: 'dispatch'
    if: |-
      ${{ needs.dispatch.outputs.command == 'triage' }}
    # ...

  invoke:
    needs: 'dispatch'
    if: |-
      ${{ needs.dispatch.outputs.command == 'invoke' }}
    uses: './.github/workflows/gemini-invoke.yml'
    permissions:
      contents: 'read'
      id-token: 'write'
      issues: 'write'
      pull-requests: 'write'
    with:
      additional_context: '${{ needs.dispatch.outputs.additional_context }}'
    secrets: 'inherit'
```

Three things to note in the workflow above:
1. The comment under the dispatch job says "For comments: only if user types @gemini-cli and is OWNER/MEMBER/COLLABORATOR". 
2. REQUEST: '${{ github.event.comment.body || github.event.review.body || github.event.issue.body }}' coerces a comment body, review body, and issue body into the REQUEST variable, such that it is populated in all cases where this workflow is triggered.
3.  **The main vulunerability**: The output of the dispatch job checks the content of `process.env.REQUEST` BEFORE checking the eventType.

Therefore, ANY user who opens an issue or a pull request, and write in the issue/PR body `@gemini-cli <prompt>` can trigger the `gemini-invoke.yml` action at will, due to `request.startsWith("@gemini-cli")` being matched before `(eventType === 'pull_request.opened')` or `(['issues.opened', 'issues.reopened'].includes(eventType))`.

From the comment in the workflow file "For comments: only if user types @gemini-cli and is OWNER/MEMBER/COLLABORATOR" and the way the checks are written, it is clear that the intended flow of this dispatch job is to trigger the `review` action if a PR is opened, and the `triage` action when an issue is opened. It is not intended to trigger the `invoke` action.

Below is the `gemini-assistant/gemini-invoke.yml` workflow minus the prompt. Again comment on the code after the code itself.
https://github.com/google-github-actions/run-gemini-cli/blob/8a300991f26b2958bf7da6e100f99af84a2faf24/examples/workflows/gemini-assistant/gemini-invoke.yml


```
name: '▶️ Gemini Invoke'

on:
  workflow_call:
    inputs:
      additional_context:
        type: 'string'
        description: 'Any additional context from the request'
        required: false

concurrency:
  group: '${{ github.workflow }}-invoke-${{ github.event_name }}-${{ github.event.pull_request.number || github.event.issue.number }}'
  cancel-in-progress: false

defaults:
  run:
    shell: 'bash'

jobs:
  invoke:
    runs-on: 'ubuntu-latest'
    permissions:
      contents: 'read'
      id-token: 'write'
      issues: 'write'
      pull-requests: 'write'
    steps:
      - name: 'Mint identity token'
        id: 'mint_identity_token'
        if: |-
          ${{ vars.APP_ID }}
        uses: 'actions/create-github-app-token@a8d616148505b5069dccd32f177bb87d7f39123b' # ratchet:actions/create-github-app-token@v2
        with:
          app-id: '${{ vars.APP_ID }}'
          private-key: '${{ secrets.APP_PRIVATE_KEY }}'
          permission-contents: 'read'
          permission-issues: 'write'
          permission-pull-requests: 'write'

      - name: 'Run Gemini CLI'
        id: 'run_gemini'
        uses: 'google-github-actions/run-gemini-cli@v0' # ratchet:exclude
        env:
          TITLE: '${{ github.event.pull_request.title || github.event.issue.title }}'
          DESCRIPTION: '${{ github.event.pull_request.body || github.event.issue.body }}'
          EVENT_NAME: '${{ github.event_name }}'
          GITHUB_TOKEN: '${{ steps.mint_identity_token.outputs.token || secrets.GITHUB_TOKEN || github.token }}'
          IS_PULL_REQUEST: '${{ !!github.event.pull_request }}'
          ISSUE_NUMBER: '${{ github.event.pull_request.number || github.event.issue.number }}'
          REPOSITORY: '${{ github.repository }}'
          ADDITIONAL_CONTEXT: '${{ inputs.additional_context }}'
        with:
          gcp_location: '${{ vars.GOOGLE_CLOUD_LOCATION }}'
          gcp_project_id: '${{ vars.GOOGLE_CLOUD_PROJECT }}'
          gcp_service_account: '${{ vars.SERVICE_ACCOUNT_EMAIL }}'
          gcp_workload_identity_provider: '${{ vars.GCP_WIF_PROVIDER }}'
          gemini_api_key: '${{ secrets.GEMINI_API_KEY }}'
          gemini_cli_version: '${{ vars.GEMINI_CLI_VERSION }}'
          gemini_debug: '${{ fromJSON(vars.DEBUG || vars.ACTIONS_STEP_DEBUG || false) }}'
          gemini_model: '${{ vars.GEMINI_MODEL }}'
          google_api_key: '${{ secrets.GOOGLE_API_KEY }}'
          use_gemini_code_assist: '${{ vars.GOOGLE_GENAI_USE_GCA }}'
          use_vertex_ai: '${{ vars.GOOGLE_GENAI_USE_VERTEXAI }}'
          settings: |-
            {
              "model": {
                "maxSessionTurns": 25
              },
              "telemetry": {
                "enabled": ${{ vars.GOOGLE_CLOUD_PROJECT != '' }},
                "target": "gcp"
              },
              "mcpServers": {
                "github": {
                  "command": "docker",
                  "args": [
                    "run",
                    "-i",
                    "--rm",
                    "-e",
                    "GITHUB_PERSONAL_ACCESS_TOKEN",
                    "ghcr.io/github/github-mcp-server"
                  ],
                  "includeTools": [
                    "add_issue_comment",
                    "get_issue",
                    "get_issue_comments",
                    "list_issues",
                    "search_issues",
                    "create_pull_request",
                    "get_pull_request",
                    "get_pull_request_comments",
                    "get_pull_request_diff",
                    "get_pull_request_files",
                    "list_pull_requests",
                    "search_pull_requests",
                    "create_branch",
                    "create_or_update_file",
                    "delete_file",
                    "fork_repository",
                    "get_commit",
                    "get_file_contents",
                    "list_commits",
                    "push_files",
                    "search_code"
                  ],
                  "env": {
                    "GITHUB_PERSONAL_ACCESS_TOKEN": "${GITHUB_TOKEN}"
                  }
                }
              },
              "tools": {
                "core": [
                  "run_shell_command(cat)",
                  "run_shell_command(echo)",
                  "run_shell_command(grep)",
                  "run_shell_command(head)",
                  "run_shell_command(tail)"
                ]
              }
            }
          prompt: |-
```

It can be seen that this action has `issues: 'write'` and `pull-requests: 'write'` attached to it, along with GitHub MCP with tools like add_issue_comment, create_pull_request, create_branch, AND **importantly** the environmnet variable `"GITHUB_PERSONAL_ACCESS_TOKEN": "${GITHUB_TOKEN}"`. This is not available in other action files in the run-gemini-cli examples.

For example, in `gemini-triage.yml` it is explicitly mentioned to NOT pass the GITHUB_TOKEN due to being run on untrusted inputs.

https://github.com/google-github-actions/run-gemini-cli/blob/8a300991f26b2958bf7da6e100f99af84a2faf24/examples/workflows/issue-triage/gemini-triage.yml#L59-L63

```
        env:
          GITHUB_TOKEN: '' # Do NOT pass any auth tokens here since this runs on untrusted inputs
          ISSUE_TITLE: '${{ github.event.issue.title }}'
          ISSUE_BODY: '${{ github.event.issue.body }}'
          AVAILABLE_LABELS: '${{ steps.get_labels.outputs.available_labels }}'
```

As the gemini instance in `gemini-assistant/gemini-invoke.yml` is allowed to 
a. Have access to priviledged credentials
b. Run on untrusted inputs such that prompt injection is possible
c. Write output to the external world via (e.g.) add_issue_comment

The [lethal trifecta](https://simonwillison.net/2025/Jun/16/the-lethal-trifecta/) is complete and so
1. The gemini instance may do harmful actions with the github tools it have
2. The GITHUB_TOKEN may be leaked.

### Attack scenario

### Who can exploit the vulnerability
All public github repositories using the /setup-github command is affected.

### What they gain when doing so
Mainly massive damage to the repository in a reversible but time-consuming way, with small probability of escalating via other workflows that triggers by actions done to issues/PRs.
1. The gemini instance may do harmful actions with the github tools it have via prompt injection
2. The GITHUB_TOKEN may be leaked, and all actions allowed by `issues: 'write'` and `pull-requests: 'write'` can be done by the attacker.
2.1 creating massive amount of spam in the repository, deleting all issues and pull requests in the repository
2.2. Add labels to issues, and cause workflows that depends on certain labels to be added to be run
3. A massive drain on github's free CI on public repositories by creating PR's that create more PRs in a chain reaction.