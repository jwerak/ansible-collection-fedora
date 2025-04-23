# Ansible Collection jwerak.fedora_setup

## Setup

Install dependencies

```bash
sudo dnf install -y python3-pip
pip install ansible ansible-navigator
ansible-galaxy collection install -r ./requirements.yml -p ./collections
```

## Run

The command below expects paswordless sudo.

```bash
ansible-galaxy collection install jwerak.fedora
ansible-navigator run --ee false ./main.yml
```

## Setup local test environment

### Preparation

Generate password - to be used in kickstart.

```bash
ansible -m debug -a msg="{{ 'my password' | password_hash('sha512') }}" localhost | awk -F\" '/msg/ {print $4}'
```

other:

- Download [Fedora ISO](https://fedoraproject.org/workstation/download)
-

### Create Fedora VM

```bash
ISO_PATH=/path/to/iso

    # --location http://download.fedoraproject.org/pub/fedora/linux/releases/41/Server/x86_64/os/
    # --location /home/jveverka/Downloads/software/OS/Fedora-Workstation-Live-x86_64-40-1.14.iso \

sudo virt-install \
    --name test-fedora \
    --memory 2048 \
    --vcpus 2 \
    --location http://download.fedoraproject.org/pub/fedora/linux/releases/41/Everything/x86_64/os/ \
    --disk size=20 \
    --os-variant=fedora-unknown \
    --network default \
    --initrd-inject=`pwd`/ks.cfg \
    --extra-args="inst.ks=file:/ks.cfg"
```

### Run Test

There is a playbook for testing against fedora, e.g in VM.

```bash
ansible-navigator run ./playbooks/site.yml -i inventory \
--eev `pwd`:/usr/share/ansible/collections/ansible_collections/jwerak/fedora:Z
```
