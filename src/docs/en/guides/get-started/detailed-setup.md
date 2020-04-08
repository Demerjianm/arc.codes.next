---
title: Detailed setup
description: 160 (or fewer) character description of this document!
sections:
  - Prerequisites
  - AWS Setup & IAM credentials
  - Configure AWS CLI
  - Install Architect
  - Example project
  - Work locally
  - Deploying
  - Clean Up
---

## Prerequisites

**To work locally**, all you need is:
- A supported [runtime](#runtimes)

**To deploy your project to AWS**, you'll need:
- A supported [runtime](#runtimes)
- [AWS CLI](https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-configure.html)
  - (Which requires [Python](https://www.python.org/downloads/))
- AWS account with admin privileges
- Your [credentials file](#local-credentials-file) set up at:
  - \*nix: `~/.aws/credentials`
  - Windows: `C:\Users\USER_NAME\.aws\credentials`


### Runtimes

Architect supports the following runtime versions when working locally:

**Recommended**
- Node: `10.x` & `8.10` using `npm`
- Ruby: `2.5` using `bundle`
- Python: `3.7` & `3.6` using `pip3`

> Working locally with the Architect `sandbox` dev server requires target runtimes to be installed and available in $PATH. For example, if you are only targeting Node, then only Node needs to be installed locally. (Likewise for Ruby and Python.)

To use the same runtime across all functions in your project, add it to your `@aws` pragma like so:

```
# Valid runtimes: `nodejs12.x`, `nodejs10.x`, `deno`, `python3.7`, `python3.6`, or `ruby2.5`
@aws
runtime python3.7
```

This setting can be overridden on a function-by-function basis with [`.arc-config`](/reference/arc-config/runtime).

Architect also supports the following runtimes in live infra, but not while working locally (at present):
- Go: `1.x`,
- .NET: `2.1`
- Java: `8`

## AWS Setup & IAM credentials

You'll need an Amazon Web Services account and credentials set up on your development machine. If you haven't done it before, here's a useful guide for [Configuring the AWS CLI](https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-getting-started.html).

On \*nix systems AWS Credentials are listed in:

```bash
~/.aws/credentials
```

Or on Windows systems:

```bash
C:\Users\USER_NAME\.aws\credentials
```

If that file doesn't exist, create it, and add something like the following (assuming you have multiple AWS accounts):

```bash
[default]
aws_access_key_id=xxx
aws_secret_access_key=xxx

[work]
aws_access_key_id=xxx
aws_secret_access_key=xxx

[personal]
aws_access_key_id=xxx
aws_secret_access_key=xxx
```

You will also need to set a default profile and region with the environment variables 

- `AWS_PROFILE`
- `AWS_REGION`

To set these variables on Linux, macOS, or UNIX, use export in your `~/.bashrc` (or equivalent shell configuration):

```bash
export AWS_PROFILE=work
export AWS_REGION=us-west-1
```

Or for Windows, add this to your PowerShell `$profile`:

```powershell
$env:AWS_PROFILE='work'
$env:AWS_REGION='us-west-1'
```

If you prefer, you can also use: *Control Panel » System » Advanced System Settings » Environment Variables*.

## Configure AWS CLI

ADD ME!

## Install Architect

```bash
npm install -g @architect/architect
```

## Example project

Create a project folder

```bash
mkdir testapp
cd testapp
```

## Work locally

3. Run `arc init` to generate a basic project:

```
/
├── src
│   └── http
│       └── get-index/index.js
└── .arc
```

4. Check out your first `.arc` file & HTTP function!

```bash
# /project/path/.arc
@app
your-app-name

@http
get /
```

```javascript
// src/http/get-index/index.js

exports.handler = async function http(request) {
  return {
    headers: { 'Content-Type': 'text/html; charset=utf-8;' },
    body: '<h1>Hello World! 🎉</h1>'
  }
}
```

## Deploying

ADD ME!


## Clean Up

ADD ME!

### Useful links:
* [Configuring the AWS CLI](https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-getting-started.html)
* [Amazon Configuration and Credential Files](https://docs.aws.amazon.com/cli/latest/userguide/cli-config-files.html)
* [Working with multiple AWS accounts](/guides/multiple-aws-accounts)