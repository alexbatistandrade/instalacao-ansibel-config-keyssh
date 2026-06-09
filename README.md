# Handbook — Instalação e Configuração do Ansible no Ubuntu

Este repositório contém um handbook em formato Markdown com o passo a passo para instalação, configuração inicial e validação do Ansible em servidores Ubuntu.

O material apresenta a instalação do Ansible via PPA oficial, a preparação do arquivo `ansible.cfg`, a organização da estrutura de diretórios em `/etc/ansible`, a configuração do inventário, a criação da chave SSH no Ansible Control Node e a aplicação de uma playbook responsável por compartilhar a chave pública SSH com os hosts gerenciados.

O objetivo deste documento é padronizar o ambiente Ansible, facilitar a comunicação entre o Control Node e os hosts, e permitir a execução de automações sem necessidade de autenticação por senha após a configuração inicial.

## Conteúdo abordado

* Instalação do Ansible no Ubuntu via PPA
* Configuração do arquivo `ansible.cfg`
* Estrutura padrão de diretórios em `/etc/ansible`
* Teste de comunicação com os hosts usando `ansible ping`
* Criação da chave SSH no Ansible Control Node
* Criação da playbook `padraohosts.yml`
* Criação da role `common`
* Configuração da task para adicionar a chave pública no `authorized_keys`
* Execução inicial com senha
* Validação do acesso SSH sem senha

## Objetivo

Padronizar a instalação e a configuração básica do Ansible em ambientes Ubuntu, garantindo que o Control Node consiga se comunicar com os hosts gerenciados de forma segura e automatizada.


# Handbook — Instalação e Configuração do Ansible no Ubuntu

## Installing Ansible on Ubuntu

O Ubuntu disponibiliza pacotes do Ansible por meio de um **PPA (Personal Package Archive)**, que geralmente contém versões mais recentes do que os repositórios padrão do sistema.

Ubuntu builds are available in a PPA here.

### Configurar o PPA e instalar o Ansible

Execute os comandos abaixo no servidor que será utilizado como **Ansible Control Node**:

```bash
sudo apt update
sudo apt install software-properties-common
sudo add-apt-repository --yes --update ppa:ansible/ansible
sudo apt install ansible
```

Referência oficial:

```text
https://docs.ansible.com/projects/ansible/latest/installation_guide/installation_distros.html#installing-ansible-on-ubuntu
```

---

## Preparando a configuração

A configuração principal do Ansible será realizada no diretório:

```bash
/etc/ansible
```

Antes de criar o novo arquivo de configuração, mova o arquivo antigo `ansible.cfg`, caso ele já exista:

```bash
mv ansible.cfg ansible.cfg-old
```

Em seguida, crie um novo arquivo `ansible.cfg`:

```bash
vim ansible.cfg
```

Adicione o conteúdo abaixo ao arquivo:

```ini
[defaults]
inventory = ./inventory/hosts.yaml
roles_path = ./roles:~/.ansible/roles:/usr/share/ansible/roles:/etc/ansible/roles
collections_path = ./collections:~/.ansible/collections:/usr/share/ansible/collections
host_key_checking = False
retry_files_enabled = False
stdout_callback = ansible.builtin.default
callback_result_format = yaml
deprecation_warnings = True
timeout = 30

[ssh_connection]
pipelining = True
control_path = ~/.ansible/cp/%%h-%%p-%%r
```

Exemplo de diretório de trabalho:

```bash
root@vml-maximoff:/etc/ansible#
```

---

## Estrutura das pastas para a configuração

A estrutura esperada para organização dos arquivos do Ansible deve seguir o modelo abaixo:

```text
/etc/ansible/
├── ansible.cfg
├── inventory/
│   └── hosts.yaml
├── playbooks/
│   └── padraohosts.yml
├── roles/
│   └── common/
│       └── tasks/
│           └── main.yml
├── collections/
│   └── ansible_collections/
│       └── namespace/
│           └── collection_name/
├── group_vars/
│   └── all.yml
└── host_vars/
    └── "servidor01".yml
```

---

## Teste de comunicação com os hosts

Após concluir a inclusão dos arquivos e adicionar os servidores no arquivo `hosts.yaml`, realize o teste de comunicação com os hosts.

Execute o comando abaixo a partir do diretório `/etc/ansible`:

```bash
ansible all -m ping --ask-pass
```

Esse comando realiza um teste de conectividade com todos os hosts definidos no inventário, solicitando a senha SSH do usuário remoto.

---

## Criar a chave diretamente no servidor Ansible

No servidor Ansible Control Node, crie o diretório `.ssh`, ajuste as permissões e gere a chave SSH:

```bash
mkdir -p ~/.ssh
chmod 700 ~/.ssh
```

Gere a chave SSH do tipo `ed25519`:

```bash
ssh-keygen -t ed25519 -C "ansible-control-node" -f ~/.ssh/id_ed25519
```

Ajuste as permissões da chave privada e da chave pública:

```bash
chmod 600 ~/.ssh/id_ed25519
chmod 644 ~/.ssh/id_ed25519.pub
```

---

## Criar a playbook

Crie o arquivo da playbook no caminho abaixo:

```bash
/etc/ansible/playbooks/padraohosts.yml
```

Conteúdo da playbook:

```yaml
- name: Compartilhar chave SSH do control node com os hosts
  hosts: all
  become: true

  roles:
    - common
```

Essa playbook será responsável por aplicar a role `common` em todos os hosts definidos no inventário.

---

## Criar a task common para compartilhar a chave SSH do Control Node Ansible com os hosts

Crie ou edite o arquivo da task principal da role `common`:

```bash
/etc/ansible/roles/common/tasks/main.yml
```

Adicione o conteúdo abaixo:

```yaml
- name: Garantir diretório .ssh do usuário remoto
  ansible.builtin.file:
    path: "/home/{{ ansible_user }}/.ssh"
    state: directory
    owner: "{{ ansible_user }}"
    group: "{{ ansible_user }}"
    mode: "0700"

- name: Adicionar chave pública do control node no authorized_keys
  ansible.builtin.authorized_key:
    user: "{{ ansible_user }}"
    state: present
    key: "{{ lookup('file', '~/.ssh/id_ed25519.pub') }}"
```

Essa task garante que o diretório `.ssh` exista no host remoto para o usuário definido em `ansible_user` e adiciona a chave pública do Ansible Control Node no arquivo `authorized_keys`.

---

## Executar a playbook pela primeira vez usando senha

Na primeira execução, ainda será necessário utilizar senha para acessar os hosts remotos.

Acesse o diretório do Ansible:

```bash
cd /etc/ansible
```

Execute a playbook informando a senha SSH e a senha de privilégio administrativo:

```bash
ansible-playbook playbooks/padraohosts.yml --ask-pass --ask-become-pass
```

Após a execução com sucesso, a chave pública do Ansible Control Node será compartilhada com os hosts remotos.

---

## Testar o acesso sem senha

Depois da aplicação da playbook, teste novamente a comunicação com os hosts:

```bash
ansible all -m ping
```

Se a configuração estiver correta, o Ansible deverá conseguir se conectar aos hosts sem solicitar senha SSH.
