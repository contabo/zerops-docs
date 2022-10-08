# Configuration: zerops.yml

[[toc]]

## Introduction

Zerops uses a YAML definition file to build your application. This file, `zerops.yml`, has to be placed in your project's root directory.

Each `zerops.yml` can contain definitions for one or more applications. It is especially valuable for a monorepo when one repository contains source code for various applications. The appropriate **service hostname** to which the application will be deployed is used to select a correct definition from the `zerops.yml`.

A definition begins with a **service hostname** to which the application will be deployed and consists of two parts: **build** and **run**. The `build` part is always required. The `run` part is optional except for Zerops services **PHP/Apache**, **Node.js**, and **Golang**.

## Example

```yaml
# Service hostname to which the application will be deployed.
nodejsapp:
  # The part used for the build phase to produce a final application runtime.
  build:
    # Which technology should be used as a base one for creating a build container.
    base: [nodejs@14]
    # Which commands should be run to install additional or custom dependencies.
    prepare:
      - apt-get [options] command [pkg]
      - curl [options] url
      - npm i -g package
    # Which commands should be run to build the application.
    build:
      - npm i
      - npm run build:production
    # Which files or directories of the application build should be stored for the next build.
    cache: [node_modules]
    # Which files and directories should be copied from a build container into a runtime container.
    deploy: [dist, node_modules, package.json]
  # The part used to run the application runtime after a successful deployment.
  run:
    # Which commands should be run to install additional libraries or tools.
    prepare:
      - apt-get [options] command [pkg]
      - curl [options] url
      - npm i -g package
    # Which commands to run after a launch or each restart of a runtime container instance.
    init:
      - <command> <options>
    # Replacement of template patterns in static files with environment variable values.
    envReplace:
      # Specification of files / folders to search for delimiters.
      target: [<file>, <folder>]
      # Specification of one or more delimiters (quote YAML special characters).
      delimiter: [$$, "%%"]
    # A command that should start your service.
    start: npm start
```

## Build part and its properties

This is used for the build phase to produce a final application runtime.

### `base` (optional)

List the Zerops technologies your build uses as the **Zerops build base image** for the build container. **You can skip this if you'd rather install everything yourself or if you don't need any technology at all.** For example, you can look at the Zerops recipe phpPgAdmin [zerops.yml](https://github.com/zeropsio/recipe-phppgadmin/blob/main/zerops.yml), where the build doesn't need any base technology, just a shell script, to prepare the **phpPgAdmin** runtime that is deployed into a Zerops PHP/Apache service.

#### Supported base images

##### `nodejs@<14, 16>`

Includes `npm`, `yarn`, `git`, `npx`.

##### `php@<7.4, 8.0, 8.1>`

Includes `composer`, `git`, `php`.

##### `go@<1>`

Includes `go`, `git`.

#### Example of using multiple base images

```yaml
base: [nodejs@14, go@1]
```

#### Example of using a single base image

```yaml
base: [php@7.4]
```

### `prepare` (optional)

Specify which commands to run to install additional or custom dependencies on top of the Zerops [base](#base-optional) technologies needed by your application build process. The **custom build image** originating from the **Zerops build base image** and the results of **prepare** commands will be used for the next build pipeline, unless:

* the content of the `base` or `prepare` property has been changed,
* you invalidated the existed **custom build image** through the Zerops GUI (not yet implemented).

If one of the above happens, the latest **Zerops build base image** is used, and the **prepare** commands run again.

Application sources are downloaded before the `prepare` phase starts and, therefore, can be accessed by this process.

<!-- markdownlint-disable DOCSMD004 -->
::: tip Using a single/multiple shell instance(s)
The commands in the `prepare` section can be run by the same way as in the [build](#example-using-a-single-shell-instance) section.
:::
<!-- markdownlint-enable DOCSMD004 -->

### `build`

Specify which commands to run to produce the final application runtime deployed into a Zerops service. These commands run during each build process from the first to the last.

#### Example using a single shell instance

This syntax allows you to run all commands one by one in the same environment context. For example, if one command changes the current directory, the next continues with that change. Or, if one command adds an environment variable, the next can also see it.

```yaml
build:
  - |
    npm install
    npm run build
```

#### Example using multiple shell instances

Each command is run in a separate environment context. For example, each shell instance starts in the home directory again. Or, if one command adds an environment variable, it won't be here for the next one.

```yaml
build:
  - npm install
  - npm run build
```

### `cache` (optional)

Allows the defining files or directories of the deployed application runtime stored as a cache for the next build. **The path starts from your project's root directory** (the location of `zerops.yml`). This is to optimize and reduce the build time. The cache that is already stored from a previous build is only updated when the current one is different.

#### Example of the cache using

```yaml
# Allows you to reuse previously installed Node.js dependency modules.
cache: [node_modules]
```

### `deploy`

Determines directories and files produced by your build, which should be deployed to your runtime service container. **The path starts from the root directory of your project** (the location of `zerops.yml`).

<!-- markdownlint-disable DOCSMD004 -->
::: tip Using a tilda character to strip a path directory
Standard behavior is to copy the directories and files with exactly the same path as they are placed in the build container. If you want to strip the path from the left side, use the tilda (**~**) character. You can strip the entire directory path or only its part from the left.
:::
<!-- markdownlint-enable DOCSMD004 -->

#### Examples of different deploy settings

<!-- markdownlint-disable DOCSMD004 -->
::: tip Using quotes
You must enclose the pattern in quotes if the directory path or the file name contains a space.
:::
<!-- markdownlint-enable DOCSMD004 -->

##### Deploys a folder, and a file from the project root directory

```yaml
deploy: [dist, package.json]
```

##### Deploys everything

```yaml
deploy: [.]
```

##### Deploys a single file on a path

```yaml
# Deploys `path/to/file.txt`, including the path directories.
deploy: [./path/to/file.txt]
```

##### Deploys all files in a directory on a path

```yaml
# Deploys everything inside `path/to/dir`, including the path directories.
deploy: [./path/to/dir/]
```

##### Deploys a single file on a path, with the entire path directory stripping

```yaml
# Deploys only `file.txt` without its path directory.
deploy: [./path/to/~/file.txt]
```

##### Deploys a single file on a path, with the partial path directory stripping

```yaml
# Deploys only `to/file.txt`, the partial path directory stripping.
deploy: [./path/~/to/file.txt]
```

##### Deploy all files in a directory, with the entire path directory stripping

```yaml
# Deploys everything inside `path/to` without the path directories.
deploy: [./path/to/~/]
```

##### Deploy all files in a directory, with the partial path directory stripping

```yaml
# Deploys everything inside `path/to` with the partial path directory stripping.
# The deployed result will be the everything inside the `to` directory.
deploy: [./path/~/to/]
```

## Run part and its properties

It is either used to customize one of the Zerops runtime services [Node.js](/documentation/services/runtimes/nodejs.html), [Golang](/documentation/services/runtimes/golang.html), [PHP](/documentation/services/runtimes/php.html), and [Static server Nginx](/documentation/services/static-servers/nginx.html) technology used as a **Zerops runtime image** for the runtime container or specifying some properties specific for a selected Zerops runtime service.

### `prepare` (optional for all services)

Specify which commands to run to install additional libraries or tools on top of the selected Zerops runtime service technology. The **custom runtime image** originating from the **Zerops runtime image** and the results of **prepare** commands will be used as the source of truth for any new runtime container, unless:

* the content of the `prepare` property has been changed,
* you invalidated the existed **custom runtime image** through the Zerops GUI (not yet implemented).

If one of the above happens, the latest **Zerops runtime image** is used, and the **prepare** commands run again.

There is no access to the application sources, the code being deployed, or any previous **custom runtime image** during the `prepare` phase, but only to the **Zerops runtime image**.

<!-- markdownlint-disable DOCSMD004 -->
::: tip Using a single/multiple shell instance(s)
The commands in the `prepare` section can be run by the same way as in the [build](#example-using-a-single-shell-instance) section.
:::
<!-- markdownlint-enable DOCSMD004 -->

### `init` (optional for all services)

Specify which commands to run after a launch or each restart of a runtime container instance and after `prepare` commands (if they exist), for example, initialization or removing a custom application cache.

Each command in the `init` section has to run successfully without any error. Otherwise, the service won't start, and the whole [run](#run-part-and-its-properties) part will be repeated.

<!-- markdownlint-disable DOCSMD004 -->
::: tip Using a single/multiple shell instance(s)
The commands in the `init` part can be run by the same way as in the [build](#example-using-a-single-shell-instance) part.
:::
<!-- markdownlint-enable DOCSMD004 -->

### envReplace (optional for all services)

This allows you to replace marked places in static files of deployed runtime applications (Node.js, Golang, PHP) with environment variable values. Those marked places are combinations of defined delimiters and environment variable keys and represent the searched patterns that should be replaced.

Zerops will execute the pattern search and replace when starting and restarting runtime containers. If a user wants to change the rules for replacing environment variables, a new deployment with an updated `zerops.yml` is needed. If a user only changes the value of an environment variable or makes any other change in the variable list, restarting the service is sufficient for the changes to take effect.

<!-- markdownlint-disable DOCSMD004 -->
::: warning When is the replacement processed?
The process is done immediately before the `run.init` commands.
:::
<!-- markdownlint-enable DOCSMD004 -->

#### target

Specify one or more files/folders in which Zerops searches for combinations of defined delimiters and environment variable keys. If it finds any of them, it replaces the found pattern with a current environment variable value if the corresponding variable key exists.

<!-- markdownlint-disable DOCSMD004 -->
::: warning What is processed in a specified folder?
Only files directly located in the given directory are processed and not in sub-directories.
:::
<!-- markdownlint-enable DOCSMD004 -->

#### delimiter

Specify one or more delimiters that Zerops uses to create the searched patterns in combination with environment variable keys. The delimiter marks both the beginning and the end of the pattern. If the delimiter is one of the following special YAML characters (`{`, `}`, `[`, `]`, `&`, `*`, `#`, `?`, `|`, `-`, `<`, `>`, `=`, `!`, `%`, `@`, `:`, `,`), remember to quote it.

#### Examples of different replacement settings

<!-- markdownlint-disable DOCSMD004 -->
::: tip Using quotes
You must enclose the pattern in quotation marks if the directory path or the file name contains a space.
:::
<!-- markdownlint-enable DOCSMD004 -->

##### Replacing in a single file located in the project root directory

```yaml
# Zerops is looking for any of the $$<environmentVariableKey>$$ patterns in the config.yml file.
# If found, it's replaced by the current value of the <environmentVariableKey> variable when it exists.
envReplace:
  target: config.yml
  delimiter: $$
```

##### Replacing in files or folders, using multiple delimiters

```yaml
# Zerops is looking for any of the $$<environmentVariableKey>$$ or %%<environmentVariableKey>%% patterns
# in the ./path/to/setting.yml file or in any file in the ./path/to/dir directory.
# If found, it's replaced by the current value of the <environmentVariableKey> variable when it exists.
envReplace:
  target: [./path/to/setting.yml, ./path/to/dir/]
  delimiter: [$$, "%%"]
```

### `start` (required only for Node.js or Golang services)

A command to start your service. This command will be executed in each container after finishing all `init` commands. This is only related to the [Node.js](/documentation/services/runtimes/nodejs.html) and [Golang](/documentation/services/runtimes/golang.html) service.

### `documentRoot` (required only for PHP/Apache service)

The value represents the folder name used as the root of the publicly accessible web server content, usually the location of your `index.php`. This is related only to the [PHP/Apache](/documentation/services/runtimes/php.html) service.
