# Vue on Wodby

What this service adds to the Nginx service it is based on. A deployed environment serves the application's static build with Nginx. A development workspace runs something else: the Vite development server, in a Node.js image.

## Deployed environments

- The service has no Dockerfile and runs no Node.js process. The image is the Nginx image with files copied into `/var/www/html`, its document root.
- The pipeline produces those files: it installs dependencies and runs the production build in a Node.js container, then builds the image from the output directory only (`dist` in the Vue boilerplate). Source files and `node_modules` are not in the image.
- A change to the name or location of the build output must be made in the pipeline's `wodby ci build --from` step as well, or the image is built from the wrong directory.
- Requests for paths that are not files are answered with `index.html`, so client-side routes work without extra Nginx configuration.
- The application is built before deployment and runs in the browser. Environment variables of the service, including `WODBY_*`, do not reach it at run time. Values it needs must be in the build, or fetched from an API.

## Variables the service sets

`PORT` and `NODE_PORT` (both 8080) and `WORKSPACE_NODE_COMMAND`. They are used by the development workspace only; Nginx ignores them.

## In a development workspace

- The container runs a Node.js development image instead of Nginx, with the checkout mounted at `/usr/src/app`. Nginx, its configuration and the pipeline's build step are not involved.
- Workspace setup installs dependencies with `workspace-node prepare`. The package manager is taken from `packageManager` in `package.json`, otherwise from the lockfile, and the install follows the lockfile exactly, with development dependencies. Run it again after changing dependencies.
- The application is started through Vite's JavaScript API with the project's own Vite configuration file, not through the `dev` script of `package.json`. Options in that script do not apply. Host, port, `strictPort` and watcher polling are set on top of the project's configuration, in memory; no file is written. `vite` must be a dependency of the project.
- The development server listens on `0.0.0.0:8080`, and the container's health checks probe that port. Do not change the port in the project configuration.
- File watching uses polling, so a saved file is rebuilt without a restart.
- The start command does not set `server.allowedHosts`. Vite rejects requests for host names it does not know: allow the hosts of the environment in `server.allowedHosts` of the project's Vite configuration.
- `NODE_ENV` is `development`.
- To start the server differently, set `WORKSPACE_NODE_COMMAND` on the service. The command must listen on `$HOST:$PORT` and configure polling itself.
