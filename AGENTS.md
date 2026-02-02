# AGENTS.md

## Overview
This repository is a collection of Ansible roles and configuration files used to provision Debian workstations.  The **agents** that work on this repo (including this model) rely on a small set of tooling and conventions to keep the code clean, reproducible, and easy to maintain.

---

## 1. Build / Lint / Test Commands

| Command | Purpose | Example |
|---------|---------|---------|
| `ansible-lint` | Lint all playbooks, roles and tasks for Ansible best‑practice violations. | `ansible-lint .` |
| `yamllint` | Lint YAML syntax and indentation. | `yamllint .` |
| `ansible-playbook --syntax-check <playbook>.yml` | Verify that a playbook is syntactically valid without executing. | `ansible-playbook --syntax-check site.yml` |
| `ansible-playbook -i <inventory> <playbook>.yml --check` | Dry‑run a playbook against an inventory (no changes applied). | `ansible-playbook -i inventory playbook.yml --check` |
| `ansible-test sanity --role <role>` | Sanity test a role in isolation. (Requires `ansible-test` package.) | `ansible-test sanity --role roles/dev` |
| `pre-commit run --all-files` | Run all configured pre‑commit hooks (linting, formatting, etc.). | `pre-commit run --all-files` |

### Running a single test
If you only want to test a specific role or playbook:

```bash
# Validate a single playbook
ansible-playbook -i inventory playbooks/dev.yml --syntax-check

# Dry‑run a single playbook
ansible-playbook -i inventory playbooks/dev.yml --check --diff

# Run a single role sanity test
ansible-test sanity --role roles/dev
```

---

## 2. Code Style Guidelines

| Category | Guideline | Rationale |
|----------|------------|-----------|
| **YAML Formatting** | Use 2‑space indentation, no tabs. | Consistent whitespace is essential for YAML parsing. |
| **Quotes** | Prefer double quotes for strings that may contain spaces or special characters. | Avoids accidental YAML parsing errors. |
| **Variable Naming** | Use `snake_case` for all Ansible variables (`my_var`). | Matches Ansible conventions and improves readability. |
| **Module Usage** | Prefer the `ansible.builtin` namespace (e.g., `ansible.builtin.shell`). | Avoids implicit imports and makes module origin explicit. |
| **File Names** | Keep file names all‑lowercase with dashes (`my-role.yml`). | Easier to reference and search. |
| **Error Handling** | Register task results (`register: result`) and use `failed_when` or `changed_when` as needed. | Makes failures explicit and easier to debug. |
| **Idempotency** | Ensure every task can be run multiple times without side effects. | Core Ansible principle. |
| **Idempotent Module Selection** | Use modules that are idempotent (`file`, `copy`, `yum`, `apt`, etc.) over raw shell when possible. | Reduces unintended side effects. |
| **Commenting** | Use a single line comment with `#` and a space. Avoid trailing comments on the same line as code. | Keeps diff clean. |
| **Documentation** | Include a short `README.md` in each role describing purpose, required variables, and examples. | Helps users understand and use the role. |
| **Testing** | Provide a `tests/` directory with sanity test cases and example inventories. | Encourages early detection of regressions. |
| **Security** | Never hard‑code secrets in YAML; use Ansible Vault or external secrets store. | Protects sensitive data. |
| **Linting** | Run `ansible-lint` before committing. | Catches common mistakes early. |

---

## 3. Ansible‑Specific Guidelines

- **Handlers**: Place handlers in `handlers/main.yml` and name them descriptively. Example: `restart apache`. Handlers should be idempotent and only run if notified.
- **Templates**: Use Jinja2 templates for configuration files that require dynamic content. Keep template logic minimal – prefer variables and conditional includes.
- **Includes vs Imports**: Use `import_tasks` for static inclusion and `include_tasks` for dynamic inclusion. Prefer static imports where possible for better linting.
- **Role Dependencies**: Declare dependencies in `meta/main.yml`. This ensures required roles are installed and ordered.
- **Variables**: Follow a clear precedence order: `vars`, `defaults`, `vars/main.yml`, `defaults/main.yml`. Keep defaults in `defaults/main.yml` and override only when necessary.
- **Avoid `shell`**: Prefer modules like `command` or `copy`. Use `shell` only when necessary.

---

## 4. Error Handling and Logging

- **Use `register`** to capture command output: `- name: List files
    ansible.builtin.shell: ls
    register: ls_output`.
- **Check `changed`** status: `failed_when: ls_output.failed` or `changed_when: ls_output.changed`.
- **Output**: Use `debug: msg="{{ ls_output.stdout }}"` for visibility.
- **Exit on Failure**: Add `any_errors_fatal: true` at the play level to stop on the first failure.
- **Retries**: Use `retries` and `delay` for transient operations (e.g., waiting for a service). Example: `register: result; retries: 5; delay: 10`.

---

## 5. Testing Strategy

- **Sanity Tests**: Use `ansible-test sanity` to run quick checks for role structure.
- **Integration Tests**: Optionally, set up Molecule to test roles against a Vagrant or Docker provider. A minimal `molecule.yml` can be added to each role.
- **Unit Tests**: Keep `tests/test_role.py` with pytest if Python code is involved. Use `ansible-lint` and `yamllint` in the CI pipeline.
- **Continuous Integration**: Configure GitHub Actions (or other CI) to run `ansible-lint` and `yamllint` on each pull request.
- **Running a Single Test**: Use the command examples from section 1 to focus on a single playbook or role.

---

## 6. Cursor / Copilot Rules

> The repository does **not** contain a `.cursor` directory or `.cursorrules` file.
> There are also no `copilot-instructions.md` files present.
> Therefore, no special rules are applied for cursor or copilot.

---

## 7. Suggested Pre‑Commit Hooks

A minimal `.pre-commit-config.yaml` could look like:

```yaml
repos:
  - repo: https://github.com/ansible/ansible-lint
    rev: v5.3.1
    hooks:
      - id: ansible-lint
  - repo: https://github.com/adrienverge/yamllint
    rev: v1.29.0
    hooks:
      - id: yamllint
        args: ["-d", "{extends: relaxed, rules: {line-length: {max: 120}}}"]
  - repo: local
    hooks:
      - id: check-syntax
        name: Check Ansible syntax
        entry: ansible-playbook --syntax-check
        language: script
        types: [yaml]
```

---

## 8. Summary

- Keep YAML tidy, 2‑space indentation, use `ansible.builtin` modules.
- Run `ansible-lint` and `yamllint` before commits.
- Use `ansible-playbook --syntax-check` and `--check` for quick tests.
- Prefer idempotent tasks; always register results and handle errors explicitly.
- Add documentation and tests for new roles to maintain quality.

Happy provisioning!
