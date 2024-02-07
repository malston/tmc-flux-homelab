# GitOps with FluxCD with Tanzu Packages or Helm Charts

This repo can be used to deploy Tanzu Packages and Helm Charts in a GitOps fashion.

![alt text](https://github.com/coreydinkens/tmc-flux-poc/blob/master/images/overview.png)

Once Flux has been successfully enabled, add kustomizations in the following order, then remove in reverse to avoid package install/removal issues with prune enabled:

1. Add Flux Kustomization(s) to the bootstrap folder that reference any ServiceAccount as a prerequisite to installing packages or charts
2. Add Flux Kustomization(s) to the flux-config folder that references any PackageInstall or HelmRelease.

The current process to update the values is:

1. Modify package values as desired under `data-values` folder
2. Encode the entire values file in base64, e.g.

    ```sh
    cat  > <packagename>/<packagename>-data-values-secret.yaml <<EOF
    apiVersion: v1
    data:
      <packagename>-data-values.yaml:
    kind: Secret
    metadata:
      annotations:
        tkg.tanzu.vmware.com/tanzu-package: <packagename>-packages
      name: <packagename>-packages-values
      namespace: packages
    type: Opaque
    EOF
    ```

    For example:

    ```sh
    cat  > contour/contour-data-values-secret.yaml <<EOF
    apiVersion: v1
    data:
      contour-data-values.yaml: $(base64 < ./data-values/contour-data-values.yaml)
    kind: Secret
    metadata:
      annotations:
        tkg.tanzu.vmware.com/tanzu-package: contour-packages
      name: contour-packages-values
      namespace: packages
    type: Opaque
    EOF
    ```

The [FluxCD Kustomization manifests](https://github.com/malston/tmc-flux-homelab/tree/main/bootstrap) have the following dependencies configured:

1. **bootstrap folder** - Installs service accounts needed to manage Tanzu Carvel packages
2. **flux-config folder** - Installs all Tanzu packages
  a. bootstrap
    1. cert-manager
    2. fluent-bit
    3. contour
      a. Harbor
      b. Prometheus
        1. Grafana
