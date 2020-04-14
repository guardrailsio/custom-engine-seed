# Custom GuardRails Engine Seed

This is a skeleton repository to speed up creation of custom engines.

## Directory Structure

The key directories and files are:

```bash
├── test-src/           # place vulnerable code here
├── tools/
|   ├── build.sh        # builds the latest docker image
|   └── run.sh          # runs the latest docker image
├── Dockerfile          # installs and executes the custom engine logic
├── guardrails.json     # GuardRails manifest file
├── mappings.json       # GuardRails mappings file
├── README.md           # basic documentation and output reference
```

## How to use it

- Use this repo as a template: https://github.com/guardrailsio/custom-engine-seed/generate

## Custom Engine Requirements

The custom engines have certain requirements to be able to run on our platform.

### The GuardRails Manifest File

The GuardRails manifest file has the following format:

```json
{
  "name": "custom-engine-name",
  "private": true,
  "allowFor": {
    "providers": {
      "github": ["orgname-A", "orgname-B"],
      "gitlab": ["orgname-A", "orgname-B"]
    }
  },
  "runForLanguages": ["general", "java"]
}
```

The `name` will be the name of the new engine.
The `private` flag determine whether the engine is accessible by other users or just by the creator.
Currently, only `"private": true` is valid.
The `allowFor` object defines which organizations/groups are allowed to run the engine.
The `runForLanguages` array defines which languages the engine can run against.
Note: `general` would run this engine for every scan against every repostiory.

### The Guardrails Mapping File

The GuardRails mapping file has the following format:

```json
{
  "customEngineRuleID1": {
    "grId": "GR000X",
    "enabled": true
  },
  "customEngineRuleID2": {
    "grId": "GR000Y",
    "enabled": false
  }
}
```

The `customEngineRuleID1` must uniquely identify a ruleID of the custom engine.
To give an example, let's say you have improved upon the `gosec` tool and have some new rules that haven't been open-sourced.
You would have to map all [existing rules of `gosec`](https://github.com/securego/gosec#available-rules) from `G101` until `G505`.
If you have created a custom rule `G606`, then the mapping file must contain that as well.
For each ruleID of the custom engine, it has to define whether it is `enabled` or not.
If it is not `enabled`, it is not going to be considered a `vulnerability` by GuardRails, rather a `finding` of a tool that is considered irrelevant. This makes it possible to customize the reporting of tools based on risk appetite, internal company requirements and ensure that the noise of tools is kept to a minimum.
Finally, every ruleId of the custom engine has to be mapped to our internal GuardRails vulnerability ID.
This is important to allow us to group different vulnerabilities across engines and run de-dupliciation of findings, amongst other useful things.

The latest mapping table can be found here: TODO: Link to docs.

### The Custom Engine Logic

Todo: This can be anything ranging from orchestrating tools, to your own internal scripts.

### The GuardRails Output Format

Todo: Explain the output format, and also add the `false positive flag`.

### The Test Source

The test source is required to allow the engines to be fully self contained and testable.

### Test Steps

- Ensure that the schema validator returns no errors
  - example command and output
- Build an Image with `tools/build`
- Run it with `tools/run` for the test command that parses the output
