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
  "version": "1.0.0",
  "private": true,
  "allowFor": {
    "providers": {
      "github": ["orgname-A", "orgname-B"],
      "gitlab": ["orgname-A", "orgname-B"]
    }
  },
  "runForLanguage": "general",
  "timeoutInSeconds": 600,
  "rules": {
    "customEngineRuleID1": {
      "grId": "GR0001",
      "enable": true
    },
    "customEngineRuleID2": {
      "grId": "GR0004",
      "enable": false
    }
  }
}
```

The `name` will be the name of the new engine.
The `private` flag determine whether the engine is accessible by other users or just by the creator.
Currently, only `"private": true` is valid.
The `allowFor` object defines which organizations/groups are allowed to run the engine.
The `runForLanguages` array defines which languages the engine can run against.
Note: `general` would run this engine for every scan against every repository.
The `timeoutInSeconds` defines a specific timeout for a scan of the engine. By default this value is 600 (10 minutes), and can be increased to a maximum of 1800 (30 minutes).
The `customEngineRuleID1` must uniquely identify a ruleID of the custom engine.
To give an example, let's say you have improved upon the `gosec` tool and have some new rules that haven't been open-sourced.
You would have to map all [existing rules of `gosec`](https://github.com/securego/gosec#available-rules) from `G101` until `G505`.
If you have created a custom rule `G606`, then the mapping file must contain that as well.

Each ruleID of the custom engine has to define whether it is enabled or not.
If `enable` is not set to `true` then it is not going to be considered a `vulnerability` by GuardRails, rather a `finding` of a tool that is considered irrelevant. This makes it possible to customize the reporting of tools based on risk appetite, internal company requirements and ensure that the noise of tools is kept to a minimum.

Finally, every ruleId of the custom engine has to be mapped to our internal GuardRails vulnerability ID.
This is important to allow us to group different vulnerabilities across engines and run de-duplication of findings, amongst other useful things.

The latest mapping table can be found below:

| GR ID  | GuardRails Category Title              |
| ------ | -------------------------------------- |
| GR0001 | Insecure Use of SQL Queries            |
| GR0002 | Insecure Use of Dangerous Function     |
| GR0003 | Insecure Use of Regular Expressions    |
| GR0004 | Hard-Coded Secrets                     |
| GR0005 | Insecure Authentication                |
| GR0006 | Insecure Access Control                |
| GR0007 | Insecure Configuration                 |
| GR0008 | Insecure File Management               |
| GR0009 | Insecure Use of Crypto                 |
| GR0010 | Insecure Use of Language/Framework API |
| GR0011 | Insecure Processing of Data            |
| GR0012 | Insecure Network Communication         |
| GR0013 | Using Vulnerable Libraries             |
| GR0014 | Privacy Concerns                       |
| GR0015 | Information Disclosure                 |

### The Custom Engine Logic

The main purpose of the custom engine is to run against repositories with the supported language for the security check.
Regardless of what the custom engine logic does, it has to ensure that a [certain output](#the-guardrails-output-format) is generated.

An example engine is provided at this [link](https://github.com/guardrailsio/detect-sensitive-artifacts). Once on the GitHub page of this repository, the engine be downloaded directly as a ZIP file by clicking on "Clone or download" -> "Download ZIP".

### The GuardRails Output Format

The following code snipped contains an example of the GuardRails output format and has clarifying comments for each line.

```json
{
    "engine": {
        "name": "custom-engine-name",
        "version": "1.0.0"
    },
    "language": "general",
    "status": "success",
    "executionTime": 17413,
    "findings": 1,
    "process": {
        "name": "engine-general-ossindex",
        "version": "1.0.0"
    },
    "output": [
        {
            "type": "sast",
            "ruleId": "OSSINDEX001",
            "location": {
                "path": "benchmark/pom.xml",
                "positions": {
                    "begin": {
                        "line": 594
                    }
                }
            },
            "metadata": {
                "lineContent": "self.try(params[:graph])",
                "description": "User controlled method execution",
                "severity": "High",
                "language": "javascript",
            }
        }
    ]
};
```

The `engine` object has two fields. The `name` and the `version`.

- The `name` should be the same as the name in the GR manifest file.
- The `version` should be the same as the version in the GR manifest file.

The `language` defines the language that this engine runs against.

The `status` defines the status of the engine scan, which can be either `successful`, `unsuccessful`, or `error`.

The `executionTime` is the time that it took the engine to run against the repository in milliseconds.

The `findings` reflects the amount of issues that were identified in the scan.

The `process` object contains the name and version of another tool that is being run as part of the engine, or otherwise has the same values as the `engine` object.

The `output` contains the array of findings of the engine.

- The `type` of the finding with possible values of `sast`, `sca`,`secret`, or `cloud`. `SAST` should be used for vulnerabilities identified in code. `SCA` should be used for vulnerable dependencies. `Secret` should be used for any hard-coded secrets, or sensitive information. `Cloud` should be used for vulnerabilities in cloud configuration.
- The `ruleId` reflects the unique `ruleID` of the custom engine, which can occur multiple times.
- The `location` object has information about the `path` and `line` that triggered the finding. This object used to support `end`, but it has been discontinued.
- The `metadata` object contains some additional information relevant to the finding, which is mostly unstructured data that will be stored with the finding.
  - The `lineContent`, which is the only required field in the metadata object, contains the line of code that triggered the finding.
  - The `description` of the finding
  - The `severity` of the finding.
  - The programming `language` for which the finding applies, which, depending on the language can be used to override the value in the main scan result. This is useful to point to the right documentation link.

Note: We don't currently support custom links to documentation, but this will be added soon.

### The Test Source

The test source is required to allow the engines to be fully self contained and testable.
That means add a sample repository, or just relevant files that contain the code which would produce at least one output for your custom engine.

### Test Steps

- Ensure that the schema validator returns no errors
  - example command and output
- Build an Image with `tools/build`
- Run it with `tools/run` for the test command that parses the output
