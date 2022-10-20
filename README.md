# Installs an Unix Vagrant machine with any version of Vault is specified and the latest Terraform OSS version

### Repository has been created using [this repo](https://github.com/kikitux/vault-dev-orcl).This repository is going to spin-up a Vault dev server with the root token `changeme`.
### We are going to demonstrate how the Vault provider for Terraform works with the Approle authentication method.


## Prerequisites
- [X] [VirtualBox](https://www.virtualbox.org/wiki/Downloads)
- [X] [Vagrant](https://www.vagrantup.com/downloads)

## Instructions

- Copy the repository and go to the repo path:

```shell
git clone https://github.com/dlavric/demo-vault-tf-provider.git
cd demo-vault-tf-provider
```

- Start Vagrant and provision it:

```shell
vagrant up
```

- Check the version of Vault inside Vagrant VM:

```shell
vagrant ssh
vault version
```

- Check the version of Terraform:
```shell
terraform version
```

- Export the Vault address to talk to Vault:
```shell
export VAULT_ADDR="http://127.0.0.1:8200"
```
- Check that Vault is initialized and unsealed:

```shell
vault status

Key             Value
---             -----
Seal Type       shamir
Initialized     true
Sealed          false
Total Shares    1
Threshold       1
Version         1.12.0
Build Date      2022-10-10T18:14:33Z
Storage Type    inmem
Cluster Name    vault-cluster-4b936374
Cluster ID      f1804c5f-2882-fd81-0d0f-1faf72162085
HA Enabled      false
```

- Go to the following path and make sure you have the `main.tf` file there:
```shell
cd /vagrant
```

- Initialize the Terraform dependecies:
```shell
terraform init

Initializing the backend...

Successfully configured the backend "local"! Terraform will automatically
use this backend unless the backend configuration changes.

Initializing provider plugins...
- Finding latest version of hashicorp/vault...
- Installing hashicorp/vault v3.9.1...
- Installed hashicorp/vault v3.9.1 (signed by HashiCorp)

Terraform has created a lock file .terraform.lock.hcl to record the provider
selections it made above. Include this file in your version control repository
so that Terraform can guarantee to make the same selections by default when
you run "terraform init" in the future.

Terraform has been successfully initialized!

You may now begin working with Terraform. Try running "terraform plan" to see
any changes that are required for your infrastructure. All Terraform commands
should now work.

If you ever set or change modules or backend configuration for Terraform,
rerun this command to reinitialize your working directory. If you forget, other
commands will detect it and remind you to do so if necessary.
```

- Apply the changes and create the Approle backend in Vault with the Vault Terraform Provider:
```shell
terraform apply

Terraform used the selected providers to generate the following execution plan. Resource actions are indicated with the following symbols:
  + create

Terraform will perform the following actions:

  # vault_approle_auth_backend_login.login will be created
  + resource "vault_approle_auth_backend_login" "login" {
      + accessor       = (known after apply)
      + backend        = (known after apply)
      + client_token   = (known after apply)
      + id             = (known after apply)
      + lease_duration = (known after apply)
      + lease_started  = (known after apply)
      + metadata       = (known after apply)
      + policies       = (known after apply)
      + renewable      = (known after apply)
      + role_id        = (known after apply)
      + secret_id      = (sensitive)
    }

  # vault_approle_auth_backend_role.example will be created
  + resource "vault_approle_auth_backend_role" "example" {
      + backend        = (known after apply)
      + bind_secret_id = true
      + id             = (known after apply)
      + role_id        = (known after apply)
      + role_name      = "test-role"
      + token_policies = [
          + "default",
          + "dev",
          + "prod",
        ]
      + token_type     = "default"
    }

  # vault_approle_auth_backend_role_secret_id.id will be created
  + resource "vault_approle_auth_backend_role_secret_id" "id" {
      + accessor          = (known after apply)
      + backend           = (known after apply)
      + id                = (known after apply)
      + role_name         = "test-role"
      + secret_id         = (sensitive value)
      + wrapping_accessor = (known after apply)
      + wrapping_token    = (sensitive value)
    }

  # vault_auth_backend.approle will be created
  + resource "vault_auth_backend" "approle" {
      + accessor        = (known after apply)
      + disable_remount = false
      + id              = (known after apply)
      + path            = (known after apply)
      + tune            = (known after apply)
      + type            = "approle"
    }

Plan: 4 to add, 0 to change, 0 to destroy.

Do you want to perform these actions?
  Terraform will perform the actions described above.
  Only 'yes' will be accepted to approve.

  Enter a value: yes

vault_auth_backend.approle: Creating...
vault_auth_backend.approle: Creation complete after 0s [id=approle]
vault_approle_auth_backend_role.example: Creating...
vault_approle_auth_backend_role.example: Creation complete after 0s [id=auth/approle/role/test-role]
vault_approle_auth_backend_role_secret_id.id: Creating...
vault_approle_auth_backend_role_secret_id.id: Creation complete after 0s [id=backend=approle::role=test-role::accessor=53b26c9b-a196-88fa-965d-5e01a6723c70]
vault_approle_auth_backend_login.login: Creating...
vault_approle_auth_backend_login.login: Creation complete after 0s [id=dvOmEfds1qroypmVbww9XEMk]

Apply complete! Resources: 4 added, 0 changed, 0 destroyed.
```

- Check that the Approle authentication method has been enabled in Vault
```shell
vault auth list

Path        Type       Accessor                 Description                Version
----        ----       --------                 -----------                -------
approle/    approle    auth_approle_4d838bd5    n/a                        n/a
token/      token      auth_token_c7e0f481      token based credentials    n/a

```

- Fetch the `role id` for the Approle:

```shell
vault read auth/approle/role/test-role/role-id

Key        Value
---        -----
role_id    9bff5d5e-f3e6-c327-6d61-ae4f9d1b102d
```

- Geth the `secret id` issued against the Approle

```shell
vault write -f auth/approle/role/test-role/secret-id

Key                   Value
---                   -----
secret_id             70f90eec-51d6-ca6c-c56c-2817cf962fa4
secret_id_accessor    091ea45e-6cbd-d93e-3fac-663c56d9a6d9
secret_id_num_uses    0
secret_id_ttl         0s
```

- Destroy the resources with Terraform

```shell
terraform destroy

vault_auth_backend.approle: Refreshing state... [id=approle]
vault_approle_auth_backend_role.example: Refreshing state... [id=auth/approle/role/test-role]
vault_approle_auth_backend_role_secret_id.id: Refreshing state... [id=backend=approle::role=test-role::accessor=53b26c9b-a196-88fa-965d-5e01a6723c70]
vault_approle_auth_backend_login.login: Refreshing state... [id=dvOmEfds1qroypmVbww9XEMk]

Terraform used the selected providers to generate the following execution plan. Resource actions are indicated with the following symbols:
  - destroy

Terraform will perform the following actions:

  # vault_approle_auth_backend_login.login will be destroyed
  - resource "vault_approle_auth_backend_login" "login" {
      - accessor       = "dvOmEfds1qroypmVbww9XEMk" -> null
      - backend        = "approle" -> null
      - client_token   = "hvs.CAESIC0OVP67f_cNynlDx0TrNnSeO5FOnXbQzViVwYdQ_bU7Gh4KHGh2cy5hNXRTZXRNSXJFRjNBcWE5WTd2aHRqRWQ" -> null
      - id             = "dvOmEfds1qroypmVbww9XEMk" -> null
      - lease_duration = 0 -> null
      - lease_started  = "2022-10-20T15:15:56Z" -> null
      - metadata       = {} -> null
      - policies       = [
          - "default",
          - "dev",
          - "prod",
        ] -> null
      - renewable      = true -> null
      - role_id        = "9bff5d5e-f3e6-c327-6d61-ae4f9d1b102d" -> null
      - secret_id      = (sensitive) -> null
    }

  # vault_approle_auth_backend_role.example will be destroyed
  - resource "vault_approle_auth_backend_role" "example" {
      - backend                 = "approle" -> null
      - bind_secret_id          = true -> null
      - id                      = "auth/approle/role/test-role" -> null
      - role_id                 = "9bff5d5e-f3e6-c327-6d61-ae4f9d1b102d" -> null
      - role_name               = "test-role" -> null
      - secret_id_bound_cidrs   = [] -> null
      - secret_id_num_uses      = 0 -> null
      - secret_id_ttl           = 0 -> null
      - token_bound_cidrs       = [] -> null
      - token_explicit_max_ttl  = 0 -> null
      - token_max_ttl           = 0 -> null
      - token_no_default_policy = false -> null
      - token_num_uses          = 0 -> null
      - token_period            = 0 -> null
      - token_policies          = [
          - "default",
          - "dev",
          - "prod",
        ] -> null
      - token_ttl               = 0 -> null
      - token_type              = "default" -> null
    }

  # vault_approle_auth_backend_role_secret_id.id will be destroyed
  - resource "vault_approle_auth_backend_role_secret_id" "id" {
      - accessor  = "53b26c9b-a196-88fa-965d-5e01a6723c70" -> null
      - backend   = "approle" -> null
      - cidr_list = [] -> null
      - id        = "backend=approle::role=test-role::accessor=53b26c9b-a196-88fa-965d-5e01a6723c70" -> null
      - metadata  = jsonencode({}) -> null
      - role_name = "test-role" -> null
      - secret_id = (sensitive value)
    }

  # vault_auth_backend.approle will be destroyed
  - resource "vault_auth_backend" "approle" {
      - accessor        = "auth_approle_4d838bd5" -> null
      - disable_remount = false -> null
      - id              = "approle" -> null
      - local           = false -> null
      - path            = "approle" -> null
      - tune            = [] -> null
      - type            = "approle" -> null
    }

Plan: 0 to add, 0 to change, 4 to destroy.

Do you really want to destroy all resources?
  Terraform will destroy all your managed infrastructure, as shown above.
  There is no undo. Only 'yes' will be accepted to confirm.

  Enter a value: yes

vault_approle_auth_backend_login.login: Destroying... [id=dvOmEfds1qroypmVbww9XEMk]
vault_approle_auth_backend_login.login: Destruction complete after 0s
vault_approle_auth_backend_role_secret_id.id: Destroying... [id=backend=approle::role=test-role::accessor=53b26c9b-a196-88fa-965d-5e01a6723c70]
vault_approle_auth_backend_role_secret_id.id: Destruction complete after 0s
vault_approle_auth_backend_role.example: Destroying... [id=auth/approle/role/test-role]
vault_approle_auth_backend_role.example: Destruction complete after 0s
vault_auth_backend.approle: Destroying... [id=approle]
vault_auth_backend.approle: Destruction complete after 0s

Destroy complete! Resources: 4 destroyed.
```

- Destroy the Vagrant VM:
```shell
vagrant destroy
```