# Passing data into workflow steps

Use parameters and outputs to pass data into your workflow steps.

Use a parameter to pass external data into a workflow step, and an output to pass data from one step into another step in a workflow.

## Passing external data into a step

To pass external data into a workflow step, first declare it in the top-level `parameters:` map, then use the `parameters` key in a template expression in your step specification.

This Slack notification example uses parameters to pass a Slack channel name and a custom message into the workflow step, which uses [a curated container image](https://hub.docker.com/r/relaysh/slack-step-message-send). When you run the workflow, Relay asks you to enter values for `channel` and `message`. This step also uses a [connection](managing-connections.md) to store and access the Slack bot user token.

```yaml
parameters:
  channel:
    description: Slack channel (include preceding hashtag)
    default: "#relay-workflows"
  message:
    description: Message to send to the channel
steps:
  - name: notify-channel
    image: relaysh/slack-step-message-send
    spec:
      channel: ${parameters.channel}
      message: ${parameters.message}
      connection: ${connections.slack.'relay-slack'}
```

For details about the parameters map, see the [workflow reference](../reference/relay-workflows.md#parameters)

## Passing internal workflow data into a step

If you need to pass data from one step in your workflow to another step, use the `outputs` key in the `spec` section of the step which consumes the data. The step which produces the data should use the `ni output set` command to do so, as in this example:

```yaml
parameters:
  message:
    description: The message to output from the final step

steps:
- name: generated-output
  image: relaysh/core
  input:
  - ni output set --key dynamic --value "$(date)"

- name: hello-world
  image: relaysh/core
  spec:
    message: ${parameters.message}
    dynamic: ${outputs.'generated-output'.dynamic}
  input:
  - echo "Hello world. Your message was $(ni get -p {.message}), and the generated output was $(ni get -p {.dynamic})."
```

Advanced users can use the [Python SDK](https://relay-sdk-python.readthedocs.io/en/latest/) in scripts to create and modify keys instead of running `ni`.
