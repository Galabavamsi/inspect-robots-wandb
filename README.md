# inspect-robots-wandb

The `inspect-robots-wandb` plugin logs Inspect Robots evaluation summaries to
[Weights & Biases](https://wandb.ai/).

## Install

```bash
pip install inspect-robots-wandb
wandb login
```

The plugin registers the `wandb` sink in the `inspect_robots.sinks` entry-point
group, so after installing it the sink appears in `inspect-robots list` and
resolves by name.

## Use from Python

Passing `sinks=` replaces the default JSON sink, so include both sinks when you
want the local immutable log and the W&B summary:

```python
from inspect_robots import eval
from inspect_robots.logging import JsonLogSink
from inspect_robots_wandb import WandbSink

logs = eval(
    "cubepick-reach",
    "scripted",
    "cubepick",
    sinks=[
        JsonLogSink("logs"),
        WandbSink(project="robot-evals", tags=("cubepick", "nightly")),
    ],
)
```

`WandbSink` creates one W&B run for each evaluation lifecycle. It stores the
evaluation specification as run config, then logs final status, scene and trial
counts, errored trials, total steps, duration, and every aggregate result metric.
The same sink instance can be reused by `eval_set()`; each task receives its own
W&B run.

> [!WARNING]
> The evaluation specification is uploaded as W&B run config. Keep credentials
> and other secrets out of policy and embodiment configuration.

If an evaluation aborts with an exception, the sink logs `eval/status: error`
with the exception text and finishes the run with exit code 1. This needs an
`inspect-robots` core with the optional `on_eval_error` sink hook. With older
cores the sink still works, but an aborted run is left for the W&B SDK to close
when the process exits.

Use `mode="offline"` to write a local W&B run for later synchronization, or
`mode="disabled"` to exercise the integration without recording data.

## Configuration

| Argument | Default | Meaning |
| --- | --- | --- |
| `project` | `inspect-robots` | W&B project name. |
| `entity` | unset | W&B team or user name. |
| `name` | unset | Display name for the run. |
| `group` | unset | W&B run group. |
| `tags` | unset | Sequence of W&B tags. |
| `mode` | unset | W&B mode such as `online`, `offline`, or `disabled`. |
| `dir` | unset | Local directory used by W&B. |

## Development

```bash
uv sync --extra dev
uv run ruff check . && uv run ruff format --check . && uv run mypy
uv run pytest
```

The tests replace the W&B SDK with an in-process double, so they need no
network access or W&B account.

## License

MIT
