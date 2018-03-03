# deploy-dev-environment

This project sets up the following services on a given host:

- openvpn - for securely accessing the internet and developing on the go

## How to use

1. Make a new user with sudo on the target machine called 'deployer', with the password from the vault
  - `sudo useradd deployer && sudo usermod -aG sudo deployer`
