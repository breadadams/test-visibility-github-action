# <img height="25" src="logos/test_visibility_logo.png" />  Datadog Test Visibility GitHub Action

GitHub Action that installs and configures [Datadog Test Visibility](https://docs.datadoghq.com/tests/). 
Supported languages are .NET, Java, Javascript, and Python.

## About Datadog Test Visibility

[Test Visibility](https://docs.datadoghq.com/tests/) provides a test-first view into your CI health by displaying important metrics and results from your tests. 
It can help you investigate and mitigate performance problems and test failures that are most relevant to your work, focusing on the code you are responsible for, rather than the pipelines which run your tests.

## Usage

1. Set [Datadog API key](https://app.datadoghq.com/organization-settings/api-keys) inside Settings > Secrets as `DD_API_KEY`.
2. Add a step to your GitHub Actions workflow YAML that uses this action. Set the language, service name and [site](https://docs.datadoghq.com/getting_started/site/) parameters: 

   ```yaml
   steps:
     - name: Configure Datadog Test Visibility
       uses: datadog/test-visibility-github-action@v1.0.3
       with:
         languages: java
         service-name: my-service
         api-key: ${{ secrets.DD_API_KEY }}
         site: US1
     - name: Run unit tests
       run: |
         mvn clean test
   ```

> [!IMPORTANT]  
> It is best if the new step comes __right before__ the step that runs your tests.
> Otherwise, installed tracing libraries might be removed by the steps that precede tests execution 
> (for example, `actions/checkout` will wipe out whatever was installed in the action workspace).

## Configuration

The action has the following parameters:

```yaml
languages: List of languages to be instrumented. Can be either "all" or any of "java", "js", "python", "dotnet" (multiple languages can be specified as a space-separated list).
service-name: The name of the service or library being tested.
api-key: Datadog API key. Can be found at https://app.datadoghq.com/organization-settings/api-keys
site: Datadog site (optional), defaults to US1. See https://docs.datadoghq.com/getting_started/site for more information about sites.
```

### Additional configuration

Any [additional configuration values](https://docs.datadoghq.com/tracing/trace_collection/library_config/) can be added directly to the step that runs your tests:

```yaml
- name: Run unit tests
  run: |
    mvn clean test
  env:
    DD_ENV: staging-tests
    DD_TAGS: layer:api,team:intake,key:value
```

## Limitations

### Tracing JS tests

For security reasons Github [does not allow](https://github.blog/changelog/2023-10-05-github-actions-node_options-is-now-restricted-from-github_env/) actions to alter the `NODE_OPTIONS` environment variable.
To work around this, the action provides a separate `DD_TRACE_PACKAGE` variable that needs to be appended to node options manually:

```yaml
- name: Run tests
  shell: bash
  run: npm run test-ci
  env:
    NODE_OPTIONS: -r ${{ env.DD_TRACE_PACKAGE }}
```