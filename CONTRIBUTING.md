# Contributor Guide

Information for contributors to the Docs AI Chatbot project.

## Project Structure

The project is structured as a monorepo, with all sub-projects using TypeScript.
We use [Lerna](https://lerna.js.org/) to manage our monorepo.

The project has the following main sub-projects, each of which correspond to a JavaScript module:

- `chat-server`: Express.js server that performs chat functionality. RESTful API.
- `ingest`: CLI application that takes data from data sources and converts it to `embedded_content`
  used by Atlas Vector Search.
- `chat-ui`: React component for interfacing with the `chat-server`.
- `chat-core`: Common resources shared across other sub-projects.
- `mongodb-atlas`: Collection of scripts related to managing the MongoDB Atlas deployment used by the project.
- `scripts`: Miscellaneous scripts to help with the project.

## Bootstrapping

To install all dependencies and build all sub-projects, run the following in the root of your project:

```sh
npm install
npm run bootstrap
```

To get the relevant environment variables for the `.env` files, ask a current project contributor.

You can run both the chat server and UI by running the following from the root of the project:

```sh
npm run dev
```

## Infrastructure

The project uses Drone for its CI/CD pipeline. All drone config is located in `.drone.yml`.

Applications are deployed on Kubernetes using the Kanopy developer platform.
Kubernetes/Kanopy configuration are found in the `<deployed project>/environments`
directories. Refer to the Kanopy documentation for more information
on the Kanopy environment files.

The applications are containerized using docker. Docker files are named
with the `*.dockerfile` extension.

## Releases

### Staging

We run a staging server that uses the latest commit on the `main` branch. When
you merge new commits into `main`, a CI/CD pipeline automatically builds and
publishes the updated staging server and demo site.

### QA Server & Demo Site

We run a QA server that serves a specific build for testing before we release to
production.

To publish to QA:

1. Check out the `qa` branch and pull any upstream changes. Here, `upstream` is
   the name of the `mongodb/docs-chatbot` remote repo.

   ```sh
   git fetch upstream
   git checkout qa
   git pull upstream qa
   ```

2. Apply any commits you want to build to the branch. In many cases you'll just
   build from the same commits as `main`. However, you might want to QA only a
   subset of commits from `main`.

3. Add a tag to the latest commit on the `qa` branch using the following naming scheme: `chat-server-qa-<Build ID>`

   ```
   git tag chat-server-qa-0.0.42 -a
   ```

4. Push the branch to this upstream GitHub repo

   ```sh
   git push upstream qa
   ```

Once you've added the tag to the upstream repo, the Drone CI automatically
builds and deploys the branch to the QA server.

### Production Deployments

We use a tool called `release-it` to prepare production releases for the
`chat-server`, `ingest`, and `chat-ui` projects.

Production releases are triggered by creating a git tag prefaced with the
package name (e.g. `chat-server-v{version-number}`).

To create a new production release:

1. Pull latest code you want to release.

   ```sh
   git pull upstream main
   ```

2. In the relevant package directory (e.g `chat-server`) run the release
   command. This gets the package ready for a new release.

   ```sh
   npm run release
   ```

   When prompted create a draft Github release. The URL for the release draft is
   present in the output of CLI operation. You can use this later.

3. Create a pull request for the branch. Get it reviewed using the standard
   review process.

4. Once the PR is approved and merged, publish the draft release. You can find
   the release draft in the draft tag:
   <https://github.com/mongodb/docs-chatbot/releases>.

When the release is published, the Drone CI picks up the corresponding git tag
and then automatically builds and deploys the branch to production.

## Manage Secrets

### Local

For local development, manage secrets using `.env` files. Wherever you need a `.env` file,
there's a `.env.example` file with the fields you need to add values for.

### CI

For our CI, we use [Drone](https://docs.drone.io/). You can manage Drone secrets using the Drone UI.

Our CI tests require secrets to run. These are run from the config in the `.drone.yml` file.

For more information on Drone secrets management, refer to <https://docs.drone.io/secret/>.

### Staging

Our staging environment is deployed to a Kubernetes deployment via the Kanopy developer platform.
We use [Kubernetes secrets](https://kubernetes.io/docs/concepts/configuration/secret/).
You can update secrets with either the [helm ksec extension](https://github.com/kanopy-platform/ksec)
or the Kanopy UI.

### Production

Same as staging, but in the production environment.