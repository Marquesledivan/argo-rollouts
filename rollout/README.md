# Blue Green

The blue green strategy is not supported by built-in Kubernetes Deployment but available via third-party Kubernetes controller.
This example demonstrates how to implement rollout deployment via [Argo Rollouts](https://github.com/argoproj/argo-rollouts):

1. Install Argo Rollouts controller: https://github.com/argoproj/argo-rollouts#installation
2. Create a sample application and sync it.

```
argocd app create --name rollout --repo https://github.com/Marquesledivan/argo-rollouts.git --dest-server https://kubernetes.default.svc --dest-namespace default --path  rollout  && argocd app sync rollout
```

Once the application is synced you can access it using `rollout-helm-guestbook` service.

3. Change image version parameter to trigger rollout deployment process:

```
argocd app set rollout -p image.tag=0.2 && argocd app sync rollout
```

Now application runs `ks-guestbook-demo:0.1` and `ks-guestbook-demo:0.2` images simultaneously.
The `ks-guestbook-demo:0.2` is still considered `blue` available only via preview service `rollout-helm-guestbook-preview`.

4. Promote `ks-guestbook-demo:0.2` to `green` by patching `Rollout` resource:

```
argocd app patch-resource rollout --kind Rollout --resource-name rollout-helm-guestbook --patch '{ "status": { "verifyingPreview": false } }' --patch-type 'application/merge-patch+json'
```

This promotes `ks-guestbook-demo:0.2` to `green` status and `Rollout` deletes old replica which runs `ks-guestbook-demo:0.1`.
