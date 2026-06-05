# sh.unless

Ansible's shell module lacks idempotence guards — commands aren't safe to
repeat and `--check` mode can break your system. This module adds them:
run a check and only act if it fails.

Every Ansible module is a specialized check/fix pair — sh.unless just
makes the pattern reusable. Supports `--check` mode and non-shell
interpreters (ruby, python).

```yaml
- sh.unless:
    state:      directory-link
    file:       /app/current
    linktarget: /app/releases/initial
    unless_state: test -L "${file}" && test -d "${file}/"
    unless_then:  rm -rf "${file}" && ln -s "${linktarget}" "${file}"
```

## Dependencies

- POSIX `/bin/sh`
- `sed`, `tr` (BSD compatible)
- `envsubst` (gettext)

## Test

```sh
ansible-playbook -v test.yml
```

```yaml
TASK [ensure directory symlink (first run - creates link)]
changed: [localhost] =>
    changed: true
    state: directory-link
    unless_state: test -L "/tmp/sh-unless-test-file" && test -d "/tmp/sh-unless-test-file/"
    unless_state_rc: 1
    unless_then_log: |-
        + rm -rf /tmp/sh-unless-test-file
        + ln -s /tmp/sh-unless-test-target /tmp/sh-unless-test-file

TASK [ensure directory symlink (second run - idempotent)]
ok: [localhost] =>
    changed: false
    state: directory-link
    unless_state: test -L "/tmp/sh-unless-test-file" && test -d "/tmp/sh-unless-test-file/"
    unless_state_rc: 0
```

## Install

Copy `library/sh.unless` to your Ansible module path, or add this repo to your playbook directory.

