Novo Nordisk Backstage Training - Code Kata #2
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
This exercise is designed to help developers gain familiarity with the `software catalog` in [Backstage](https://github.com/backstage/backstage/) and will build on the previous exercises to help deepen their understanding of how to manage and utilize this key feature. As this exercise will only cover the basics, participants are encouraged to extend on the subject by reading the upstream documentation [here](https://backstage.io/docs/features/software-catalog/).

Once completed the participants will have gained a basic understanding of the following aspects of the `software catalog`:

* Configuring the software catalog.
* Registering APIs and components within the software catalog.
* Implement metadata, such as labels, descriptions, and versions, for software entities.
* Extending the software catalog with custom software entities.

### 1. Create a kata directory
First we setup a directory for our exercise files. This involves copying the kata folder from the previous step to carry over our modifications, thankfully that is very straight forward:

```bash
cd ../
cp kata1 kata2
cd kata2
```

### 2. Configuring the software catalog
Backstage `software catalog` configuration is managed in `app-config.yaml` under the `catalog` configuration element. The simplest configuration, as shown in the default `@backstage/create-app template`, is to declaratively add `locations` pointing to `YAML` files with static configuration:

```yaml
catalog:
  locations:
    - type: url
      target: https://github.com/backstage/backstage/blob/master/packages/catalog-model/examples/components/artist-look
```

The `url` type locations are handled by the `UrlReaderProcessor` that is bundled with `Backstage`, thus no `processor` configuration is needed. This `processor` does however need an integration to understand how to retrieve a given URL. For the example above, you would need to configure the [GitHub integration](https://backstage.io/docs/integrations/github/locations) to read files from github.com.

In addition to `url` locations, you can use the `file` location type to bring in content from the local file system. You should only use this for local development, test setups, and example data, not for production data. You are also not able to use placeholders in them like `$text`. You can however reference other files relative to the current file, as follows:

```yaml
catalog:
  locations:
    - type: url
      target: https://github.com/backstage/backstage/blob/master/packages/catalog-model/examples/components/artist-look
    - type: file
      target: ../../examples/all.yaml
```

By default, the `catalog` will only allow the ingestion of `entities` with the kind `Component`, `API`, and `Location`. In order to allow entities of other kinds to be added, you need to add `rules` to the catalog. `Rules` are added either in a separate `catalog.rules` key or added to statically configured locations:

```yaml
catalog:
  rules:
    - allow: [Component, API, Location, Template]

  locations:
    - type: url
      target: https://github.com/backstage/backstage/blob/master/packages/catalog-model/examples/components/artist-look
      rules:
        - allow: [Group]
    - type: file
      target: ../../examples/all.yaml
      rules:
        - allow: [API]
```

***Note*** <br/>
Keep in mind that paths in our configration file are relative to the `package` executing a given piece of code, thus in order for us to step out of the `backend package` and into the file system we need to use `../../` to go up two levels from the current execution path, which is typically `packages/backend/`. For more information about configration of the software catalog please go [here](https://backstage.io/docs/conf/)

### 3. Working with entities
In `Backstage`, `entities` represent different types of `metadata` about software teams, APIs, and documentation that can be discovered via the `software catalog`.

YAML schemas in `Backstage` define the `metadata` associated with various components and provide a standardized way to describe and manage these entities within the `software catalog`. Here's a brief overview of some common `entities` and their YAML schemas:

```yaml
apiVersion: backstage.io/v1alpha1
kind: Group
metadata:
  name: string
  description?: string
  annotations?: {}
spec:
  /* group-specific properties */
```

```yaml
apiVersion: backstage.io/v1alpha1
kind: User
metadata:
  name: string
  description?: string
  annotations?: {}
spec:
  /* user-specific properties */
```

```yaml
apiVersion: backstage.io/v1alpha1
kind: Domain
metadata:
  name: string
  description?: string
  annotations?: {}
spec:
  type: string
  lifecycle?: string
  owner?: string
  /* Other domain-specific properties */
```

```yaml
apiVersion: backstage.io/v1alpha1
kind: System
metadata:
  name: string
  description?: string
  annotations?: {}
spec:
  type: string
  lifecycle?: string
  owner?: string
  /* Other system-specific properties */
```

```yaml
apiVersion: backstage.io/v1alpha1
kind: Component
metadata:
  name: string
  description?: string
  annotations?: {}
spec:
  type: string
  lifecycle?: string
  owner?: string
  techStack?: []
  /* Other component-specific properties */
```

```yaml
apiVersion: backstage.io/v1alpha1
kind: API
metadata:
  name: string
  description?: string
  annotations?: {}
spec:
  type: string
  lifecycle?: string
  owner?: string
  /* Other api-specific properties */
```

```yaml
apiVersion: backstage.io/v1alpha1
kind: Resource
metadata:
  name: string
  description?: string
  annotations?: {}
spec:
  type: string
  lifecycle?: string
  owner?: string
  /* Other resource-specific properties */
```

The above schemas provide a baseline to define and manage `descriptors` for a given `capability`, which in turn makes it easier for teams to create, organize, and share information about their software assets within the `Backstage` platform. For more information on how to perform [substitition](https://backstage.io/docs/features/software-catalog/descriptor-format/#substitutions-in-the-descriptor-format) or [extend the model](https://backstage.io/docs/features/software-catalog/extending-the-model) a moment to browse the linked documentation.

### 4. Working with metadata
TODO: https://backstage.io/docs/features/software-catalog/well-known-annotations
TODO: https://backstage.io/docs/features/software-catalog/well-known-relations
TODO: https://backstage.io/docs/features/software-catalog/well-known-statuses

### 5. Creating a new Component definition
TODO: https://backstage.io/docs/features/software-catalog/descriptor-format#overall-shape-of-an-entity
TODO: https://backstage.io/docs/features/software-catalog/extending-the-model

### 6. Registering our new Component definition
TODO: https://backstage.io/docs/features/software-catalog/software-catalog-api

## Want to help make our training material better?
 * Want to **log an issue** or **request a new kata**? Feel free to visit our [GitHub site](https://github.com/NovoNordisk-OpenSource/dojo/issues).
