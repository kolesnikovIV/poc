# https://trello.com/c/EpQ4mfMj

- if client ensure that need to store app config in git then easiest way to do this:
  1. replace config db password to `DB_PASSWORD` keyword
  2. write up script which will replace `DB_PASSWORD` by env var `DB_PASSWORD` with sed -i and put that script into entrypoint.
  3. that's it, build container. save metadata info
  4. create k8s secret and store it, like:
```
kind: Secret
metadata:
    name: production-secret
spec:
    stringData:
        DB_PASSWORD: "supersecret" 
```
  5. inject that secret into deployment as env variable (https://kubernetes.io/docs/concepts/configuration/secret/#using-secrets-as-environment-variables)
  6. convert deployment to helm chart and write a variable called `environment`, reuse {{ .Values.environment }} in deployment to setup env var `ENV` and secret name like `name: {{ .Values.environment }}-secret`

# Summary
 there is lot of different ways to manage k8s secrets, if client are using terraform to deploy RDS for an example so, need to think about secret storage, here i would recommend to use aws secrets and then mount aws secrets as kubernetes secrets via secret-store-csi(https://kubernetes-sigs.github.io/secrets-store-csi-driver/charts) or external-secrets (as ans example). But there some issues, like password might be updated for some reason and here is another issue. So, to get new secret needs to restart app, then need to setup annotation with secret checksum (if use helm to calc k8s secret checksum), but there another issue, need to reconcilie helm release, so need to use some tools, like argo/flux/etc. But another way to keep up to date is inject aws secret by code with aws sdk, and **reconcile** aws secret to make sure it's up to date. Another way to implement secret injector like that (https://github.com/aws-samples/aws-secret-sidecar-injector), if team has capacity limitation/etc :) 