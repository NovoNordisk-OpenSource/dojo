Novo Nordisk Backstage Training - Code Kata #4
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
In this exercise we will explore `software templates` in [Backstage](https://github.com/backstage/backstage/) with the goal of creating a new `software template` for a `hello-react` template that can be reused by other teams to generate websites easily within our organization.

Upon completion the participants will have gained a basic understanding of the folllowing aspects of the `software template` concept:

* `Software template` configuration.
* Creating a new `software template` for a `hello-react` template.
* Adding `parameters` & `steps` to our new `software template`.

### 1. Create a kata directory
First we setup a directory for our exercise files. This involves copying the kata folder from the previous step to carry over our modifications, thankfully that is very straight forward:

```bash
cp kata3 kata4
cd kata4
cd your-backstage-app-name
```

### 2. Initialize Git repository
The `git init` command serves as the foundational step in initializing a new `Git` repository within a directory. It initializes an empty `Git` repository, creating the necessary data structures and configuration files, including the `.git` directory, which houses the repository's metadata and versioning information. This command marks the starting point for version control, enabling tracking of changes and facilitating collaboration:

```bash
git init
```

### 3. Creating a new software template
By default `software templates` has the ability to load skeletons of code, template in some variables, and publish the template to some locations like `GitHub` or `Azure DevOps`. Templates are stored in the `software catalog` with kind `Template`. The minimum that is needed to define a `template` is a `template.yaml` file, but it would be good to also have some files in there that can be templated in to represent the code we want our template to scaffold. In order to achieve this quickly we will use the `npx` CLI to scaffold a simple `hello-react` template:

```bash
npx create-react-app hello-react
cd hello-react
yarn start
```

Once we have verified that the scaffoleded code works as intended, meaning `yarn start` succesfully launches the application, we can start building our `software template` around it to turn it into a reusable component in `Backstage`. To achieve this, you'll need to create a `template.yaml` file containing a `definition` for our `hello-react` template:

```bash
cd..
cat > template.yaml
```

Then we can open the file and start filling in the logic:

```yaml
apiVersion: scaffolder.backstage.io/v1beta3
kind: Template
# some metadata about the template itself
metadata:
  name: hello-react
  title: Hello React template
  description: Template for Hello React website
  tags:
    - react
    - cra
spec:
  owner: backstage/techdocs-core
  type: website
```

***Note*** <br/>
If you're running `Backstage` with `Node v20` or later, you'll need to pass the flag `--no-node-snapshot` to `Node` in order to use the `software templates` feature. One way to do this is to set the environment variable in our local shell: `export NODE_OPTIONS=--no-node-snapshot`.

### 4. Add input parameters to our new software template
When enhancing a `Backstage software template` by adding input parameters, the focus lies in augmenting customization and user flexibility during the generation process. Incorporating input parameters involves defining fields within the template structure that prompt users to provide specific values or configurations tailored to their needs. These parameters could encompass various aspects, such as application names, author details, URLs, or any other essential settings crucial for the generated software. By introducing these input parameters effectively, the template becomes more dynamic, enabling users to personalize and fine-tune the generated output according to their requirements, thereby fostering a more adaptable and user-centric developer experience within the `Backstage` platform.

Adding input parameters to our `software template` is as simple as extending our current `template.yaml` with a `parameters` section and mapping the respective input parameters to `template` logic:

```yaml
apiVersion: scaffolder.backstage.io/v1beta3
kind: Template
metadata:
  name: hello-react
  title: Hello React template
  description: Template for Hello React website
  tags:
    - react
    - cra
spec:
  owner: backstage/techdocs-core
  type: website
  parameters:
    - title: Provide some simple information
      required:
        - component_id
        - owner
      properties:
        component_id:
          title: Name
          type: string
          description: Unique name of the component
          ui:field: EntityNamePicker
        description:
          title: Description
          type: string
          description: Help others understand what this website is for.
        owner:
          title: Owner
          type: string
          description: Owner of the component
          ui:field: OwnerPicker
          ui:options:
            allowedKinds:
              - Group
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

### 5. Add steps to our new software template
With the input parameters in place we can now proceed to extend our template with a series of `steps`. Within this sequence, you'll initiate by fetching the foundational `template` necessary for the component and proceed to publishing the newly scaffolded `hello-react` website onto a specified `GitHub` repository, utilizing repository details and an `OAuth` token for authentication. Lastly, the sequence concludes by registering the created application within the `Backstage software catalog`, ensuring its comprehensive integration and visibility within the broader `Backstage` ecosystem. However enough words, lets update our `template.yaml` add the `steps` logic:

```yaml
apiVersion: scaffolder.backstage.io/v1beta3
kind: Template
metadata:
  name: hello-react
  title: Hello React template
  description: Template for Hello React website
  tags:
    - react
    - cra
spec:
  owner: backstage/techdocs-core
  type: website
  parameters:
    - title: Provide some simple information
      required:
        - component_id
        - owner
      properties:
        component_id:
          title: Name
          type: string
          description: Unique name of the component
          ui:field: EntityNamePicker
        description:
          title: Description
          type: string
          description: Help others understand what this website is for.
        owner:
          title: Owner
          type: string
          description: Owner of the component
          ui:field: OwnerPicker
          ui:options:
            allowedKinds:
              - Group
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
    - id: template
      name: Fetch hello-react template
      action: fetch:template
      input:
        url: ./hello-react
        copyWithoutRender:
          - .github/workflows/*
        values:
          component_id: ${{ parameters.component_id }}
          description: ${{ parameters.description }}
          destination: ${{ parameters.repoUrl | parseRepoUrl }}
          owner: ${{ parameters.owner }}

    - id: publish
      name: Publish
      action: publish:github
      input:
        allowedHosts:
          - github.com
        description: This is ${{ parameters.component_id }}
        repoUrl: ${{ parameters.repoUrl }}

    - id: register
      name: Register
      action: catalog:register
      input:
        repoContentsUrl: ${{ steps.publish.output.repoContentsUrl }}
        catalogInfoPath: "/catalog-info.yaml"

  output:
    links:
      - title: Repository
        url: ${{ steps.publish.output.remoteUrl }}
      - title: Open in catalog
        icon: catalog
        entityRef: ${{ steps.register.output.entityRef }}
```

### 6. Publish software template
Publishing a `software template` with `git push` involves several steps to ensure its availability and usability by others. Once the `template` is crafted or modified, committing the changes locally using `Git` is the initial step. Then, upon ensuring that the remote repository is correctly set as the destination, employing `git push` pushes the committed changes to the remote repository. With proper permissions and access, this action enables the template to become accessible to collaborators or users who can utilize it for their own `Backstage` projects. This publication process via `git push` streamlines the distribution and availability of the `software template`, allowing others to easily integrate it into their development workflows and contribute enhancements or modifications as needed.

```bash
git add .
git commit -m "feat:template"  
git branch -M main
git remote add origin https://github.com/username/hello-react.git
git push -u origin main
```

### 7. Registering software templates via app-config.yaml
While we can import our `software template` using the same approach as our `TechDocs code kata`, for the sake of completeness we will register our new `template` with `Backstage` using the `app-config.yaml`:

```yaml
catalog:
  locations:
    - type: url
      target: https://github.com/novonordisk-opensource/templates/template.yaml # Sample URL location, is NOT valid :)
      rules:
        - allow: [Template]
    - type: file
      target: template.yaml # Sample file location, Backstage will expect the file to be in packages/backend/template.yaml
```

## Want to help make our training material better?
 * Want to **log an issue** or **request a new kata**? Feel free to visit our [GitHub site](https://github.com/NovoNordisk-OpenSource/dojo/issues).
