# eengstrom.bootstrap

Ansible role to bootstrap a fresh system to accept subsequent Ansible plays.  Specifically, install a minimal version of `python` so that subsequent plays may use Ansible's typical `python`-based functionality.

## Requirements

This role requires a minimal version (> 1.2) of Ansible on your control node.

## Dependencies

None.

## Role Variables

### `bootstrap_gather_facts` (default: `'all'`)

Specifies the subset of facts to be gathered by the [`setup` module](https://docs.ansible.com/ansible/latest/modules/setup_module.html) *after* the bootstrap installation of python.  May be a single item (string) or a list of string values. See documentation for the `gather_subset` parameter of the `setup` module for a complete list of possible values.

Defaults to `'all'`, which is the `setup` module default.

May be set to `false` (or an empty list or an empty string) to disable the gathering of facts post bootstrap.

## Example Playbook

There is very little to this role, however, because it's primary task is to install `python`, any playbook that makes use of it must disable the gathering of facts before it runs; for example:

    - hosts: all
      gather_facts: false
      become: true
      roles:
        - eengstrom.bootstrap

After bootstrap, the module will attempt to re-gather facts not yet in Ansible's brain per the variable `bootstrap_gather_facts`.

## License

[BSD 3-Clause "New" or "Revised" License](https://spdx.org/licenses/BSD-3-Clause.html)

## Testing

Testing of this role uses [`molecule`](https://molecule.readthedocs.io/en/latest/index.html) and it's docker driver.  To test, I use [`pipenv`](https://pipenv.readthedocs.io/en/latest/) for a local python environment, something like this:

    pipenv install docker molecule
    pipenv shell
    molecule test
    exit

## Author Information

- Eric Engstrom

... with thanks to Willem de Groot, who wrote the [GitHub Gist](https://gist.github.com/gwillem/4ba393dceb55e5ae276a87300f6b8e6f) upon which this is based.
