# sh.unless

Idempotent shell commands for Ansible.

Ansible's `shell` module runs every time, which breaks `--check` mode and can cause failures when commands aren't safe to repeat — `mkdir` on an existing directory, `ln` creating nested links. This module fixes the problem.

This module runs the script in `unless_state` and only runs `unless_then` if the check fails. Every Ansible module is a specialized check/fix pair — this makes the pattern explicit and can in principle replace any module. Supports check mode and non-shell interpreters (ruby, python).

```yaml
- sh.unless:
    state:  directory-link
    link:   /app/current
    target: /app/v1.0
    unless_state: test -L "${link}" && test "$(readlink "${link}")" = "${target}"
    unless_then:  ln -sfn "${target}" "${link}"
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
    unless_state: test -L "/tmp/sh-unless-test-link" && test "$(readlink "/tmp/sh-unless-test-link")" = "/tmp/sh-unless-test-target"
    unless_state_rc: 0
    unless_then_log: + ln -sfn /tmp/sh-unless-test-target /tmp/sh-unless-test-link

TASK [ensure directory symlink (second run - idempotent)]
ok: [localhost] =>
    changed: false
    state: directory-link
    unless_state: test -L "/tmp/sh-unless-test-link" && test "$(readlink "/tmp/sh-unless-test-link")" = "/tmp/sh-unless-test-target"
    unless_state_rc: 0
```

## Install

Copy `library/sh.unless` to your Ansible module path, or add this repo to your playbook directory.

## License

[BSD-0](https://spdx.org/licenses/0BSD.html)
