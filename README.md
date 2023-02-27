# Configure Fedora laptop

## Setup

Install dependencies

`ansible-galaxy collection install -r ./requirements.yml -p ./collections`

## Run
`ansible-navigator run --ee false ./main.yml`
