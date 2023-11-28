Novo Nordisk Backstage Training - Code Kata #1
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
This instructional exercise is meticulously designed to serve as a comprehensive guide, taking you step-by-step through the intricate process of installing and setting up [Backstage](https://github.com/backstage/backstage/) within the confines of a localized environment. Its purpose is not solely limited to installation; it also aims to provide a detailed walkthrough that acquaints you with the nuanced functionalities and intricacies of the solution, thereby enabling you to gain a comprehensive understanding of its multifaceted capabilities.

Moreover, this exercise doesn't merely stop at installation and CLI usage; it also extends to hands-on demonstrations that meticulously illustrate the foundational aspects of navigating within the intricate Backstage platform. By offering practical demonstrations and insights into basic features, this exercise aims to foster a profound understanding, thereby laying the groundwork for subsequent explorations and expansions into the extensive capabilities and versatile plugins that [Backstage](https://github.com/backstage/backstage/) encompasses. This initial foray into [Backstage](https://github.com/backstage/backstage/) serves as a pivotal stepping stone, providing a robust foundational understanding that is essential for harnessing the full spectrum of its capabilities and the multitude of possibilities it offers through its diverse range of plugins.

### 1. Create a kata directory
First we setup a directory for our exercise files. It's pretty straight forward:

```
mkdir kata1
cd kata1
```

### 2. Install Backstage via NPM
Next we start diving into the [Backstage](https://github.com/backstage/backstage/) ecosystem by installing [Backstage](https://github.com/backstage/backstage/) using `npx` and `yarn`. Once `npx` completes the scaffolding process for your new developer portal, navigate into the freshly generated app directory. From here, fire up the engine of your application using the `yarn start` command to initiate and configure the required dependencies:

```bash
npx @backstage/create-app@latest
cd your-backstage-app-name
yarn start
```

***Note*** <br/>
Its worth noting that once Backstage is installed we can access the CLI directly via `yarn`. E.g. `yarn backstage-cli repo test`

### 3. Test installation
After completing the scaffolding process, it's essential to ensure the integrity of your setup by running the `test` command. This step confirms not only the successful installation of NPM dependencies but also validates the code generation to guarantee a robust foundation for your project.

Execute the following command to conduct the tests:

```bash
yarn test:all
```

The test command will trigger a suite of pre-configured tests, examining various aspects of your project's functionality and dependencies. This verification process aids in identifying any potential issues early on, ensuring that your project operates smoothly. Upon completion, review the test output in the console for any reported errors or warnings, enabling you to address them promptly.

***Note*** <br/>
You can find more information on how to debug Backstage [here](https://backstage.io/docs/local-dev/debugging)

### 4. Configure persistant storage
[Backstage](https://github.com/backstage/backstage/) uses an `:inmemory:` provider for backend persistancy out-of-the box. While this is great for getting us started it becomes somewhat cumbersome to work with if the application resets itself all the time. To fix this issue we will swap from [`SQLite`](https://www.sqlite.org/index.html) to [`postgres`](https://www.postgresql.org/) by editing the `backend.database` configuration element in the `app-config.yaml` file that can be found in the root of the folder containing the scaffolded assets generated via the Backstage CLI.

Once we have located the configuration element in `app-config.yaml`, we can edit it from this:

```yaml
backend:
  database:
    client: better-sqlite3
    connection: ':memory:'
```

To this:

```yaml
backend:
  database:
    client: pg
    connection:
      host: 'my_postgres'
      port: 1234
      user: 'admin'
      password: 'supersecretpassword'
```

***Note*** <br/>
Make sure you have configured the required environment variables correctly to target your local `postgres` installation.

### 5. Initialize a local postgres container
Once we have the new configuration in place for our persistant storage, we need to initializ a postgres container on our local machine to match the configuration:

```bash
docker run --name my_postgres -e POSTGRES_PASSWORD=supersecretpassword -e POSTGRES_USER=admin -e POSTGRES_HOST=1234 -e POSTGRES_PORT=1234 -d postgres
```

### 6. Restart Backstage
Give the `postgres` database a minute to get ready while you terminate the current [Backstage](https://github.com/backstage/backstage/) instance and allow the database services to come online inside our `my_postgres` container. Once the shell running the `postgres` workload prints messges indicating that the database is ready for external connections restart [Backstage](https://github.com/backstage/backstage/) using the `start` command as follows:

```bash
yarn start
```

### 7. Verify configuration via Backstage Dashboard
Once the reconfigured [Backstage](https://github.com/backstage/backstage/) process is up and running again, open your web browser and go to http://localhost:3000 to verify that your local Backstage instance is servicing connections. This will take you to the Backstage dashboard displaying various cards such as Catalog, Explore, Create, and more.

Check the various features are working and that your developer tools (F11) does not report `CSP` or `CORS` errors.

***Note*** <br/>
You can find more information on how to customize the GUI theme [here](https://backstage.io/docs/getting-started/app-custom-theme)

## Want to help make our training material better?
 * Want to **log an issue** or **request a new kata**? Feel free to visit our [GitHub site](https://github.com/NovoNordisk-OpenSource/dojo/issues).
