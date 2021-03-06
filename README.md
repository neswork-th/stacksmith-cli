# stacksmith-cli
CLI utility to talk to https://stacksmith.bitnami.com

The main use case is to call this utility from the build automation (CI) tool that
already builds your application and let Stacksmith build cloud images for you.

## Install

You can fetch a binary for your platform via the [Github releases](https://github.com/bitnami/stacksmith-cli/releases) page. For example, on linux:

```
  wget https://github.com/bitnami/stacksmith-cli/releases/download/v0.10.0/stacksmith-linux-amd64 -O /tmp/stacksmith
  sudo install /tmp/stacksmith /usr/local/bin/stacksmith
```

The stacksmith CLI is also available as a docker image:

```
docker run -ti gcr.io/bitnami-labs/stacksmith:0.10.0
```

## First steps

1. Sign up to https://stacksmith.bitnami.com and pick a project
2. Use `stacksmith init` to create a `Stackerfile.yml` file pointing to your app and your local build artifacts to upload
4. Run `stacksmith build`

See full example in https://github.com/bitnami-labs/stacksmith-ci-example using Travis CI.

## Documentation

You can learn about the possible fields of the `Stackerfile.yml` file in its [reference documentation](https://github.com/bitnami/stacksmith-cli/blob/master/docs/schema.md).

Furthermore, the CLI tool offers inline documentation for commands and topics:

```
$ stacksmith help
$ stacksmith help build
...
```

Common topics will be presented in this README.


## Configuring CI authorization

If you need to run stacksmith build on another machine, e.g. as part of an automated build job,
you obviously cannot rely on the interactive `stacksmith auth login` tool on the server.

You can generate and export a token for such purposes by following these instructions:

1. Make sure you have a working account on https://stacksmith.bitnami.com

2. Download the stacksmith client (see the [release page](https://github.com/bitnami/stacksmith-cli/releases)) for your platform and authenticate. This command will open a browser window which might ask you to log-in into stacksmith if you haven't already:

```
stacksmith auth login
```


3. Generate a new access token. You can add a description to your newly created token so you can later manage your access tokens more easily:

```
stacksmith auth access-tokens create --description my-CI-integration
```

4. Copy&paste the output of the previous command (the long string starting with `MDAxOGxvY2F0aW9uI...`) into a secret env var, e.g. `STACKSMITH_ACCESS_TOKEN` on your CI system.

## Consuming build results

The `stacksmith` CLI utility will output the build logs to stderr and, in case of success, a Build Spec JSON object on stdout.

The Build Spec JSON object contains, among other things, pointers to the build results, such as the AMI ID or the docker
image name or the deployment template URL.

Here's an example that shows how to perform a build and retrieve the Helm chart generated by Stacksmith:

```
$ stacksmith build | tee image_spec.json
$ stacksmith get template -s image_spec.json -o my-helm-chat.tgz
```

The Build Spec also contains a snapshot of all the installed packages. You may want to instruct your CI system to
archive this file along with your build artifacts for future reference.

## Using arbitrary base images

You can specify a base image for each platform. E.g. this shows how to use a custom AMI:

```
baseImages:
  aws:
    amiId: "ami-4bf3d731"
```

Each distribution might have a different convention w.r.t the default username.
Amazon linux and RHEL use **ec2-user**, but some other distributions use other usernames.

For example here is how to use a Ubuntu base image:

```
baseImages:
  aws:
    amiId: "ami-0a313d6098716f372"
    sshUsername: ubuntu
```

## Defining a multi-image application

You can specify multiple images to build for a single application. Each will be
built separately, and then the deployment template will be able to reference
each of the images.

Notes: This currently only works for AWS AMI builds. Other targets will be
available soon.

In order to specify multiple images use the `images:` key, passing
an array of image descriptions.

```
images:
  - name: "mariadb"
    files:
      userScripts:
        boot: mariadb-boot.sh
  - name: "app"
    files:
      user_scripts:
        run: app-run.sh
      userUploads:
        - app.zip
```

In addition you will need to specify a deployment template so that
you can reference each of the images.

```
deployment_templates:
  aws: cf.json
```

See [the stacksmith docs](https://stacksmith.bitnami.com/+/support/creating-a-stack-template#Creating-a-custom-deployment-template)
for how to provide a custom deployment template.
