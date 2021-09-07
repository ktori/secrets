# Secrets

---

⚠️ DISCLAIMER: This solution was not audited by any security experts. While from what I know it should be secure enough, I can't guarantee it. 

---

# Summary

Simple template-based secret management solution with minimal dependencies.

## TODO

* Make sure this is *actually* secure
* Don't hardcode `secrets`, `secrets_key.pem`, `secrets_key_pub.pem`
* Password-protected key support
* Make readme clearer

# Usage

## Setup

1. Add this repository as a submodule or just copy the scripts to a `secrets` subfolder in your repository
2. Generate a key pair in your repository root:
    * Private key: `openssl genpkey -algorithm RSA -out secrets_key.pem -pkeyopt rsa_keygen_bits:2048`
    * Public key: `openssl rsa -pubout -in secrets_key.pem -out secrets_key_pub.pem`
3. You should end up with something like this:

```
repo/
    secrets/
        decrypt-aes
        ...
        unwrap
    secrets_key.pem
    secrets_key_pub.pem
    ...
    conf/
        secret$passwords.conf
    README
    (rest of your files)
    ...
```
4. Add `secrets_key.pem` to your gitignore and store it safely. It is only required to decrypt the secrets and it should be present only where the decryption is needed. Commit `secrets_key_pub.pem` to the repository.

## Client side

*Client* refers here to the party that needs to encrypt secrets for the server. 

* To create a secret (short): `secrets/encrypt-rsa`
* To create a secret (long): `secrets/encrypt-aes`

Both scripts take input from stdin and expect the public key to be available at `./secrets_key_pub.pem`. 

* ⚠️ Make sure to terminate stdin with Ctrl+D before pressing Enter or you will end up with extra newlines at the end of your secret

Store those secret blocks inline in files where you want those secrets to appear. Each time a `{% secret %}` block is encountered during unwrap on server side, it will get replaced with secret text. 

* ⚠️ Files with `{% secret %}` blocks **MUST** be prefixed with `secret$` or unwrap script won't affect them

* ⚠️ Add filenames without `secret$` prefix to gitignore to prevent accidental commits of secret data

## Server side

*Server* refers here to the party that will need the decrypted secrets.

To decrypt all secrets, call `scripts/unwrap`. The script expects private key to be available at `./secrets_key.pem`. Script will generate decrypted `name` files for each `secret$name` file.

# Dependencies

- perl
- openssl (cli)

