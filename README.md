# palantir

<p align="center">
  <strong>See into any cluster, one keystroke at a time.</strong><br>
  A fuzzy-finder driven Kubernetes navigator for developers who don't want to memorize kubectl.
</p>

<p align="center">
  <a href="https://github.com/prashant0085/palantir/stargazers"><img src="https://img.shields.io/github/stars/prashant0085/palantir?style=flat-square" alt="GitHub stars"></a>
  <img src="https://img.shields.io/badge/shell-bash-121011?style=flat-square&logo=gnu-bash&logoColor=white" alt="Bash">
</p>

<p align="center">
  <img src="demo/palantir-demo-v1.svg" alt="Animated palantir terminal demo" width="900">
</p>

`palantir` is a `k9s`-style terminal navigator for Kubernetes, built on top of `fzf` instead of a custom TUI. Pick a context, drill into a namespace, browse resources, and act on them — all with arrow keys and fuzzy search, no flags to remember.

It's aimed at developers who need to check on their app in a cluster, not at operators who already live in `kubectl`.

## Highlights

- Context -> namespace -> resource type -> resource, all fuzzy-searchable with `fzf`
- Live `kubectl describe` preview pane while browsing resources
- One-key actions per resource: describe, get YAML, logs (including `-p` and `-f`), exec shell, delete
- `__back__` at every level to step up instead of restarting
- Guarded delete with a typed confirmation
- Zero config, zero flags — just run `palantir`

## Install

### One-line install

Install the latest version into `~/.local/bin` without `sudo`:

```bash
curl -fsSL https://raw.githubusercontent.com/prashant0085/palantir/main/install.sh | bash
```

Then run:

```bash
palantir
```

If `~/.local/bin` is not in your `PATH`:

```bash
export PATH="$HOME/.local/bin:$PATH"
```

### Review before installing

```bash
curl -fsSL https://raw.githubusercontent.com/prashant0085/palantir/main/install.sh -o /tmp/palantir-install.sh
less /tmp/palantir-install.sh
bash /tmp/palantir-install.sh
```

### From source

```bash
git clone https://github.com/prashant0085/palantir.git
cd palantir
mkdir -p ~/.local/bin
cp palantir ~/.local/bin/palantir
chmod +x ~/.local/bin/palantir
```

## Quick start

```bash
palantir
```

1. Pick a kube-context with fuzzy search.
2. Pick a namespace.
3. Pick a resource type: pods, deployments, services, secrets, configmaps, ingress, statefulsets, daemonsets, jobs, cronjobs, replicasets, events, pvc, hpa, or nodes.
4. Pick a resource — a live `describe` preview shows on the right as you move through the list.
5. Pick an action: `describe`, `get-yaml`, `delete`, and for pods also `logs`, `logs-previous`, `logs-follow`, `exec-sh`.

`__back__` is always the first item in a list — pick it (or press `Esc`) to go up one level instead of exiting the whole tool.

## Why not just use k9s?

`k9s` is a great tool if you already know Kubernetes resource types and live in a terminal all day. `palantir` optimizes for a different reader: a developer who wants to answer "what's going on with my pod" without knowing the resource kind, the right flag, or the YAML path to look at. It reuses `fzf`, a tool many developers already have muscle memory for from `kubectx`/`kubens`, instead of introducing a new keybinding scheme.

## Requirements

| Tool | Used for |
| --- | --- |
| `kubectl` | All cluster interaction |
| `fzf` | Interactive selection at every level |

`palantir` assumes your kubeconfig contexts are already set up (e.g. via `kubectx` or manually).

## Responsible use

`delete` prompts for a `y` confirmation before removing anything. Even so, `palantir` operates with whatever RBAC permissions your current kube-context has — treat it the same as `kubectl` and only point it at clusters and namespaces you're authorized to modify.

## Project status

`palantir` is a young project, currently a single bash script. Ideas and contributions are welcome, especially around faster resource-list caching and smarter, app-centric navigation instead of raw resource-type browsing.

### TODO

- Replace the illustrative animated SVG with a recorded terminal demo, similar to the demo used by [`kube-ps1`](https://github.com/jonmosco/kube-ps1).

**High value, dev-facing:**

- "Why is my pod broken?" mode — instead of a raw `describe`, detect `CrashLoopBackOff`/`ImagePullBackOff`/`Pending`/`OOMKilled` and print a plain-English diagnosis (last restart reason, exit code, relevant events) instead of dumping full YAML for the developer to parse themselves.
- App/label-based grouping instead of resource-type browsing — developers think "my service `checkout`", not "list all deployments then all pods then match them up." Let them fuzzy-search by app/label name across pods+deployments+services+ingress at once and show everything related to that app on one screen.
- One-key common actions — `l` for logs (auto `tail -f`), `r` for restart (`rollout restart`), `s` for shell exec, `p` for port-forward — without needing to know the underlying flags exist.
- Port-forward shortcut — huge for developers debugging locally; k9s has this but it's buried. Surface it as a first-class action with an auto-picked local port.
- Config/secret diff-friendly view — decode secrets and pretty-print configmaps by default (developers don't know the `-o jsonpath` base64-decode tricks).

**Nice-to-have polish:**

- Remember recent/favorite context+namespace combos to skip re-navigating every time.
- Copy-to-clipboard for pod name / image / logs snippet (for pasting into Slack when asking DevOps for help).
- Non-destructive by default — hide `delete`/`edit` unless a `--dangerous` flag is set or a typed-name confirmation step is completed, since this tool targets developers, not admins.

**Speed and rendering:**

- Cache resource lists per level for a few seconds to reduce repeated `kubectl get` calls and speed up navigation, instead of re-spawning a `kubectl get -o name` call and a `describe` preview subprocess on every screen and keystroke.
- Size `--preview-window` relative to the terminal instead of a fixed percentage, to fix layout not scaling on some terminals.

## License

This project is currently under active development. A formal open-source license will be added before the first stable release.
