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
The objective of this exercise is to guide developers through the process of setting up the `Backstage TechDocs` plugin. It will built on the first `code kata` to extend our `Backstage` application and explore the different `TechDocs` configuration options such as specifying publishing behaviours.

Once completed the participants will have gained a basic understanding of the following aspects of `TechDocs`:

* `TechDocs` configuration (`app package` & `backend package`)
* `TechDocs` documentation (`catalog-info.yaml` & `mkdocs.yml` )
* `TechDocs` AddOns Framework
* `TechDocs` publishing options

### 1. Create a kata directory
First we setup a directory for our exercise files. This involves copying the kata folder from the previous step to carry over our modifications, thankfully that is very straight forward:

```bash
cd ../
cp kata2 kata3
cd kata3
```

### 2. Activate TechDocs frontend (optional)
Next we have to ensure that the frontend plugin of `TechDocs` is activate by ensuring that our `packages/app` includes the `TechDocs plugin` assets. If the assets are not bundled in your current version of `Backstage`, the procedure involes interfacing with the `packages/app/src/App.tsx` asset and installing a `@backstage/plugin-techdocs` package via `yarn` by navigating to the `packages/app` directory and executing the following command:

```bash
# From your Backstage root directory
yarn add --cwd packages/app @backstage/plugin-techdocs
```

Once the package has been succesfully installed, we need to import the plugin into the app. Open up `packages/app/src/App.tsx`, import `custom elements` and add new `Route` entries to enable navigation inside the `Backstage` frontend application, which leave our `packages/app/src/App.tsx` looking like this:

```typescript
import {
  DefaultTechDocsHome,
  TechDocsIndexPage,
  TechDocsReaderPage,
} from '@backstage/plugin-techdocs';

const AppRoutes = () => {
  <FlatRoutes>
    <Route path="/docs" element={<TechDocsIndexPage />}>
      <DefaultTechDocsHome />
    </Route>
    <Route
      path="/docs/:namespace/:kind/:name/*"
      element={<TechDocsReaderPage />}
    />
  </FlatRoutes>;
};
```

### 3. Activate TechDocs backend (optional)
With the frontend plugin activated we need to repeat the process for our `packages/backend` assets to ensure that it is correctly configured. If the assets are not bundled in your current version of Backstage, the procedure involes creating a new asset at the following location `packages/backend/src/plugins/techdocs.ts` and installing the `@backstage/plugin-techdocs` via `yarn` by navigating to the `packages/backend` directory and executing the following command:

```bash
# From your Backstage root directory
yarn add --cwd packages/backend @backstage/plugin-techdocs-backend
```

Once the package has been installed, we need to create a new `techdocs.ts` asset and add the following logic:

```typescript
import { DockerContainerRunner } from '@backstage/backend-common';
import {
  createRouter,
  Generators,
  Preparers,
  Publisher,
} from '@backstage/plugin-techdocs-backend';
import Docker from 'dockerode';
import { Router } from 'express';
import { PluginEnvironment } from '../types';

export default async function createPlugin(
  env: PluginEnvironment,
): Promise<Router> {
  // Preparers are responsible for fetching source files for documentation.
  const preparers = await Preparers.fromConfig(env.config, {
    logger: env.logger,
    reader: env.reader,
  });

  // Docker client (conditionally) used by the generators, based on techdocs.generators config.
  const dockerClient = new Docker();
  const containerRunner = new DockerContainerRunner({ dockerClient });

  // Generators are used for generating documentation sites.
  const generators = await Generators.fromConfig(env.config, {
    logger: env.logger,
    containerRunner,
  });

  // Publisher is used for
  // 1. Publishing generated files to storage
  // 2. Fetching files from storage and passing them to TechDocs frontend.
  const publisher = await Publisher.fromConfig(env.config, {
    logger: env.logger,
    discovery: env.discovery,
  });

  // checks if the publisher is working and logs the result
  await publisher.getReadiness();

  return await createRouter({
    preparers,
    generators,
    publisher,
    logger: env.logger,
    config: env.config,
    discovery: env.discovery,
    cache: env.cache,
  });
}
```

Lastly we need to register the newly created `techdocs.ts` asset for usage in the backend application. Thankfully that is simply adding a new entry to the `packages/backend/src/index.ts` file, so that it ends up looking like this:

```typescript
import techdocs from './plugins/techdocs';

// .... main should already be present.
async function main() {
  // ... other backend plugin envs
  const techdocsEnv = useHotMemoize(module, () => createEnv('techdocs'));

  // ... other backend plugin routes
  apiRouter.use('/techdocs', await techdocs(techdocsEnv));
}
```

***Note*** <br/>
You can find more information on how to configure the `TechDocs` plugins [here](https://backstage.io/docs/features/techdocs/configuration)

### 4. Exploring TechDocs AddOns
Enhancing our pages with additional embellishments adds a delightful touch to the developer experience. Imagine the convenience of having a link readily available, seamlessly directing you to a new issue page whenever specific text in your documentation is highlighted. Isn't that an intriguing concept? Let's delve into mastering this technique by harnessing the capabilities of `TechDocs Addons`!

The `TechDocs Addon framework` offers the ability to incorporate [React](https://react.dev/) components directly into your documentation pages. These `addons` can be sourced from any `Backstage` plugin, providing a wide spectrum of possibilities. The framework itself is accessed through the exportation of the `@backstage/plugin-techdocs-react` package. Furthermore, within the `@backstage/plugin-techdocs-module-addons-contrib` package, resides a ready-to-use `<ReportIssue /> Addon`. You can leverage this feature once you've successfully installed two essential dependencies:

```bash
yarn add --cwd packages/app @backstage/plugin-techdocs-react
yarn add --cwd packages/app @backstage/plugin-techdocs-module-addons-contrib
```

Once the required packages has been succesfully installed, we need to import the new plugins into our app by extending `packages/app/src/App.tsx` with two additional `imports` and some `custom elements`, to look like this:

```typescript
import {
  DefaultTechDocsHome,
  TechDocsIndexPage,
  TechDocsReaderPage,
} from '@backstage/plugin-techdocs';
import { TechDocsAddons } from '@backstage/plugin-techdocs-react';
import { ReportIssue } from '@backstage/plugin-techdocs-module-addons-contrib';

const AppRoutes = () => {
  <FlatRoutes>
    {/* ... other plugin routes */}
    <Route path="/docs" element={<TechDocsIndexPage />}>
      <DefaultTechDocsHome />
    </Route>
    <Route
      path="/docs/:namespace/:kind/:name/*"
      element={<TechDocsReaderPage />}
    >
     <TechDocsAddons>
        <ReportIssue />
      </TechDocsAddons>
    </Route>
  </FlatRoutes>;
};
```

***Note*** <br/>
You can find more information on how to work with `TechDocs AddOns` [here](https://backstage.io/docs/features/techdocs/addons)

### 5. Understanding the role of mkdocs.yml
`TechDocs` is founded on the principle of integrating documentation with the code, aka `docs-as-code`, advocating for proximity between the two. Within your Backstage app, a suite of software templates comes preloaded. Each template is equipped with the essentials required to initiate your `TechDocs` platform and commence documentation creation. In the absence of of a software template and/or `catalog-info.yaml` we can manually implement `TechDocs` into any repository by creating a new asset in the root of the target repository named `mkdocs.yml` with the following content:

```yaml
site_name: 'example-docs'

nav:
  - Home: index.md

plugins:
  - techdocs-core
```

To better understand how nav elements are resolved take a minute to read the documentation on [docs_dir](https://www.mkdocs.org/user-guide/configuration/#docs_dir) to understand how `mkdocs` resolves `paths` relative to the `docs_dir`, rather then the parent folder of `mkdocs.yml`.

***Note*** <br/>
Keep in mind that for `Backstage` to be able to access external repositories in [Azure DevOps](https://backstage.io/docs/integrations/azure/locations) and/or [GitHub](https://backstage.io/docs/integrations/github/locations) you will need to configure the respective integations in your `app-config.yaml`.

### 6. Understanding the role of catalog-info.yml
The `catalog-info.yaml` file in `Backstage` serves as a fundamental descriptor for `components`, `services`, or `plugins` registered in the `software catalog`. This file contains essential metadata defining the `root entity`, such as its name, description, type, owner, tags, and other relevant details. This metadata is crucial for users to discover, search, and comprehend various services, components, or plugins available within Backstage.

Moreover, `catalog-info.yaml` promotes consistency by standardizing metadata structure, facilitating seamless integration and providing a foundation for customization and extensibility according to specific user requirements. Ultimately, it acts as a central repository for crucial information, aiding in the visibility, organization, and management of entities within the Backstage catalog.

For the purpose of this exercise we will only focus on a minimalistic `catalog-info.yaml` to illustrate how `TechDocs` hooks into the overall scheme of `catalog-info.yaml` via `annotations`. In later exercises we will revisit this VERY important asset to explore its many aspects, but for now we will simple create a barebones `catalog-info.yaml` with a simple payload:

```yaml
apiVersion: backstage.io/v1alpha1
kind: Component
metadata:
  name: demo-api
  description: Best API ever
  annotations:
    backstage.io/techdocs-ref: dir:.
spec:
  type: service
  lifecycle: production
  owner: demo-api-team
  system: demo-api-portal
```

***Note*** <br/>
To better understand the different `Backstage annotations` feel free to visit the documentation [here](https://backstage.io/docs/features/software-catalog/well-known-annotations#annotations)

### 7. TechDocs publishing
`TechDocs` uses an internal `Publisher` implementation to manage the publication of documentation, this feature exposes some basic configuration options via `app-config.yaml` that governs the overall code generation, building and publishing behaviour(s).

To get started quickly we will set `techdocs.builder` in `app-config.yaml` to `local` so that `TechDocs` is responsible for generating documentation sites directly on the `Backstage` local machine. If set to `external`, `Backstage` will assume that the sites are being generated in each entity's [CI/CD pipeline](https://backstage.io/docs/features/techdocs/configuring-ci-cd), and are being stored in an external repository ready to be imported into the `Backstage postgres` database via an appropriate `catalog-info.yaml`. Once configured our `app-config.yaml` should contain a configuraton section named `techdocs` with the following content:

```yaml
techdocs:
  builder: 'local'
  publisher:
    type: 'local'
```

The `TechDocs` backend plugin runs a docker container with `mkdocs` installed to generate the frontend of the docs from source files ([markdown](https://www.markdownguide.org/)). If you are deploying `Backstage` using `Docker`, this means your application container will try to mount the `local docker daemon` and run another container internally to build the `HTML` assets it needs for publication.

To avoid this causing problems during local development, we can set `techdocs.generator.runIn` in our `app-config.yaml` to `local` to instruct the `techdocs generator` that it should run `mkdocs` locally and not via `Docker`, which leaves our updated configuration section looking like this:

```yaml
techdocs:
  builder: 'local'
  publisher:
    type: 'local'
  generator:
    runIn: local
```

***Note*** <br/>
To better understand the inner workings on `TechDocs` generation, buillding and publishing we encourage participants to spend some personal time exploring the `TechDocs` architecture at the following [location](https://backstage.io/docs/features/techdocs/architecture)

### 8. Registering software templates via the catalog-import plugin
With our `TechDocs` in place we need to add it to the `software catalog`, however to accomplish this we need to power up our local backstage instance via `yarn`:

```bash
yarn dev
```

Once this is done we can register our `TechDocs` via the `catalog-import` plugin, which unless configured differently should be running on `/catalog-import`. To do so simply open a browser once backstage is up and running, navigate to the newly created `Backstage` instance and upload the new `software catalog template`.


## Want to help make our training material better?
 * Want to **log an issue** or **request a new kata**? Feel free to visit our [GitHub site](https://github.com/NovoNordisk-OpenSource/dojo/issues).
