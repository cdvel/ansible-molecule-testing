# Testing Ansible roles using Molecule

Molecule: open source infrastructure testing tool written in Python that can provision and test Ansible roles.

In this repo find the basic steps to test you ansible role (via example) and how to integrate with Github Actions.

## Steps

1. Create a basic playbook: Installs apache and sets index.html

2. Run `molecule init scenario`

3. Update `molecule/default/converge.yml`

4. Run molecule converge; molecule login; molecule destroy

5. Update molecule.yaml from
```yaml
  ---
  dependency:
    name: galaxy
  driver:
    name: docker
  platforms:
    - name: instance
      image: docker.io/pycontribs/centos:7
      pre_build_image: true
  provisioner:
    name: ansible
  verifier:
    name: ansible

```
to current file = molecule.yaml

6. modify verify.yml to check serving  page ok

> NOTE: molecule test = converge + verify but slower,  also check `Verifier` in molecule docs

7. Add lint via yamllint => ansible lint + yamlint

Add block to molecule.yml to include lint into molecule configuration: fails if lint not ok

```yaml
lint: |
  set -e
  yamllint .
  ansible-lint
```

8. Add a github Action to run all tests in multiple distros. Also works for Pull requests:

from `.github/workflows/ci.yml`

```yaml

---
name: CI
'on':
  pull_request:
  push:
    branches:
      - master
jobs:

  test:
    name: Moleculex
    runs-on: ubuntu-latest
    strategy:
      matrix:
        distro:
          - centos8
          - debian10
    steps:
      - name: Check out the codebase.
        uses: actions/checkout@v2

      - name: Set up Python 3.
        uses: actions/setup-python@v2
        with:
          python-version: '3.x'

      - name: Install test dependencies.
        run: pip3 install molecule docker yamllint ansible-lint

      - name: Run Molecule tests.
        run: molecule test
        env:
          PY_COLORS: '1'
          ANSIBLE_FORCE_COLOR: '1'
          MOLECULE_DISTRO: ${{ matrix.distro }}⏎
```
