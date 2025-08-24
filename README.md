Provision my home lab with K3s using the [k3s-ansible](https://github.com/k3s-io/k3s-ansible) collection

1. Install k3s-ansible collection `ansible-galaxy collection install git+https://github.com/k3s-io/k3s-ansible.git`
1. Create [inventory.yaml](https://github.com/k3s-io/k3s-ansible/blob/master/inventory-sample.yml) with [ansible_user](https://docs.ansible.com/ansible/latest/reference_appendices/special_variables.html#term-ansible_user) and [ansible_host](https://docs.ansible.com/ansible/latest/reference_appendices/special_variables.html#term-ansible_host)
1. Run --check `ansible-playbook -i inventory.yaml cluster.yaml --check --become`
1. Run `ansible-playbook -i inventory.yaml cluster.yaml --become`
1. Create `ClusterSecretStore`:
    ```
    apiVersion: external-secrets.io/v1
    kind: ClusterSecretStore
    metadata:
      name: bitwarden-secretsmanager
    spec:
      provider:
        bitwardensecretsmanager:
          apiURL: https://api.bitwarden.com
          identityURL: https://identity.bitwarden.com
          auth:
            secretRef:
              credentials:
                key: token
                name: bitwarden-access-token
                namespace: external-secrets
          bitwardenServerSDKURL: https://bitwarden-sdk-server.external-secrets.svc.cluster.local:9998
          caProvider:
            type: ConfigMap
            name: selfsigned-ca
            namespace: external-secrets
            key: ca.crt
          organizationID: <ORGANIZATION_ID>
          projectID: <PROJECT_ID>
    ```
1. Create `bitwarden-access-token` Secret:
    ```
      kubectl -n external-secrets create secret generic bitwarden-access-token --from-literal=token=<TOKEN>
    ```
1. Create flux GitRepository and Kustomization to deploy [manifests](https://github.com/axeal/manifests)
    ```
    apiVersion: source.toolkit.fluxcd.io/v1
    kind: GitRepository
    metadata:
      name: manifests
      namespace: flux-system
    spec:
      interval: 60m
      ref:
        branch: master
      timeout: 60s
      url: https://github.com/axeal/manifests.git
    ---
    apiVersion: kustomize.toolkit.fluxcd.io/v1
    kind: Kustomization
    metadata:
      name: manifests
      namespace: flux-system
    spec:
      force: false
      interval: 15m
      path: ./clusters/home.axe.al/kustomizations/production
      prune: true
      sourceRef:
        kind: GitRepository
        name: manifests
    ```
