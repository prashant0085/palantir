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
- Live preview pane while browsing resources — `describe` for most kinds, decoded values for secrets, and pretty-printed data for configmaps
- Preview pane resizes based on terminal width instead of using a fixed layout
- Resource lists are cached for a few seconds per context/namespace/type to avoid re-running `kubectl get` on every keystroke
- Plain-English pod diagnosis (`d`) — surfaces `CrashLoopBackOff`/`ImagePullBackOff`/`OOMKilled`/restart counts and recent events instead of raw YAML
- One-key actions directly from the pod list: `l` logs (follow), `r` restart, `s` shell, `p` port-forward, `d` diagnose, `y` copy pod name to clipboard
- Port-forward with an automatically picked free local port
- Remembers your last context + namespace and offers them first next run
- `__back__` at every level to step up instead of restarting
- Non-destructive by default — `delete` is hidden unless you start with `palantir --dangerous`, and even then requires typing the resource name to confirm
- Zero config, zero required flags — just run `palantir`

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

1. Pick a kube-context with fuzzy search (your last context + namespace is offered first).
2. Pick a namespace.
3. Pick a resource type: pods, deployments, services, secrets, configmaps, ingress, statefulsets, daemonsets, jobs, cronjobs, replicasets, events, pvc, hpa, or nodes.
4. Pick a resource — a live preview shows on the right as you move through the list (decoded values for secrets, pretty-printed data for configmaps, `describe` for everything else).
5. Pick an action: `describe`, `get-yaml`, `copy-name`, and for pods also `diagnose`, `logs`, `logs-previous`, `logs-follow`, `exec-sh`, `port-forward`. `delete` only appears when you run `palantir --dangerous`, and it requires typing the resource name to confirm.

`__back__` is always the first item in a list — pick it (or press `Esc`) to go up one level instead of exiting the whole tool.

### Pod list shortcuts

While browsing pods, you don't need to open the action menu for common tasks:

| Key | Action |
| --- | --- |
| `l` | Tail logs (`kubectl logs -f`) |
| `r` | Restart the pod (deletes it so its controller recreates it) |
| `s` | Exec into a shell |
| `p` | Port-forward, with an automatically picked free local port |
| `d` | Plain-English diagnosis (crash reasons, restart counts, recent events) |
| `y` | Copy the pod name to the clipboard |

## Why not just use k9s?

`k9s` is a great tool if you already know Kubernetes resource types and live in a terminal all day. `palantir` optimizes for a different reader: a developer who wants to answer "what's going on with my pod" without knowing the resource kind, the right flag, or the YAML path to look at. It reuses `fzf`, a tool many developers already have muscle memory for from `kubectx`/`kubens`, instead of introducing a new keybinding scheme.

## Requirements

| Tool | Used for |
| --- | --- |
| `kubectl` | All cluster interaction |
| `fzf` | Interactive selection at every level |
| `jq` | Decoding secrets and pretty-printing configmaps, pod diagnosis |
| `pbcopy` | Copy-to-clipboard (macOS only; skip this action on other platforms) |
| `nc` | Finding a free local port for port-forwarding |

`palantir` assumes your kubeconfig contexts are already set up (e.g. via `kubectx` or manually).

## Responsible use

`delete` is hidden entirely unless you run `palantir --dangerous`, and even then it requires typing the resource name to confirm before anything is removed. Even so, `palantir` operates with whatever RBAC permissions your current kube-context has — treat it the same as `kubectl` and only point it at clusters and namespaces you're authorized to modify.

## Project status

`palantir` is a young project, currently a single bash script. Ideas and contributions are welcome, especially around faster resource-list caching and smarter, app-centric navigation instead of raw resource-type browsing.

### TODO

- Replace the illustrative animated SVG with a recorded terminal demo, similar to the demo used by [`kube-ps1`](https://github.com/jonmosco/kube-ps1).
- App/label-based grouping instead of resource-type browsing — developers think "my service `checkout`", not "list all deployments then all pods then match them up." Let them fuzzy-search by app/label name across pods+deployments+services+ingress at once and show everything related to that app on one screen. This is a bigger navigation-model change than the items below and needs its own design pass.

Done:

- ~~"Why is my pod broken?" mode~~ — see `d` (diagnose) shortcut and the `diagnose` action.
- ~~One-key common actions~~ — `l`/`r`/`s`/`p`/`d`/`y` on the pod list.
- ~~Port-forward shortcut~~ — `p` action, auto-picks a free local port.
- ~~Config/secret diff-friendly view~~ — secrets are base64-decoded and configmaps pretty-printed in the preview pane.
- ~~Remember recent context+namespace~~ — last selection is offered first on the next run.
- ~~Copy-to-clipboard~~ — `y` on the pod list, or the `copy-name` action.
- ~~Non-destructive by default~~ — `delete` requires `palantir --dangerous` plus a typed-name confirmation.
- ~~Resource list caching~~ — cached per context/namespace/type for a few seconds.
- ~~Relative preview sizing~~ — preview pane switches between right-side and below based on terminal width.

## License

This project is currently under active development. A formal open-source license will be added before the first stable release.
