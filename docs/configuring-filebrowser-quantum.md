<!--
SPDX-FileCopyrightText: 2020 - 2024 MDAD project contributors
SPDX-FileCopyrightText: 2020 - 2024 Slavi Pantaleev
SPDX-FileCopyrightText: 2020 Aaron Raimist
SPDX-FileCopyrightText: 2020 Chris van Dijk
SPDX-FileCopyrightText: 2020 Dominik Zajac
SPDX-FileCopyrightText: 2020 Mickaël Cornière
SPDX-FileCopyrightText: 2022 François Darveau
SPDX-FileCopyrightText: 2022 Julian Foad
SPDX-FileCopyrightText: 2022 Warren Bailey
SPDX-FileCopyrightText: 2023 Antonis Christofides
SPDX-FileCopyrightText: 2023 Felix Stupp
SPDX-FileCopyrightText: 2023 Pierre 'McFly' Marty
SPDX-FileCopyrightText: 2024 - 2025 Suguru Hirahara

SPDX-License-Identifier: AGPL-3.0-or-later
-->

# Setting up FileBrowser Quantum

This is an [Ansible](https://www.ansible.com/) role which installs [FileBrowser Quantum](https://filebrowserquantum.com) to run as a [Docker](https://www.docker.com/) container wrapped in a systemd service.

FileBrowser Quantum is a free self-hosted web-based file manager.

See the project's [documentation](https://filebrowserquantum.com/en/docs/) to learn what FileBrowser Quantum does and why it might be useful to you.

## Adjusting the playbook configuration

To enable FileBrowser Quantum with this role, add the following configuration to your `vars.yml` file.

**Note**: the path should be something like `inventory/host_vars/mash.example.com/vars.yml` if you use the [MASH Ansible playbook](https://github.com/mother-of-all-self-hosting/mash-playbook).

```yaml
########################################################################
#                                                                      #
# filebrowser_quantum                                                  #
#                                                                      #
########################################################################

filebrowser_quantum_enabled: true

########################################################################
#                                                                      #
# /filebrowser_quantum                                                 #
#                                                                      #
########################################################################
```

### Set the hostname

To enable FileBrowser Quantum you need to set the hostname as well. To do so, add the following configuration to your `vars.yml` file. Make sure to replace `example.com` with your own value.

```yaml
filebrowser_quantum_hostname: "example.com"
```

After adjusting the hostname, make sure to adjust your DNS records to point the domain to your server.

### Set an administrator's password

If the password authentication is enabled, you need to set a log in password for the administrator by adding the following configuration to your `vars.yml` file:

```yaml
filebrowser_quantum_environment_variables_filebrowser_admin_password: YOUR_ADMIN_PASSWORD_HERE
```

Replace `YOUR_ADMIN_PASSWORD_HERE` with your own value.

### Configuring OIDC authentication (optional)

You can enable OIDC authentication for FileBrowser Quantum by adding the following settings to your `vars.yml` file:

```yaml
filebrowser_quantum_config_auth_methods_oidc: true

# Specify OIDC client ID
filebrowser_quantum_config_auth_methods_oidc_clientid: YOUR_OIDC_CLIENT_ID_HERE

# Specify OIDC client secret
filebrowser_quantum_environment_variables_filebrowser_oidc_client_secret: YOUR_OIDC_CLIENT_SECRET_HERE

# Specify OIDC provider URL
filebrowser_quantum_config_auth_methods_oidc_issuerurl: YOUR_OIDC_PROVIDER_URL_HERE

# Set to `false` to prevent users from being create on the first login
filebrowser_quantum_config_auth_methods_oidc_createuser: true
```

Make sure to replace each placeholder with actual values.

Refer to [this page](https://filebrowserquantum.com/en/docs/configuration/authentication/oidc/) on the documentation for details.

### Configuring ONLYOFFICE Docs integration (optional)

You also can configure [ONLYOFFICE Docs](https://helpcenter.onlyoffice.com/docs) to be integrated to FileBrowser Quantum for editing document files. To enable it, add the following settings to your `vars.yml` file:

```yaml
filebrowser_quantum_config_integrations_office: true
filebrowser_quantum_config_integrations_office_url: YOUR_ONLYOFFICE_DOCS_INSTANCE_URL_HERE
filebrowser_quantum_config_integrations_office_secret: YOUR_ONLYOFFICE_DOCS_INSTANCE_SECRET_HERE
```

To `filebrowser_quantum_config_integrations_office_secret` set the same value as one specified to `JWT_SECRET` for ONLYOFFICE Docs instance.

See [this page](https://filebrowserquantum.com/en/docs/integrations/office/configuration/) for configuration details.

If you are looking for an Ansible role for ONLYOFFICE Docs, you can check out [ansible-role-onlyoffice-docs](https://app.radicle.xyz/nodes/seed.radicle.garden/rad:z3kozTn4Kn5eJtgJQj1aCFUpqxW5Y) maintained by the [Mother-of-All-Self-Hosting (MASH)](https://github.com/mother-of-all-self-hosting) team.

### Extending the configuration

There are some additional things you may wish to configure about the service.

Take a look at:

- [`defaults/main.yml`](../defaults/main.yml) for some variables that you can customize via your `vars.yml` file. You can override settings (even those that don't have dedicated playbook variables) using the `filebrowser_quantum_config_additional_configurations` and `filebrowser_quantum_environment_variables_additional_variables` variables

See the [documentation](https://filebrowserquantum.com/en/docs/configuration/configuration-overview/) for a complete list of FileBrowser Quantum's config options that you could put in `filebrowser_quantum_config_additional_configurations`.

## Installing

After configuring the playbook, run the installation command of your playbook as below:

```sh
ansible-playbook -i inventory/hosts setup.yml --tags=setup-all,start
```

If you use the MASH playbook, the shortcut commands with the [`just` program](https://github.com/mother-of-all-self-hosting/mash-playbook/blob/main/docs/just.md) are also available: `just install-all` or `just setup-all`

## Usage

After running the command for installation, FileBrowser Quantum becomes available at the specified hostname like `https://example.com`.

To get started, open the URL with a web browser, and log in to the instance with the administrator account.

If password authentication is enabled, you can log in with the default username set to `filebrowser_quantum_config_auth_adminusername` and the password specified to `filebrowser_quantum_environment_variables_filebrowser_admin_password`.

If OIDC authentication is enabled, click "OpenID Connect" button to log in to the service.

## Troubleshooting

### How to fix the `user attempted to login with wrong login method` error

If OIDC authentication is enabled with the configuration file and logging in to the UI fails due to the 403 error (`user attempted to login with wrong login method`), make sure that the login method is set to `OIDC` to your user on the user management settings at `https://example.com/settings#users-main`.

Note that logging in with the password is disabled if the method is set to `OIDC`.

### Check the service's logs

You can find the logs in [systemd-journald](https://www.freedesktop.org/software/systemd/man/systemd-journald.service.html) by logging in to the server with SSH and running `journalctl -fu filebrowser-quantum` (or how you/your playbook named the service, e.g. `mash-filebrowser-quantum`).
