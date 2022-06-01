---
title: Ansible 用户指南
date: 2022-05-17 00:01:21
tags: [Linux, ansible]
categories: Linux
---

# 安装指南

## 安装 Ansible

### 前提条件

#### 控制节点要求 （[Control node requirements](https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html#id8)）

除Windows外，其他任何安装了 Python 3.8 或更新版本的计算机都能成为控制节点，包括 RHEL，Debian，CentOS 以及任何 BSD 等等。

#### 被管理节点要求 （[Managed node requirements](https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html#id9)）

对于大多数被管理的节点，Ansible使用SSH和SFTP来创建连接。但如果SFTP不可用，也可以使用SCP。同时，还需要运行Python。

### 选择要安装的Ansible版本

有2种方式安装Ansible：S

- 使用各操作系统的包管理器安装预编译的Ansible包
- 使用`pip`安装Ansible

### 使用`pip`安装并升级Ansible

#### 使用`pip`安装Ansible

安装Ansible：

```bash
$ python -m pip install --user ansible
```

安装paramiko：

```bash
$ python -m pip install --user paramiko
```

全局安装Ansible：

```bash
$ sudo python -m pip install ansible
```

> Running `pip` with `sudo` will make global changes to the system. Since `pip` does not coordinate with system package managers, it could make changes to your system that leaves it in an inconsistent or non-functioning state. This is particularly true for macOS. Installing with `--user` is recommended unless you understand fully the implications of modifying global files on the system.

#### 在虚拟环境中使用`pip`安装Ansible

```bash
$ python -m virtualenv ansible  # Create a virtualenv if one does not already exist
$ source ansible/bin/activate   # Activate the virtual environment
$ python -m pip install ansible
```

> 高版本的Python3都自带了虚拟环境模块`venv`，不需要单独安装`virtualenv`

<!-- more -->

### 验证安装

不管使用哪种安装方式都可以使用以下命令验证Ansible安装正确：

```bash
$ ansible all -m ping --ask-pass
```

### 为Ansible命令添加Shell补全功能

#### [在RHEL, CentOS, or Fedora上安装`argcomplete`](https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html#installing-argcomplete-on-rhel-centos-or-fedora)

在Fedora上:

```bash
$ sudo dnf install python-argcomplete
```

在RHEL and CentOS上:

```bash
$ sudo yum install epel-release
$ sudo yum install python-argcomplete
```

#### [使用`pip`安装`argcomplete`](https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html#id41)

```bash
$ python -m pip install argcomplete
```

#### [配置`argcomplete`](https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html#id42)

有2种方式可以配置`argcomplete`：

- 全局配置（Global configuration）
- 每命令配置 （Per command configuration）

全局补全配置要求bash 4.2：

```bash
$ sudo activate-global-python-argcomplete
```

如果没有bash 4.2，则必须为每个脚本独立注册。将以下命令放入shell配置文件`~/.profile` 或者 `~/.bash_profile`：

```bash
$ eval $(register-python-argcomplete ansible)
$ eval $(register-python-argcomplete ansible-config)
$ eval $(register-python-argcomplete ansible-console)
$ eval $(register-python-argcomplete ansible-doc)
$ eval $(register-python-argcomplete ansible-galaxy)
$ eval $(register-python-argcomplete ansible-inventory)
$ eval $(register-python-argcomplete ansible-playbook)
$ eval $(register-python-argcomplete ansible-pull)
$ eval $(register-python-argcomplete ansible-vault)
```

## Ansible配置文件

### 获取最新的配置

如果是从包管理器安装Ansible，则最新的`ansible.cfg`文件应该在`/etc/ansible`中。

如果使用`pip`安装Ansible，你可能需要手动创建此文件以覆盖旧的Ansible配置。可以从github获取[最新的`ansible.cfg`实例](https://github.com/ansible/ansible/blob/devel/examples/ansible.cfg)。

从Ansible 2.4开始，可以使用`ansible-config`命令行工具查看所有可用的选项并检查当前的值。

从Ansible 2.12(core) 开始，要生成示例配置文件（所有默认`disabled`的配置都被注释）可以使用`ansible-config init`命令：

```bash
$ ansible-config init --disabled > ansible.cfg
```

通过包含存在的插件还生成更完整的配置文件：

```bash
$ ansible-config init --disabled -t all > ansible.cfg
```

要获得所有可用的配置请访问[configuration_settings](https://docs.ansible.com/ansible/latest/reference_appendices/config.html#ansible-configuration-settings).

### 环境配置

Ansible也允许使用环境变量配置。如果这些环境变量被设置，则从配置文件加载的配置将被覆盖。

要获得所有可用的环境变量，请访问[Ansible Configuration Settings](https://docs.ansible.com/ansible/latest/reference_appendices/config.html#ansible-configuration-settings)

### 命令行选项

命令行只包含最常用的配置选项。命令行中的参数将覆盖从配置文件和环境变量传入的配置。

# 用户手册

## Ansible中的概念

- 控制节点 (Control node)：必须是Linux。需要安装Python。
- 被管理节点 (Managed nodes)：
- Inventory:
- Collections:
- Modules:
- Tasks
- Playbooks

