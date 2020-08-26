# Vault Client Image

The Operator initiates Jobs that make API calls to Vault using the Vault CLI.

The default configuration is to use the `vault:latest` image.

To use a different image change the value of `vault_client_image` in the following files.

*   [vaultkubeauth](../roles/vaultkubeauth/defaults/main.yml) defaults file
*   [vaultkuberole](../roles/vaultkuberole/defaults/main.yml) defaults file
