# How Zerops Build Works

Adding [zerops.yml](/documentation/build/build-config.html) to your application source code (Zerops looks for it in the **root directory**), lets Zerops know which [base](/documentation/build/build-config.html#base-optional) technologies should be used, and which steps to run during the application [build](/documentation/build/build-config.html#build) phase to produce a final application runtime. The [deploy](/documentation/build/build-config.html#deploy) specifies which files will be copied to a runtime service container(s) when the build is successfully completed.

<!-- markdownlint-disable DOCSMD004 -->
::: tip Applications in monorepo environment
Each `zerops.yml` can contain definitions for one or more applications. It is especially valuable for a monorepo when one repository contains source code for various applications. The appropriate service hostname to which the application will be deployed is used to select a correct definition from the `zerops.yml`.
:::
<!-- markdownlint-enable DOCSMD004 -->

Internally, Zerops creates a new temporal build container inside your project that does not affect the existing running application in any way. Every step of the ongoing build process (and its [log](/documentation/zerops-logs/build-logs.html)) can be monitored within the Zerops GUI.

The same [zerops.yml](/documentation/build/build-config.html) file is also used to define the steps that Zerops has to invoke to [run](/documentation/build/build-config.html#run-part-and-its-properties) the application runtime after a successful deployment.

A new build can be triggered by using the Zerops [zCLI](/documentation/cli/installation-authorization.html) or connecting a Zerops service to a [GitHub](/documentation/github/github-integration.html) / [GitLab](/documentation/gitlab/gitlab-integration.html) repository.

The build process has access to all of the project service [environment variables](/documentation/environment-variables/overview.html#referencing-environment-variables) so that you can utilize them.

## Examples

### Node.js (using Npm)

```yaml
# Service hostname to which the application will be deployed.
nodejsapp:
  build:
    # Which technology should be used as a base one for creating a build container.
    base: [nodejs@14]
    # Which commands should be run to build the application.
    build:
      - npm i
      - npm run build:production
    # Which files and directories should be copied from a build container into a runtime container.
    deploy: [dist, node_modules, package.json]
  run:
    # A command that should start your service.
    start: npm start
```

### Golang

```yaml
# Service hostname to which the application will be deployed.
goapp:
  build:
    # Which technology should be used as a base one to create a build container.
    base: [go@1]
    # Which commands should be run to build the application.
    build:
      - go build -o ./bin/server ./app/server.go
    # Which files and directories should be copied from a build container into a runtime container.
    deploy: [./bin/server]
  run:
    # A command that should start your service.
    start: ./bin/server
```

### PHP (using Composer)

```yaml
# Service hostname to which the application will be deployed.
phpapp:
  build:
    # Which container type should be used as a base one to create a build container.
    base: [php@8.0]
    build:
      # Which tasks should be run to build the application.
      - composer install
    # Which files and directories should be copied from a build container into a runtime container.
    deploy: ['vendor', 'public']
  run:
    # Folder name used as the root of the publicly accessible web server content.
    documentRoot: public
```

See the full [zerops.yml](/documentation/build/build-config.html) documentation.

<!-- markdownlint-disable DOCSMD004 -->
::: warning Problems with Node.js dependency modules
Sometimes strange errors can happen when Node.js applications are built, and it looks as if they are related to the dependency tree in the `node_modules` directory. Before trying to investigate what it could be and spending time on it, add this command into the `build` section: `npm cache clean --force` as shown below:

```yaml
nodejsapp:
  build:
    # Which technology should be used as a base one for creating a build container.
    base: [nodejs@14]
    # Which commands should be run to build the application.
    build:
      - npm cache clean --force
      - npm i
      - npm run build:production
    # Which files and directories should be copied from a build container into a runtime container.
    deploy: [dist, node_modules, package.json]
  run:
    # A command that should start your service.
    start: npm start
```

:::
<!-- markdownlint-enable DOCSMD004 -->
