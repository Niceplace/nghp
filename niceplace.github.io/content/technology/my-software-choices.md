---
title: "Ansible and my development enviroment"
type: "post"
description: "What I am using (software, configuration) in my development enviroment"
date: 2017-06-20
publishdate: 2017-05-20
showDate: true
draft: true
tags: [ "technology", "configuration", "ansible", "chef", "puppet", "linux", "ubuntu" ]
categories: ["index", "technology", "development"]
keywords: [ "technology", "development", "index"]
clearReading: false
thumbnailImage: ../images/Mydevelopmentenvironment.png
thumbnailImagePosition: left
autoThumbnailImage: yes
metaAlignment: center

---
This is a short introduction to Ansible and a description of my first project, the automation of the configuration of my development laptop.
<!--more-->

## Link to project

The project discussed in this post can be found [on my github](https://github.com/Niceplace/config)


# Ansible

I love Ansible! I am using it at work to automate the configuration of developer VMs. Compared to other configuration management tools (Chef, Puppet) it was my first choice because of its simplicity. My usage of Ansible is a bit limited since I only apply the configuration to a single computer at a time (its main purpose is to manage the configuration of multiple hosts at once) but I still gained some very valuable knowledge that can be applied in different situations.

## Some vocabulary

Technical terms, buzzwords, foreign concepts! In order to prevent some unnecessary googling and question marks, here is a short list of useful terms related to Ansible and this project that may help explain the context. Most of the terms explained here are related to core concepts which also happen to be the name of the main directories in a playbook's layout but I also included a few other terms.

| Term           | Definition | More information |
|----------------|------------|------------------|
| Playbook       | A playbook represents the desired state of the machine it will be applied on. The written configuration can also be used for documentation purposes.           | [Ansible docs - Playbooks](http://docs.ansible.com/ansible/playbooks.html)                 |
| Modules        | A module is a specific function of Ansible that replicates a well known command and its various parameters. (i.e. apt-get, yum, get_url - similar to curl or wget -, docker, etc.)           | [Ansible docs - Modules](http://docs.ansible.com/ansible/modules.html)                 |
| Roles          | A role contains a list of tasks. A role represents a series of tasks that are combined to configure an element of the playbook.           | [Ansible docs - Roles](http://docs.ansible.com/ansible/playbooks_roles.html)                 |
| Tasks          | A task is the smallest possible operation that can be performed in Ansible. It represents a single command with or without arguments, performed with the help of a module           | [Ansible docs - Tasks](http://docs.ansible.com/ansible/playbooks_intro.html#tasks-list)                 |
| Handlers       | A handler is a task that is triggered by another task (i.e. restart a service after a config change).           | [Ansible docs - Handlers](http://docs.ansible.com/ansible/glossary.html#term-handlers)                 |
| Meta           | The "meta" folder inside a role is used to declare dependencies to other roles.           | [Ansible docs - Meta](http://docs.ansible.com/ansible/playbooks_roles.html#role-dependencies)                 |
| Template       | Templates are made with Jinja2 templating, use these when you want to replace some values inside a file with variable content (i.e. a local settings.xml for maven).           | [Ansible docs - Template](http://docs.ansible.com/ansible/template_module.html)                 |
| Become         | Used in roles / tasks to represent privilege escalation (e.g. sudo).            | [Ansible docs - Become](http://docs.ansible.com/ansible/become.html)                 |
| Check mode     | Check mode is useful to check your playbook for syntax errors. It runs the full playbook except it does not apply any changes, it just checks if the execution would run properly. Note: not all modules support check mode.           | [Ansible docs - Check mode](http://docs.ansible.com/ansible/playbooks_checkmode.html)                 |

## Best practices

To organize the content of my playbook, I followed the best practices for directory layout, as specified in the [Ansible docs - directory layout best practices](http://docs.ansible.com/ansible/playbooks_best_practices.html#directory-layout).

I find that using best practices for directory layout really helps with maintaining the playbook. In most cases, one role represents one piece of software and its configuration so it makes it easy to make fixes and/or add new stuff to install.


## Under the hood

{{< alert info >}}
If you want to check out the documentations for all projects related to ansible, you can read it at [docs.ansible.com](http://docs.ansible.com/).   
There is a container specific project **Ansible Container**, which I have not explored yet and also some information on the **Ansible Tower** project.  
{{< /alert >}}

One of the key points of Ansible is that its installation is very easy and it does not require live agents on all managed nodes (i.e. servers and/or personal computers). Once you have written your playbook, it simply connects via SSH to the node(s) that are to be managed, executes the necessary commands and then exits. This is a plus for me because because using Chef / Foreman to document & automate the configuration of one single laptop is  the equivalent of using a hydraulic jackhammer to grind a few grains of peppercorn for my steak. For those who want to use Ansible to manage configurations at scale, it is worth mentioning that Ansible has been bought by Redhat some time ago and they offer enterprise grade solutions (Ansible Tower) that compete with Chef / Puppet and the likes.

## Installation

{{< alert info >}}
Note : When building a playbook, depending on which [module](http://docs.ansible.com/ansible/modules_intro.html) you decide to use, the installations of other python packages might be necessary.
{{< /alert >}}

Ansible is a small (ish) Python program that is easily installed (in my case, on Ubuntu) with the help of a few commands.
From the [installation instructions](http://docs.ansible.com/ansible/intro_installation.html#latest-releases-via-apt-ubuntu), its pretty straightforward :

``` bash
$ sudo apt-get install software-properties-common
$ sudo apt-add-repository ppa:ansible/ansible
$ sudo apt-get update
$ sudo apt-get install ansible
```
## Using modules

## Validating a playbook

{{< alert info >}}
When running in check node, please note that not all modules support it and may throw errors that will halt the execution. The reason for this is that some technical challenges prevent every single possible command to be executed "just to check the output" without actually performing any operation. The [command](http://docs.ansible.com/ansible/command_module.html#notes) and [shell](http://docs.ansible.com/ansible/shell_module.html) modules are examples of this. In that case you might want to [ignore errors or skip the execution](http://docs.ansible.com/ansible/playbooks_checkmode.html#information-about-check-mode-in-variables)
{{< /alert >}}

When a playbook is newly created or updated, it is necessary to perform some basic validation on it to ensure that errors are avoided. The first validation that I do is run the playbook, using the ["check mode"](http://docs.ansible.com/ansible/playbooks_checkmode.html).

{{< alert info >}}
Replace $HOSTS_INVENTORY_FILE with the (relative|absolute) path of your inventory file if you have one. If you don't need this you can remove the "-i" option.  
Replace $PLAYBOOK_PATH with the absolute (relative|absolute) in the YAML playbook file.
{{< /alert >}}

``` bash
$ ansible-playbook -i $HOSTS_INVENTORY_FILE $PLAYBOOK_PATH --check
```

In my case, I use this comand :

{{< alert info >}}
I use privilege escalation and the --ask-sudo-pass tells ansible to prompt me once for the sudo password before running the playbok.  
It's not the only way to do this, you can also specify the sudo password directly with the --extra-vars flag `--extra-vars "ansible_sudo_pass=$PASSWORD"` (replace $PASSWORD with your sudo password)
{{< /alert >}}

``` bash
$ ansible-playbook -i hosts --ask-sudo-pass devenv.yml --check
```

Bear in mind that using the check mode does not cover all possible cases, but it helps clean up a lot of easy mistakes (quoting variables, errors in YAML parsing). The rest of the testing can be done by executing the playbook on a separate vm that mimics the current environment.

## Running the playbook

Once you have passed validation and you are ready to execute the playbook, use the `ansible-playbook` command-line tool. It is the exact same command as running it in check mode, just without the `--check` flag.

Generic command (-i flag is optional, please refer to the previous section for an explanation)

``` bash
$ ansible-playbook -i $HOSTS_INVENTORY_FILE $PLAYBOOK_PATH
```

In my case, I use it like this :

``` bash
$ ansible-playbook -i hosts --ask-sudo-pass devenv.yml
```

That's it for now ! There are many other topics I could explore with regards to Ansible but I wanted to keep the length of the post manageable.
If you have any questions or you noticed errors in this post please don't hesitate to leave a comment and I will get back to you shortly.


While writing this post, I was listening to : [Jaytech Music - Podcast 114](https://soundcloud.com/jaytechmusic/jaytech-music-podcast-114)
