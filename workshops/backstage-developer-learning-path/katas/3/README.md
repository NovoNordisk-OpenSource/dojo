Novo Nordisk Backstage Training - Code Kata #3
======================================

This training exercise is a **beginner-level** course on [Backstage](https://github.com/backstage/backstage/) at Novo Nordisk and serves as a starting point for developers looking to onboard [Backstage.io](https://backstage.io/).

## Getting started
These instructions will help you prepare for the kata and ensure that your training machine has the tools installed you will need to complete the assignment(s). If you find yourself in a situation where one or more tools might not be available for your training environment please reach out to your instructor for assistance on how to proceed, post an [issue in our repository](https://github.com/NovoNordisk-OpenSource/dojo/issues) or fix it yourself and update the kata via a [pull request](https://github.com/NovoNordisk-OpenSource/dojo/pulls).

### Prerequisites
* [Git](https://git-scm.com/book/en/v2/Getting-Started-Installing-Git)
* [NodeJS](https://nodejs.org/)
* [NPM](https://docs.npmjs.com/downloading-and-installing-node-js-and-npm)
* [Docker Desktop](https://docs.docker.com/desktop/)
* [WSL2](https://learn.microsoft.com/en-us/windows/wsl/install) (windows only)

## Exercise
In this exercise we will explore `software templates` in [Backstage](https://github.com/backstage/backstage/) with the goal of creating a new `software template` for a `hello-react` application that can be reused by other teams to generate similar applications easily within our organization.

Upon completion the participants will have gained a basic understanding of the folllowing aspects of the `software template` concept:

* `Software template` configuration.
* Creating a new `software template` for a `hello-react` application.
* `Parameterization` & `customization` our new `software template`.

### 1. Create a kata directory
First we setup a directory for our exercise files. It's pretty straight forward:

```bash
mkdir kata3
cd kata3
```

### 2. `Software template` configuration
[TEXT]

### 3. Creating a new software template
By default `software templates` has the ability to load skeletons of code, template in some variables, and publish the template to some locations like `GitHub` or `Azure DevOps`. Templates are stored in the `software satalog` under a kind `Template`. The minimum that is needed to define a `template` is a `template.yaml` file, but it would be good to also have some files in there that can be templated in to represent the code we want our template to scaffold. In order to achieve this quickly we will use the `npx` CLI to scaffold a simple `hello-react` application:

```bash
npx create-react-app hello-react
cd hello-react
yarn start
```

Once we have verified that the scaffoleded code works as intended, meaning `yarn start` succesfully launches the application, we can start building our `software template` around it to turn it into a reusable component in Backstage. To achieve this, you'll need to create a `template.yaml` file containing a `definition` for our `hello-react-template` wrapper:

```yaml
apiVersion: backstage.io/v1alpha1
kind: Template
metadata:
  name: hello-react-template
  title: Hello React template
  description: Template for Hello React application in Backstage
spec:
  owner: backstage/techdocs-core
  type: service
  # Define parameters or other configurations if needed
  # ...

  # Define file paths to include in the template
  files:
    - path: src/
      template:
        # Define templating variables if necessary
        # ...
    - path: public/
      template:
        # Define templating variables if necessary
        # ...
    # Add other files or directories needed for your template
    # ...
```

***Note*** <br/>
If you're running `Backstage` with `Node 20` or later, you'll need to pass the flag `--no-node-snapshot` to `Node` in order to use the `software templates` feature. One way to do this is to set the environment variable in our local shell: `export NODE_OPTIONS=--no-node-snapshot`.

### 4. Adding input parameters to our new software template
[TEXT]

```yaml
apiVersion: backstage.io/v1alpha1
kind: Template
metadata:
  name: hello-react-template
  description: Template for Hello React application in Backstage
spec:
  parameters:
    - name: appName
      description: Name of the React application
      type: string
      default: hello-react
    - name: authorName
      description: Name of the author
      type: string
      default: John Doe

  files:
    - path: src/
      template:
        # Templating variables used within the files
        - name: appName
          default: '{{ .Values.appName }}'
        - name: authorName
          default: '{{ .Values.authorName }}'
      replaceRules:
        - replace: 'HelloReact'
          with: '{{ .Values.appName }}'

    - path: public/
      template:
        # Templating variables used within the files
        - name: appName
          default: '{{ .Values.appName }}'
        - name: authorName
          default: '{{ .Values.authorName }}'
      replaceRules:
        - replace: 'Hello React'
          with: '{{ .Values.appName }}'

    # Add other files or directories needed for your template
    # ...

  values:
    appName: '{{ .Parameters.appName }}'
    authorName: '{{ .Parameters.authorName }}'
```

### 5. Adding custom build steps to our new software template
[TEXT]

```yaml
apiVersion: backstage.io/v1alpha1
kind: Template
metadata:
  name: hello-react-template
  description: Template for Hello React application in Backstage
spec:
  parameters:
    - name: appName
      description: Name of the React application
      type: string
      default: hello-react
    - name: authorName
      description: Name of the author
      type: string
      default: John Doe

      - title: Fill in some steps
      required:
        - name
      properties:
        name:
          title: Name
          type: string
          description: Unique name of the component
          ui:autofocus: true
          ui:options:
            rows: 5
    - title: Choose a location
      required:
        - repoUrl
      properties:
        repoUrl:
          title: Repository Location
          type: string
          ui:field: RepoUrlPicker
          ui:options:
            allowedHosts:
              - github.com

  steps:
    - id: install-dependencies
      name: Install Dependencies
      description: Install project dependencies
      action: plugin:your-company/yarn-install

    - id: build-app
      name: Build Application
      description: Build the React application
      action: plugin:your-company/yarn-build

    - id: test-app
      name: Test Application
      description: Run tests for the React application
      action: plugin:your-company/yarn-test

  files:
    - path: src/
      template:
        # Templating variables used within the files
        - name: appName
          default: '{{ .Values.appName }}'
        - name: authorName
          default: '{{ .Values.authorName }}'
      replaceRules:
        - replace: 'HelloReact'
          with: '{{ .Values.appName }}'

    - path: public/
      template:
        # Templating variables used within the files
        - name: appName
          default: '{{ .Values.appName }}'
        - name: authorName
          default: '{{ .Values.authorName }}'
      replaceRules:
        - replace: 'Hello React'
          with: '{{ .Values.appName }}'

    # Add other files or directories needed for your template
    # ...

  values:
    appName: '{{ .Parameters.appName }}'
    authorName: '{{ .Parameters.authorName }}'
```

## Want to help make our training material better?
 * Want to **log an issue** or **request a new kata**? Feel free to visit our [GitHub site](https://github.com/NovoNordisk-OpenSource/dojo/issues).
