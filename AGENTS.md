# Repository Guidelines

This repository is a personal learning notebook. Keep each contribution focused on a question, experiment, or lesson that will be easy to revisit later. Preserve the author's first-person observations when editing notes.

## Finding a home for a note

- Use `networking/`, `monitoring/`, `aws/`, or `k8s-storage/` for topic-specific notes. Follow the numbered sequence when adding a lesson to an ordered series such as `gitops-flux-lab/flux-basic/`.
- Keep runnable examples beside their explanations. In `k8s-homelab/`, cluster configuration lives in `config/`, Kubernetes resources in `manifests/`, and setup scripts in `scripts/`.
- Keep diagrams near the notes that reference them, as in `gitops-flux-lab/flux-basic/photos/`. Use relative Markdown links so images work in a local checkout.

## Writing and naming

Use descriptive, lowercase, hyphenated names such as `block-pool.yaml`. For a lab, record the starting environment, the commands you tried, and the output that showed whether they worked. For troubleshooting notes, include the symptom and the fix. Use fenced code blocks with a language tag such as `bash` or `yaml`. Match the surrounding manifests' two-space YAML indentation and use placeholders for credentials.

## Checking a change

Checks depend on the topic. Preview edited Markdown and follow any changed links. For a shell script, check syntax from the repository root, for example with `bash -n k8s-homelab/scripts/init-master.sh`.

For Flux infrastructure changes, run these targets from `gitops-flux-lab/infrastructure/`. Both default to staging.

- `make validate-infra` checks that Kustomize can assemble the overlay.
- `make build-infra` prints the rendered resources for inspection.

## Commits and pull requests

Keep linked Git worktrees under `worktrees/<name>/` at the repository root. The repository's `.gitignore` covers this directory. Run `git worktree list` before creating or removing one.

Use a short, specific commit subject like the existing `Add S3 FUSE guide`. Sign off with `git commit -s`. In a pull request, identify the topic, summarize what you learned or corrected, and mention the checks you ran. Include an issue link when the change follows up on a tracked question.
