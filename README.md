# aws-otp: Yet another AWS MFA helper

## Motivation

* Keep long-term credentials secure (not in a plain-text), e.g. in a password manager.
* Automate the process of obtaining short-lived credentials with OTP stored either in a password manager (less secure) or on Yubikey.
* Allow installing short-term credentials on the remote host without storing long-term credentials there. This is an attempt to mimic ssh-agent for AWS credentials.

## Usage

This needs some configuration in `$HOME/.aws/credentials`:

1. Create a profile with a `-long-term` suffix, e.g. `acme-long-term` with your long-term AWS credentials
2. Add `aws_mfa_defice` (compatible with [aws-mfa](https://github.com/broamski/aws-mfa)) and `otp_command`  (see below) to your long-term AWS profile.

Then, either:
* `$ aws-otp PROFILE` (e.g. `aws-otp acme` to setup `acme` profile locally based on `acme-long-term`)
* `$ aws-otp PROFILE SSH_HOST` (e.g. `aws-otp acme mainframe` to setup `acme` profile on ssh host based on local `acme-long-term`)

## Requirements

* bash
* jq
* aws-cli

## Example OTP commands

### YubiKey integration

Use YubiKey to provide OTP, e.g.:

```shell
ykman oath accounts code --single AWS:me@acme
```

### 1Password

Use 1Password CLI to get OTP, e.g.:

```shell
op item get "AWS acme" --otp
```

## Storing long-term credentials securely

AWS CLI allows providing long-term credentials with an external command. For instance, one can fetch credentials from 1Password:

```
[acme-long-term]
credential_process = sh -c "op item get --cache \"Amazon (ACME)\" --fields label=\"AWS_ACCESS_KEY_ID\",\"AWS_SECRET_ACCESS_KEY\" --format json | jq '. | map({(.label):.}) | add | {Version:1, AccessKeyId: .\"AWS_ACCESS_KEY_ID\".value, SecretAccessKey:.\"AWS_SECRET_ACCESS_KEY\".value}'"
```

## Related projects

* [aws-mfa](https://github.com/broamski/aws-mfa)
* [aws-credential-1password](https://github.com/claui/aws-credential-1password)
