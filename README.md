# deploy-dev-environment

This project sets up the following services on a given host:

- openvpn - for securely accessing the internet and developing on the go

## How to use


1. [Preparation](#Preparation)
1. [OpenVPN](#OpenVPN)
1. [Jenkins](#Jenkins)

## Preparation
1. [Install ansible on your local machine.](http://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html)

1. Make a new user with sudo on the target machine called 'deployer', with the password from the vault.
    ```
    sudo useradd deployer && sudo usermod -aG sudo deployer
    ```
1. Create a `.yml` file containing the following structure in the `group_vars/all` folder of this project.
    ```
    user:
      name: deployer

    ansible_ssh_pass: <password for the deployer user you created earlier>
    ansible_become_pass: <password for the deployer user you created earlier>
    ```
  Note: it is recommended to use [ansible-vault]("http://docs.ansible.com/ansible/latest/user_guide/playbooks_vault.html") to encrypt any files you create in this project that contain sensitive information.

---

## OpenVPN
1. Create a `.hosts` file referencing the target machine in the `openvpn` group.
    ```
    [openvpn]
    <hostname or IP address of target machine>
    ```
1. Create a `.yml` file with the following structure in `group_vars/openvpn` folder of this project, again it is recommended to use `ansible-vault`
    ```
    openvpn:
      ca_cert_password: <a strong password that will encrypt the CA certificate>
      common_name: <a unique name identifying the server>
    ```
1. You're all set! Now just run the `common.yml` playbook first, which installs some packages needed for running & debugging ansible on the target machine.
    ```
    ansible-playbook -i <path to .hosts file created earlier> common.yml
    ```
    Then run the `openvpn.yml` playbook to install OpenVPN.
    ```
    ansible-playbook -i <path to .hosts file created earlier> openvpn.yml
    ```

### Getting a signed client certificate:
1. [Install easyrsa3 by downloading and extracting the .tgz or .zip from here.](https://github.com/OpenVPN/easy-rsa/releases)

1. Somewhere inside the extracted directory structure is a shel script simply called `easyrsa`, find it and run
    ```
    ./easyrsa init-pki
    ```
    then
    ```
    ./easyrsa --pki-dir=<path to pki folder generated by previous command> gen-req <a unique name identifying this client>
    ```
    This may ask you for a password, it is strongly suggested that you password protect your certificate.

1. The previous command generated a `.key` file and a `.req` file, so now run
    ```
    ansible-playbook -i <path to .hosts file created earlier> sign_openvpn_client_reqs.yml -e "local_req_file=<path to .req file>"
    ```
  Done! You now have:
    - a signed client certificate `<unique client name>.crt`
    - the key file for your client cert `<path to your pki folder>/private/<unique client name>.key`
    - the certificate authority certificate `ca.crt`
    - and the static key `ta.key`


### What next?
Depending on your operating system and personal preference, you'll want to **install an OpenVPN client**.
However, to set any of them up you'll always need:
 - the three files generated in the client certficate steps
 - a client config file

I've provided a sample config file in this repo, `sample-client-config.conf`, to use with Ubuntu and Android, I can't yet attest to how well it works on Windows.
The entries you will _need_ to replace are:
  - remote _my.openvpn.server_ 1194
  - ca _/path/to/ca/cert/file/ca.crt_
  - cert _/path/to/client/cert/file/client.crt_
  - key _/path/to/client/cert/key/file/client.key_
  - tls-auth _/path/to/ta.key_ 1

---

## Jenkins
1. Create a `.hosts` file referencing the target machine in the `jenkins` group.
    ```
    [jenkins]
    <hostname or IP address of target machine>
    ```
1. Create a `.yml` file with the following structure in `group_vars/jenkins` folder of this project, again it is recommended to use `ansible-vault`
    ```
    jenkins_vault:
      create_users_args:
        username: <username you will use to login to Jenkins>
        password: <strong password you will use to login to Jenkins>
    ```
1. You're all set! Now just run the `common.yml` playbook first, which installs some packages needed for running & debugging ansible on the target machine.
    ```
    ansible-playbook -i <path to .hosts file created earlier> common.yml
    ```
    Then run the `jenkins.yml` playbook to install Jenkins.
    ```
    ansible-playbook -i <path to .hosts file created earlier> jenkins.yml
    ```
