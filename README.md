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
