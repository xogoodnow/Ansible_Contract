# Ansible Best Practices Guide for SRE Team

## Introduction
This document outlines the best practices for using Ansible within our SRE team. It serves as a guide for writing efficient, maintainable, and reusable Ansible code.

## Table of Contents
- [YAML for Inventories](#yaml-for-inventories)
- [Defining Roles](#defining-roles)
- [Naming Conventions](#naming-conventions)
  - [Roles](#roles)
  - [Playbooks](#playbooks)
  - [Tags](#tags)
  - [Variables](#variables)
  - [Tasks](#tasks)
- [Directory Structure](#directory-structure)
  - [Playbooks](#playbooks-1)
  - [Roles](#roles-1)
  - [Variables](#variables-1)
- [Best Practices](#best-practices)
  - [Using Tags](#using-tags)
  - [Variable Management](#variable-management)
  - [Importing Roles](#importing-roles)
  - [Error Handling](#error-handling)
  - [Task Formatting](#task-formatting)

## 0.0.0: Main principles
### 0.0.1: When simple is possible, do it simple.
### 0.0.2: Do not expose sensitive information without vault.
### 0.0.3: Keep in mind that whatever you are creating is for someone else and not you.
### 0.0.4: We start counting from `1` and not `0`.
### 0.0.5: We use `yaml` and not `yml` for our files.

## 1.0.0: Inventories
### 1.0.1:
  * YAML is preferred over INI for inventories due to its readability and support for complex data structures. This format enhances clarity and maintainability.
### 1.0.2:
  * In the Inventories folder, there are multiple zones.
    * As an example: `simin-production or simin-stage`

## 2.0.0: Roles
### 2.0.1:
  * A role in Ansible is a set of tasks and additional files to configure a host to serve a specific purpose. Following the `SOLID` principle, each role should have a single responsibility, be open for extension but closed for modification, and be interchangeable.
    * Solid principles:
      1. Single Responsibility Principle (SRP): Each Ansible role or task should have one specific job
      2. Open-Closed Principle (OCP): Ansible roles should be designed to be extendable. You should be able to add new features or functionalities to a role without modifying the existing code
      3. Liskov Substitution Principle (LSP): This principle is more relevant to object-oriented programming and might not apply directly to Ansible. However, the concept of substitutability can be applied to Ansible roles. For example, you should be able to replace one role with another that performs a similar function without affecting the rest of the system
      4. Interface Segregation Principle (ISP): Ansible roles should be designed to be as small and focused as possible.
      5. Dependency Inversion Principle (DIP): High-level playbooks should not depend on low-level roles. Instead, both should depend on abstract roles.

### 2.0.2:
  * All role names must be only the name ofg the component.
    * As an example: `elasticsearch_cluster`

### 2.0.3:
  * For sensitive tasks, a moderate use of the debug module is appreciated.

### 2.0.4:
  * All role names must be lowercase and alphanumeric; also they must start with a letter.
    * As an example: `elasticsearch_cluster`

### 2.0.5:
  * All task file names must be expressive of their nature.
    * As an example: `elasticsearch_exporter_setup.yaml`

### 2.0.6:
  * Task names must start with an uppercase letter.
    * As an example: `Check connectivity`

### 2.0.7:
  * No role should contain more than 40 tasks unless they are for debugging purposes

## 3.0.0: Playbooks
### 3.0.1:
  * Use one main playbook for a component and manage different operations with tags.
    * As an example: `elasticsearch_deploy.yaml`
### 3.0.2:
  * Include all Playbooks in site.yaml
```
---
# file: site.yml
- import_playbook: webservers.yml
- import_playbook: dbservers.yml
```


## 4.0.0: Tags
### 4.0.1:
  * Use tags for operations related to maintaining a service, not initial deployments.
### 4.0.2:
  * Tag names should be a combination of the component name and the associated action.
    * For example: `docker:pull`
### 4.0.3:
  * Tag names should be alphabetical and lowercased
    * For example: `docker:start`

## 5.0.0: Variable Management
### 5.0.1:
  * Avoid default values for variables; always provide descriptive comments for each variable.
### 5.0.2:
  * All variables must be in lowercase.
    * For example: `kafka_base_image`
### 5.0.3:
  * Use comments to provide alternate values for variables.
    * For example: `kafka_exporter_base_image: bitnami/kafka-exporter #danielqsj/kafka-exporter`
### 5.0.4:
  * When providing a directory path as the value for a variable, always use the final "/".
    * For example: `kafka_cert_directory: /root/Kafka/Config/Certs/`
### 5.0.5:
  * Use group_vars or host_vars Files to set variables.
    * For example: `Kafka_Variables_Simin_Production.yaml`

## 6.0.0: Directory Structure

### 6.0.1:
  * Each part of a services must be in a logically seperated directory in servers

    ```bash

    root/
      ├─ Kafka/
      │  ├─ Config/
      │  │   ├─ Certs
      │  │   │   ├─ ca-cert
      │  │
      │  │─ Docker
      │  │  ├─ docker-compose.yaml
      │  │
      │  │─ Exporter
      │  │  │─ Docker
      │  │  │  ├─ docker-compose.yaml
      │  ├─ Gui/
      │  │  │─ Docker
      │  │  │  ├─ docker-compose.yaml

    ```

## 7.0.0: Agreed Upon Practices

### 7.0.1:
  * When working with containers, use docker compose template instead of directly running the container so this way the service would be much easier to manager in the future

### 7.0.2:
  * Prefer `include_roles` over `import_roles` or `roles` for better performance and readability.

### 7.0.3:
  * Set `any_errors_fatal: true` to ensure clean execution of playbooks.

### 7.0.4:
  * Use `command` instead of `shell` since command is faster.

### 7.0.5:
  * Use `delegate_to` instead of `local_action`.

### 7.0.6:
  * Do not use empty string comparission.
    * As an example: `when: var | lengh > 0` instead of `when: var != ""`

### 7.0.7:
  * Do not use `ignore_errors=True`; instead use `failed_when`.

### 7.0.8:
  * Do not use inline variables, instead include the variable in the vars file.

### 7.0.9:
  * Unless there is a good reason, do not use latest, explicitly specify the version for all packages and images.

### 7.1.0:
  * Use `when: var != True` instead of `when: not var`.

### 7.1.1:
  * Use `no_log=True` for sensitive information so they wont be exposed when a play is running.

### 7.1.2:
  * Do not prompt for anything so the play can also be used in CI/CD if needed.

### 7.1.3:
  * Explicitly specify paths and do not use relative addressing.

### 7.1.4:
  * Use `pipefail` when piping the output of a command to another so if the first command failed the whole task would be failed.

### 7.1.5:
  * on playbooks, do not specify the path to the role and leave it to ansible to find the role.
CONTRIBUTING

### 7.1.6:
  * Do not specify var files for roles if not necessary, use `defaults` and `group_vars` and `host_vars` directories for automatic detection of variables.
