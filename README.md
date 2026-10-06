# Andon

A CI/CD companion for iPhone that explains failed builds on the device.

Andon watches your pipelines on GitHub Actions and GitLab CI. When a job fails, it reduces the log to the lines that matter and asks an on-device language model why the job failed and what to change. The log is never sent to a server.

> **Status: design stage.** There is no application code in this repository yet. The architecture and the first tasks are defined; implementation starts with the project skeleton. Everything below describes the plan, not shipped behaviour.

## Why

A failed pipeline usually reaches you when you are away from your desk. Reading a twenty-megabyte log on a phone is not practical, and many teams are not allowed to paste build logs into a cloud assistant. Andon is for that gap: a short, checkable explanation, produced locally.

## How the analysis works

```
failed job log (megabytes)
        │
        ▼
deterministic reducer        strips noise, finds error anchors, keeps the lines
        │                    around them, packs the result into a token budget
        ▼
digest (a few thousand tokens, every line keeps its original line number)
        │
        ├──► shown as is — useful with no model at all
        ▼
on-device model              category, root cause, suggested change,
        │                    and the line numbers the answer is based on
        ▼
answer + the log lines that support it
```

Two design rules follow from this:

- **The model never sees the raw log.** An on-device model reads a few thousand tokens; a log is millions of characters. The reducer does the heavy lifting and is ordinary, testable code.
- **Every claim points at log lines.** A statement the model cannot tie to lines in the digest is dropped rather than shown.

The reducer and the model are measured separately against a labelled corpus of real failed logs. The corpus and the numbers will live in this repository.

## Privacy

The promise is narrow and meant to be verifiable from the source:

- The app connects only to the CI hosts you add. There is no Andon server.
- Log contents, digests and diagnoses stay on the device.
- Inference runs on the device. Server-hosted models, including Apple's Private Cloud Compute, are not used.
- No analytics and no third-party SDKs.
- Access tokens are stored in the Keychain and are not synced.

What this does not cover: your CI server sees the app's requests like any other client; the diagnosis is model output and can be wrong, which is why the supporting lines are always shown; the system model changes with OS updates.

## Planned scope

| | 1.0 | Later |
|---|---|---|
| Providers | GitHub Actions, GitLab CI (including self-managed) | Jenkins |
| Monitoring | Watched projects, runs, jobs, steps, logs | iPad, Mac, widgets |
| Analysis | Log digest; diagnosis with Apple Foundation Models | An open model as a second engine |
| Actions | Re-run, cancel, approve or reject deployments | |
| Notifications | Local notifications from background refresh | |

Failure notifications rely on iOS background refresh, so they are delayed by however long iOS decides. Real push needs a relay server, which 1.0 deliberately does not have.

## Requirements

- iOS 26.4 or later
- Diagnosis needs a device that supports Apple Intelligence, with it turned on. Without it the app still shows the digest.
- Xcode 26.6 to build

## How this project is built

Andon is written by AI coding agents working from written specifications, with a human owner who sets the task, accepts or rejects design decisions, and reviews every pull request. Agent-authored commits carry a `Co-Authored-By` trailer.

The project is also an experiment with a simple question: given precise instructions, does agent-written code hold up to the review standards applied to hand-written code? The quality criteria were fixed before any code existed — zero compiler warnings under Swift 6 strict concurrency, every acceptance criterion traced to a test, no dead or duplicated code, and a blind review by outside engineers before the public beta. The results will be published whichever way they come out.

## Roadmap

1. Project skeleton, quality gates, CI
2. GitHub Actions, read-only
3. Log reducer and the labelled corpus
4. On-device diagnosis
5. Actions and notifications
6. Public beta
7. GitLab CI
8. 1.0

## License

[MIT](LICENSE)
