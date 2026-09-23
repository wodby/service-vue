# Vue service for Wodby

Run statically built Vue applications on Nginx with [Wodby](https://wodby.com).

This service inherits the Nginx runtime and specializes its application build contract with the [Vue boilerplate](https://github.com/wodby/vue-boilerplate). It is intended for use through the [Vue stack](https://github.com/wodby/stack-vue) or from a custom Wodby stack.

## Manifest design

- Base service: `nginx` `2.0.0`, compatible with `^2.0.0`
- Runtime, endpoint, scaling, and Helm configuration: inherited from Nginx
- Build source connection: enabled
- Starter: Vue boilerplate

Validate the manifest with:

```sh
wodby service validate-manifest service.yml --org <org-id>
```

See the [service manifest reference](https://wodby.com/docs/2.0/services/template/) and [managed services index](https://github.com/wodby/services).

## Development workspaces

Standard environments keep Nginx and their existing build pipeline. Workspaces select a Node development image and the Vite polling helper on port 8080. Dependencies are installed during preparation; custom package lifecycle startup scripts require a `WORKSPACE_NODE_COMMAND` override. Configure the framework to allow the actual preview hostname.

Requires a runtime image declaring workspace contract version 1. Ordinary and development option tags must use matching revisions.
