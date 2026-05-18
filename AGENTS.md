# AGENTS.md

Guidance for AI/code agents and contributors working on this role.

## Scope

- Repository: `githubixx.kubernetes_controller`
- Primary concerns: control-plane provisioning reliability, Molecule test quality, lint-clean Ansible/Jinja.

## Working Style

- Keep changes **small and commit-sized** (one test/theme per change where possible).
- Prefer root-cause fixes over suppressions.
- Do not add unrelated refactors while addressing lint/test failures.

## Molecule Expectations

- Scenario sequence must include idempotence and verify:
  - `prepare`
  - `converge`
  - `idempotence`
  - `verify`
- Verify coverage is expected for:
  - control-plane service health
  - listener ports
  - API health endpoints
  - artifact integrity (binaries/certs/kubeconfigs)
  - RBAC wiring
  - admin kubeconfig functional access
  - dependency readiness before/after converge

## Ansible Lint Rules to Respect

- Avoid `systemctl` via `command`/`shell` for state checks.
  - Use `ansible.builtin.service_facts` + `ansible.builtin.assert`.
- If `shell` uses pipes, enable pipefail:
  - `set -o pipefail && ...`
  - `args.executable: /bin/bash`
- Avoid `curl` in tasks where `uri` is appropriate.
  - Use `ansible.builtin.uri` for HTTP health checks.

## Facts Access (Important)

- Injected top-level facts are deprecated; do not rely on `ansible_<iface>` or `ansible_default_ipv4` as top-level vars.
- Use `ansible_facts` dictionary access with fallbacks, e.g.:
  - `ansible_facts.get(interface, {}).get('ipv4', {}).get('address', ansible_facts.get('default_ipv4', {}).get('address'))`
- When accessing other hosts, use:
  - `hostvars[host].ansible_facts...`

## Jinja Templating Pitfalls

- Be careful with folded scalars (`>-`) and multiline Jinja: they can inject whitespace/newlines into CLI args.
- For single-line output values used in URLs/flags, use whitespace control:
  - `{%- ... -%}` and `{{- ... -}}`
- Keep long expressions readable, but ensure rendered output stays whitespace-safe.

## Verify Playbook Conventions

- `verify.yml` should validate system state and API behavior without mutating cluster state.
- For robustness in Molecule, prefer local fallback vars when role defaults may not be in scope.
- Keep task names explicit and failure messages signal-rich via assertions.

## Documentation Sync

- If runtime templates/defaults change (especially fact access style), update `README.md` examples in the same change.
- Keep changelog entries concise and grouped by update type.

## Quick Pre-merge Checklist

- `ansible-lint` clean for changed files.
- No deprecated injected-facts usage introduced.
- No shell pipe without pipefail.
- Molecule scenario still includes idempotence and verify.
- New/changed verify tasks are deterministic across supported test VMs.
