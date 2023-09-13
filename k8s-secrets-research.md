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
  