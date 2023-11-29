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
In this exercise we will explore `software templates` in [Backstage](https://github.com/backstage/backstage/) with the goal of creating a new `software template` for a `hello-react` component that can be reused by other teams to generate similar components easily within our organization.

Upon completion the participants will have gained a basic understanding of the folllowing aspects of the `software template` concept:

* `Software template` configuration.
* Creating a new `software template` for a `hello-react` component.
* Adding `parameters` & `steps` to our new `software template`.

### 1. Create a kata directory
First we setup a directory for our exercise files. This involves copying the kata folder from the previous step to carry over our modifications, thankfully that is very straight forward:

```bash
cd ../
cp kata2 kata3
cd kata3
```

### 2. Creating a new software template
By default `software templates` has the ability to load skeletons of code, template in some variables, and publish the template to some locations like `GitHub` or `Azure DevOps`. Templates are stored in the `software catalog` with kind `Template`. The minimum that is needed to define a `template` is a `template.yaml` file, but it would be good to also have some files in there that can be templated in to represent the code we want our template to scaffold. In order to achieve this quickly we will use the `npx` CLI to scaffold a simple `hello-react` component:

```bash
npx create-react-app hello-react
cd hello-react
yarn start
```

Once we have verified that the scaffoleded code works as intended, meaning `yarn start` succesfully launches the application, we can start building our `software template` around it to turn it into a reusable component in `Backstage`. To achieve this, you'll need to create a `template.yaml` file containing a `definition` for our `hello-react-template` wrapper:

```yaml
apiVersion: backstage.io/v1alpha1
kind: Template
# some metadata about the template itself
metadata:
  name: hello-react-template
  title: Hello React template
  description: Template for Hello React component in Backstage
spec:
  owner: backstage/techdocs-core
  type: component

  # Define file paths to include in the template
  files:
    - path: src/
    - path: public/
```

***Note*** <br/>
If you're running `Backstage` with `Node v20` or later, you'll need to pass the flag `--no-node-snapshot` to `Node` in order to use the `software templates` feature. One way to do this is to set the environment variable in our local shell: `export NODE_OPTIONS=--no-node-snapshot`.

### 3. Add input parameters to our new software template
When enhancing a `Backstage software template` by adding input parameters, the focus lies in augmenting customization and user flexibility during the generation process. Incorporating input parameters involves defining fields within the template structure that prompt users to provide specific values or configurations tailored to their needs. These parameters could encompass various aspects, such as application names, author details, URLs, or any other essential settings crucial for the generated software. By introducing these input parameters effectively, the template becomes more dynamic, enabling users to personalize and fine-tune the generated output according to their requirements, thereby fostering a more adaptable and user-centric developer experience within the `Backstage` platform.

Adding input parameters to our `software template` is as simple as extending our current `template.yaml` with a `parameters` section and mapping the respective input parameters to `template` logic nested inside our various `files.path` elements:

```yaml
apiVersion: backstage.io/v1alpha1
kind: Template
metadata:
  name: hello-react-template
  description: Template for Hello React application in Backstage
spec:
  owner: backstage/techdocs-core
  type: component

  files:
    - path: src/
      template:
        # Templating variables used within the files
        - name: name
          default: '{{ .Values.name }}'
        - name: author
          default: '{{ .Values.author }}'
        - name: repoUrl
          default: '{{ .Values.repoUrl }}'
      replaceRules:
        - replace: 'hello-react'
          with: '{{ .Values.name }}'

    - path: public/
      template:
        # Templating variables used within the files
        - name: name
          default: '{{ .Values.name }}'
        - name: author
          default: '{{ .Values.author }}'
        - name: repoUrl
          default: '{{ .Values.repoUrl }}'
      replaceRules:
        - replace: 'hello-react'
          with: '{{ .Values.name }}'

  # values mapped to files.path.template
  values:
    name: '{{ .Parameters.name }}'
    author: '{{ .Parameters.author }}'
    repoUrl: '{{ .Parameters.repoUrl }}'

  # define parameters which are rendered in Backstage as form input
  parameters:
    - title: Fill in the name of your application
      required:
        - name
      properties:
        name:
          title: Name
          type: string
          description: Unique name of the Hello React application
          default: hello-react
          ui:autofocus: true
          ui:options:
            rows: 5
   - title: Fill in the author of your application
      required:
        - author
      properties:
        name:
          title: Author
          type: string
          description: Author of the Hello React application
          default: John Doe
          ui:autofocus: false
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
```

### 4. Add steps to our new software template
With the input parameters in place we can now proceed to extend our template with a series of `steps`. Within this sequence, you'll initiate by fetching the foundational `template` necessary for the component and proceed to publishing the newly scaffolded `hello-react` component onto a specified `GitHub` repository, utilizing repository details and an `OAuth` token for authentication. Lastly, the sequence concludes by registering the created application within the `Backstage software catalog`, ensuring its comprehensive integration and visibility within the broader `Backstage` ecosystem. However enough words, lets update our `template.yaml` add the `steps` logic:

```yaml
apiVersion: backstage.io/v1alpha1
kind: Template
metadata:
  name: hello-react-template
  description: Template for Hello React application in Backstage
spec:
  owner: backstage/techdocs-core
  type: component

  files:
    - path: src/
      template:
        - name: name
          default: '{{ .Values.name }}'
        - name: author
          default: '{{ .Values.author }}'
        - name: repoUrl
          default: '{{ .Values.repoUrl }}'
      replaceRules:
        - replace: 'hello-react'
          with: '{{ .Values.name }}'

    - path: public/
      template:
        - name: name
          default: '{{ .Values.name }}'
        - name: author
          default: '{{ .Values.author }}'
        - name: repoUrl
          default: '{{ .Values.repoUrl }}'
      replaceRules:
        - replace: 'hello-react'
          with: '{{ .Values.name }}'

  values:
    name: '{{ .Parameters.name }}'
    author: '{{ .Parameters.author }}'
    repoUrl: '{{ .Parameters.repoUrl }}'

  parameters:
    - title: Fill in the name of your application
      required:
        - name
      properties:
        name:
          title: Name
          type: string
          description: Unique name of the Hello React application
          default: hello-react
          ui:autofocus: true
          ui:options:
            rows: 5
   - title: Fill in the author of your application
      required:
        - author
      properties:
        name:
          title: Author
          type: string
          description: Author of the Hello React application
          default: John Doe
          ui:autofocus: false
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
  
  # define steps that are executed sequentially in the scaffolder backend
  steps:
    - id: fetch-base
      name: Fetch Base
      action: fetch:template
      input:
        url: ./template
        values:
          name: ${{ parameters.name }}

    - id: publish
      name: Publish
      action: publish:github
      input:
        allowedHosts: ['github.com']
        description: This is ${{ parameters.name }}
        repoUrl: ${{ parameters.repoUrl }}
        # backstage users oauth token
        token: ${{ secrets.USER_OAUTH_TOKEN }}

    - id: register
      name: Register
      action: catalog:register
      input:
        repoContentsUrl: ${{ steps['publish'].output.repoContentsUrl }}
        catalogInfoPath: '/catalog-info.yaml'
```

### 5. Registering software templates with our local Backstage instance
TODO

## Want to help make our training material better?
 * Want to **log an issue** or **request a new kata**? Feel free to visit our [GitHub site](https://github.com/NovoNordisk-OpenSource/dojo/issues).
