# Ansible Bootstrap Role

Ansible role to bootstrap a fresh system to accept subsequent Ansible plays.

This is a minimal role that installs a (minimal) version of `python`,
to ensure that subsequent plays may use Ansible's typical `python`-based functionality.

## Requirements

This role requires (on your control node)
 - a minimal version (> 1.2) of Ansible

## Dependencies

None.

## Role Variables

* `bootstrap_python_verison` (default: `'python3'`)

  Select the version of python to install.
  Assumes `python3`, but could set to `python` if you need old-school.  Further assumes that you have your [ansible configuration for interpreter discovery](https://docs.ansible.com/ansible/latest/reference_appendices/interpreter_discovery.html) set, or you are happy with the defaults.

* `bootstrap_python_auto_fallback` (default: `true`)

  If true, auto-fallback to just `python[2]` on older distributions where, there is no trivial (via `apt|yum|pkg`) installation of minimal version of python required by ansible.  This currently affects on the following distributions, where this role will install `python` (aka `python2`) instead:

    - Centos 6
    - Debian Jessie (Debian 8)
    - Ubuntu Trusty Tahr (Ubuntu 14.04)

* `bootstrap_gather_facts` (default: `'all'`)

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

## SSH Host Key Caching

Because this playbook is intended to be run on a fresh system, typically you will also need to accept or cache ssh keys for the new systems.  For that, see my [`eengstrom.known_hosts` role](https://github.com/eengstrom/ansible-role-known-hosts).

## Testing

Testing of this role uses [`molecule`](https://molecule.readthedocs.io/en/latest/index.html) and it's docker driver.  To test, I use [`pipenv`](https://pipenv.readthedocs.io/en/latest/) for a local python environment, something like this:

    pipenv install docker molecule
    pipenv shell
    molecule test
    exit

CAVEAT: I do not have testing setup for FreeBSD, largely because using docker on Linux to spin up FreeBSD containers is not really easy.  I'll have to move to VirtualBox testing at some point.

## Author Information

- Eric Engstrom

... with thanks to Willem de Groot, who wrote the [GitHub Gist](https://gist.github.com/gwillem/4ba393dceb55e5ae276a87300f6b8e6f) upon which this is based.

## License

[BSD 3-Clause "New" or "Revised" License](https://spdx.org/licenses/BSD-3-Clause.html)
