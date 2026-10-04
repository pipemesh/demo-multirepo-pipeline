# demo-multirepo-pipeline

A Pipemesh pipeline that spans three repositories. This one holds only
`pipemesh.yaml`; the code lives in the repositories it declares:

| Repository | What it holds |
| --- | --- |
| [demo-multirepo-orders](https://github.com/pipemesh/demo-multirepo-orders) | the orders service and its client library (`client/`) |
| [demo-multirepo-billing](https://github.com/pipemesh/demo-multirepo-billing) | a service that calls orders through the client library |

The client library crosses repositories as an artifact. `orders_client`
builds `orders-client.jar` from orders' `client/` directory and produces
it; `billing` consumes that jar and builds against it, without reading
orders' sources.

Each build says what it checks out in the repository it works in
(`repo: orders` with `checkout: [client]`), and gets exactly that; the
deploys are `job_type: deploy` and check out nothing, so the jar they consume
is their only input.

## What runs

| The push | orders_client | orders | billing | deploys |
| --- | --- | --- | --- | --- |
| orders `service/` | reused | runs | reused | orders only |
| orders `client/` | runs | runs | runs (new client jar) | both |
| a comment in orders `client/` | runs | runs | reused (same jar) | skip (same jars) |
| billing `src/` | reused | reused | runs | billing only |
| this README | reused | reused | reused | skip |

Every revision pins one commit of each repository, and a push to any of
the three starts a revision.

The [Pipemesh docs](https://pipemesh.io/docs/pipeline-yaml) describe `repos:`, `repo:`, `checkout:` and `consumes:`.
