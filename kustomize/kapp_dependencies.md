## Resource ordering
### kapp.k14s.io/change-group and kapp.k14s.io/change-rule annotation
`kubectl apply` on a cf4k8s will break because it does not have the required ordering rules. However there are other k8s ecosystem tools which also solve the ordering problem.
- kustomize (kubectl apply -k)
- Helm....see ordering list [here](https://github.com/helm/helm/blob/0498f0e5a2f1bae656e4ce876ab5d993bd68133f/pkg/releaseutil/kind_sorter.go#L31)

## Blocking/waiting on a deploy (like bosh)
I do not know if there is an alternative for this tooling. Operator could also `watch kubectl get pods -n cf-system`

## Rebase rules
Allows deploy to default to cluster chosen values and not show a diff when deploying

See rule [here](https://github.com/cloudfoundry/cf-for-k8s/blob/5016da8f1ab0b90d1e626109897801301f01b4d1/config/kapp-rebase-rules.yml)

Tracker story: https://www.pivotaltracker.com/n/projects/1382120/stories/171372962

## Update strategy
### kapp.k14s.io/update-strategy: fallback-on-replace
Docs about this behavior [here](https://github.com/k14s/kapp/blob/develop/docs/apply.md#kappk14sioupdate-strategy)
This is used for upgrading cf4k8s when there is a change to an immutable resource. If a resource can't be updated because it is immutable, then kapp will delete and recreate it
Usages:
1. ccdb-migrate job. 
alternative could be to version the name of the job
1. Istio [pod disruption budget](https://github.com/cloudfoundry/cf-k8s-networking/blob/1876e5bd991c2abd2b7515a7fbd140d491c0d0d6/config/istio/overlays/kapp-compatible.yaml#L16-L23)

## Versioning secrets
### kapp.k14s.io/versioned annotation
Rotates the secret name and corresponding secret name ref when secret changes

Note: We also tell kapp about one non-standard reference to a secret so that they are also updated with versions. Here the Istio Gateway references the cf-system-cert secret in a non-standard way (spec.servers is an array, within the array the tls item has the field `credentialName: cf-system-cert`.

```
----
-apiVersion: kapp.k14s.io/v1alpha1
-kind: Config
-metadata:
-  name: kapp-istio-gateway-rules

-templateRules:
-- resourceMatchers:
-  - apiVersionKindMatcher: {apiVersion: v1, kind: Secret}
-  affectedResources:
-    objectReferences:
-    - path: [spec, servers, {allIndexes: true}, tls]
-      resourceMatchers:
-      - apiVersionKindMatcher: {apiVersion: networking.istio.io/v1alpha3, kind: Gateway}
-      nameKey: credentialName
```