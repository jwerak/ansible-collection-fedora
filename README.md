# Configure Fedora laptop

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
ansible-navigator run --ee false ./main.yml
```
