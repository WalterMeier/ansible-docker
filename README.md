# ansible in Docker

`Dockerfile` and instructions for running `ansible` in `docker`.

Mainly used on Windows, to work around `ansible` not working natively on it.

# Getting started

## SSH private key

Copy your private SSH key to this repository, so that it can be copied to the image during `docker build`.

_Alternatively,_ if you won't be using an SSH key when communicating via `ansible`,
just generate an empty key here, with the following command. 
(It's needed so that `docker build` doesn't fail)

```shell
echo "" > id_rsa
```

> `id_rsa` is already in `.gitignore`

## Docker image build

After copying the key, run the following to build the `docker` image.

```shell
docker build -t ansible .
```

Or alternatively, you can specify which `ansible` version you want during build.

```shell
docker build --build-arg ANSIBLE_VERSION=2.7.0 -t ansible .
```

The resulting image will be used by all the other scripts, described in the **Usage** section.

# Usage

## exec

The `exec` directory contains the executable wrapper script(-s),
that let you use your newly created image in a way, similar to
how you would use `ansible <arg>` commands if you had `ansible` installed locally.

Just add this directory to your `PATH`, and start using them.

The following is an explanation of how these scripts work.

## ansible-playbook &lt;args&gt; | ansible-galaxy &lt;args&gt;

Execute `ansible-playbook` or `ansible-galaxy` commands in your current working directory.
```
cd /path/to/repo/with/ansible/playbooks
ansible-playbook --version
ansible-playbook -u user1 site.yml
ansible-galaxy install -r requirements.yml
# etc
```
> In a nutshell, these scripts just mount the current directory
to the `ansible` container and run the `ansible-playbook <args>`
or `ansible-galaxy <args>` commands against it.

# FAQ
## Why not just install `ansible` locally?
The main reason is Windows. 
`ansible` currently doesn't have Windows support, but this solution 
can be run on any system as long is it has `docker`, including Windows.

# Known issues
* As one might expect, this implementation of `ansible` is more limited
than the standard way of installing on a native linux machine.
It has the following known limitation:
    * It assumes that `ansible.cfg` will only exist in the current working directory
    * The scripts in `exec` dir run the `ansible` container with the `--rm` option.
    Meaning that you'll need to save your roles in the current work dir,
    otherwise they will be saved inside the container and lost after the container is removed. 
    `ansible.cfg` line that can be used for this purpose: `roles_path = ./roles`