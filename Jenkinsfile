#!/usr/bin/env groovy
properties([
    parameters([
        string(name: 'git_project', defaultValue: '', description: 'Nom du project Git | ex: spring-api'),
        string(name: 'git_branch', defaultValue: 'master', description: 'Nom de la branche ou tag Git | ex: fix/big_blue_button'),
        string(name: 'ans_playbook', defaultValue: '', description: 'Nom du fichier playbook Ansible | ex: deploy.yml'),
        string(name: 'ans_inventory', defaultValue: '', description: 'chemin de inventaire ansible | ex: inventories/int'),
        string(name: 'ans_extra', defaultValue: '', description: 'Ajout dextra vars ansible | ex: mavar1=val1 mavar2=val2'),
        string(name: 'ans_limit', defaultValue: '', description: 'Limit Ansible| ex: host1,host2'),
        string(name: 'ans_tags', defaultValue: 'all', description: 'Tag Ansible | ex: tag1,tag2'),
        choice(name: 'ans_verbosity', choices:['-v', '-vv', '-vvv', '-vvvv'], description: 'Selection de la verbosité Ansible, par defaut un -v suffit'),
        credentials(name: 'ans_vault_key', description: 'clé de dechiffrement Vault Ansible', defaultValue: '', credentialTypes: 'org.jenkinsci.plugins.plaincredentials.impl.StringCredentialsImpl', required: false),
        credentials(name: 'ssh_connection_user', description: 'compte utilisater SSh pour que Ansible se connecte aux machines du play', defaultValue: 'sdiawa', credentialTypes: 'com.cloudbees.plugins.credentials.impl.UsernamePasswordCredentialsImpl', required: false),
    ])
])
   

pipeline{
    agent any
    options {
        ansiColor('xterm')
    }
    environment {
        ANSIBLE_FORCE_COLOR = 'True'
        ANSIBLE_PARAMIKO_RECORD_HOST_KEYS = 'False'
        ANSIBLE_HOST_KEY_CHECKING = 'False'
    }
    stages {
        stage('Clone Repo') {
            steps {
                checkout([$class: 'GitSCM', branches: [[name: "*/${params.git_branch}"]], doGenerateSubmoduleConfigurations: false, extensions: [], submoduleCfg: [], userRemoteConfigs: [[credentialsId: "sdiawa", name: "${params.git_project}", url: "https://github.com/sdiawa/${params.git_project}.git"]]])
            }
        }
   }
}