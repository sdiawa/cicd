#!/usr/bin/env groovy
properties([
    parameters([
        string(name: 'git_projet', defaultValue: '', description: 'Nom du project Git | ex: spring-api'),
        string(name: 'git_branch', defaultValue: 'master', description: 'Nom de la branche ou tag Git | ex: fix/big_blue_button'),
        string(name: 'ans_playbook', defaultValue: '', description: 'Nom du fichier playbook Ansible | ex: deploy.yml'),
        string(name: 'ans_inventory', defaultValue: '', description: 'chemin de inventaire ansible | ex: inventories/int'),
        string(name: 'ans_extra', defaultValue: '', description: 'Ajout dextra vars ansible | ex: mavar1=val1 mavar2=val2'),
        string(name: 'ans_limit', defaultValue: '', description: 'Limit Ansible| ex: host1,host2'),
        string(name: 'ans_tags', defaultValue: 'all', description: 'Tag Ansible | ex: tag1,tag2'),
        choice(name: 'ans_verbosity', choices:['-v', '-vv', '-vvv', '-vvvv'], description: 'Selection de la verbosité Ansible, par defaut un -v suffit'),
        credentials(name: 'ans_vault_key', description: 'clé de dechiffrement Vault Ansible', defaultValue: '', credentialTypes: 'org.jenkinsci.plugins.plaincredentials.impl.StringCredentialsImpl', required: false),
        credentials(name: 'ssh_connection_user', description: 'compte utilisater SSh pour que Ansible se connecte aux machines du play', defaultValue: 'USERJENKINS', credentialTypes: 'com.cloudbees.plugins.credentials.impl.UsernamePasswordCredentialsImpl', required: false),
    ])
])


pipeline{
    agent { node { label 'ansible' } }
    options {
        ansicolor('xterm')
    }
    environment {
        ANSIBLE_FORCE_COLOR = 'True'
        ANSIBLE_PARAMIKO_RECORD_HOST_KEYS = 'False'
        ANSIBLE_HOST_KEY_CHECKING = 'False'
    }
    stages {
        stage('Clone Repo') {
            steps {
                checkout([$class: 'GitSCM', branches: [[name: "*/${params.git_branch}"]], doGenerateSubmoduleConfigurations: false, extensions: [], submoduleCfg: [], userRemoteConfigs: [[credentialsId: "USERJENKINS", name: "${params.git_project}", url: "https://github.com/sdiawa/${params.git_project}.git"]]])
            }
        }
        stage('Ansible Prepare') {
            steps {
                sh(label: '',
                script: """ansible --version""")
            }
        }
        stage('Run Playbook'){
            steps{
                //Ansible Vault
                if (params.ans_vault_key) {
                    withCredentials([string(credentialsId: "${params.ans_vault_key}", variable: 'VAULT')]){
                        writeFile(file: ".vault", text: "${VAULT}", encoding: "UTF-8")
                    }
                    env.ANSIBLE_VAULT_PASSWORD_FILE = './.vault'
                }

                //Bastion SSH (023)
                sshagent (credentials: ['ssh_bastion_023']) {
                    //SSH User/pwd
                    if (params.ssh_connection_user) {
                        withCredentials([UsernamePassword(credentialsId: "${params.ssh_connection_user}", passwordVariable: 'ansible_password', usernameVariable: 'ansible_user')]) {
                            sh(label: '',
                            script: """ansible-playbook -c paramiko -i "${params.ans_inventory}" "${params.ans_playbook}" --extra-vars \\"ansible_ssh_pass=${ansible_password} ansible_become_pass=${ansible_password}\\" --extra-vars \\"${params.ans_extra}\\" --tags "${params.ans_tags}" --limit "${params.ans_limit}" -u "${ansible_user}" --ssh-common-args "-o \\"ProxyCommand ssh -o StrictHostKeyChecking=no -o UserKnownHostsFile=/dev/null -W %h:%p USERJENKINSDEPLY@host \\"" "${params.ans_verbosity}" """ ) 
                        }
                    }
                }

            }
        }

    }
}

