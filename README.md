# Ansible Bootstrap Role

Ansible role to bootstrap a fresh system to accept subsequent Ansible plays.  Specifically:
 - cache `ssh` host key(s) into `known_hosts` file.
 - install a minimal version of `python`.

This is a minimal role to ensure that subsequent plays may use Ansible's typical `python`-based functionality.

## Requirements

This role requires (on your control node)
 - a minimal version (> 1.2) of Ansible
 - `dnspython` python library (only for `known_hosts` caching)

## Dependencies

None.

## Role Variables

* `bootstrap_ssh_key_auto_cache` (default: `true`)

  Should this role automatically cache the `ssh` host keys of new hosts?
  This is executed on `localhost` using the `ssh-keyscan` command to fetch the key,
  and the `known_hosts` module to cache it.

* `bootstrap_ssh_key_hash_hostname` (default: `false`)

  If true, hostname will be hashed in the `known_hosts` file.

* `bootstrap_ssh_key_cache_ipv4` (default: `true`)

  If true, cache the IP[V4] address(es) of the host as well.

* `bootstrap_ssh_key_cache_blindly` (default: `false`)

  If true, update cached keys on EVERY invocation.
  **SECURITY NOTE**: this runs a higher risk of MITM attacks,
  as the any cached host keys are replaced blindly every time.
  However, you might want to turn this on in certain situations,
  for example if you know the host keys have changed recently.
  Can be turned on either by changing this variable, or temporarily
  at invocation by providing the tag/option `-t cache_blindly`.

* `bootstrap_ssh_key_type` (default: `"ecdsa"`)

  What ssh key type should we fetch?  Typical options include:
    - `dsa`  (**NOTE: this is deprecated by most SSH installations as insecure**)
    - `rsa`
    - `ecdsa` (default)
    - `ed25519`

* `bootstrap_ssh_key_hosts_file` (default: `"~/.ssh/known_hosts"`)

  Location of the file into which the host key will be stored.  If unset (the default), the location is the default of the ansible `known_hosts` module, which is `~/.ssh/known_hosts`.

* `bootstrap_gather_facts` (default: `'all'`)

  Specifies the subset of facts to be gathered by the [`setup` module](https://docs.ansible.com/ansible/latest/modules/setup_module.html) *after* the bootstrap installation of python.  May be a single item (string) or a list of string values. See documentation for the `gather_subset` parameter of the `setup` module for a complete list of possible values.

  Defaults to `'all'`, which is the `setup` module default.

  May be set to `false` (or an empty list or an empty string) to disable the gathering of facts post bootstrap.

* `bootstrap_python_verison` (default: `'python3'`)

  Select the version of python to install.
  Assumes `python3`, but could set to `python` if you need old-school.  Further assumes that you have your [ansible configuration for interpreter discovery](https://docs.ansible.com/ansible/latest/reference_appendices/interpreter_discovery.html) set, or you are happy with the defaults.

* `bootstrap_python_auto_fallback` (default: `true`)

If true, auto-fallback to just `python[2]` on older distributions where, there is no trivial (via `apt|yum|pkg`) installation of minimal version of python required by ansible.  This currently affects on the following distributions, where this role will install `python` (aka `python2`) instead:

    - Centos 6
    - Debian Jessie (Debian 8)
    - Ubuntu Trusty Tahr (Ubuntu 14.04)

## Example Playbook

There is very little to this role, however, because it's primary task is to install `python`, any playbook that makes use of it must disable the gathering of facts before it runs; for example:

    - hosts: all
      gather_facts: false
      become: true
      roles:
        - eengstrom.bootstrap

After bootstrap, the module will attempt to re-gather facts not yet in Ansible's brain per the variable `bootstrap_gather_facts`.

## SSH Host Key Caching Caveats

Because this playbook is intended to be run on a fresh system, typically it will be run with `gather_facts: false`; see above.  Because of this expected usage scenario, the caching of SSH host keys (see `bootstrap_ssh_key_auto_cache`) will use the value of `inventory_hostname` rather than `ansible_hostname`, since the latter requires facts to have been gathered, which will not have yet occurred.  This has the unfortunate *requirement* and assumption that your systems' hostnames and inventory names match.  If this is not the case, you may have issues.  If someone has a better way to do "the right thing", please submit a PR.

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
