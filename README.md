# Explanation Generator using ChatGPT

This repository contains a Github action that generates explanations of changes using the OpenAI API as a comment in pull request. It analyzes the differences between two commits in a pull request and provides a summary of the changes.

# Notable Features
- A `token limit` for both `prompt` and `response` can be set to control costs. In the event that the token limit is reached a comment will not be created. The default limit is `10000` prompt tokens and `256` response tokens.
- Prompts containing more then 3000 tokens are segmented into multipe requests to OpenAIApi and collated into a single response.
- The `initial comment` will be generated by comparing the `Head SHA` of the pull request branch against the `Base SHA` (I.E Main).
- `Subsequent comments` will be generated by comparing the `Head SHA` of the pull request branch against the `Parent SHA` of the pull request branch.
- A `custom prompt` can be configured allowing for flexibiliy in the response from ChatGPT. This can be configured via the input `custom-prompt`. The default value is `Given the diff of all parts. Summarize the changes in 300 words or less`.
- Specific files and paths can be ignored when generating the changes by configuring the input `ignore-paths`.
- The model used can be configured by setting the input `model`. The default model is `text-davinci-003`

# API UML diagram

![image](https://github.com/ps-aartread-org/chatgpt-comment-pull-request/assets/56265677/4aeef9d6-7284-409d-9ef9-cbc142d1c3de)


## Prerequisites

Before running this, ensure you have the following:

- An OpenAI API key.
- A GitHub personal access token.


## Usage

```yml
      - name: ChatGpt Comment
        uses: ps-aartread-org/chatgpt-comment-pull-request@main
```

This action will retrieve the pull request information and generate an explanation of the changes using the OpenAI API. If the explanation exceeds the maximum token limit or there are no changes after filtering, a comment will not be created.

## Action inputs

| Name | Description | Default | Required |
| --- | --- | --- | --- |
| `github-token` | `GITHUB_TOKEN` (permissions `contents: write` and `pull-requests: write`) or a `repo` scoped [Personal Access Token (PAT)](https://docs.github.com/en/github/authenticating-to-github/creating-a-personal-access-token). | `GITHUB_TOKEN` | true |
| `open-api-key` | OPENAI API Token created from https://platform.openai.com/account/api-keys. | `CHATGPT_API_KEY` | true |
| `custom-prompt` | The prompt to feed to ChatGPT | Given the diff of all parts. Summarize the changes in 300 words or less | false
| `max-prompt-tokens` | The max-prompt-tokens variable is used to limit the number of tokens that are sent to OpenAI when generating an explanation of the changes in a pull request. The default value of 10000 is used. | `10000` | false |
| `ignore-paths` | Comma separated list of paths and files those needs to be ignored from explanation | `All files are scanned if nothing is provided` | false |
| `model` | The model to use for the AI | `text-davinci-003` | false
| `temperature` | Parameter that controls how much randomness is in the input | `1` | false
| `max-response-tokens` | The number of tokens allowed in the response | `256` | false
| `top_p` | Controls how many of the highest-probability words are selected to be included in the generated text | `1` | false
| `frequency-penalty` | Reduces the probability of words that have already been generated | `0` | false
| `response-penalty` | Reduces the probability of a word if it already appeared in the predicted text |     `0` | false

## Configuration

Action provides several configuration options that you can modify based on your requirements:

- `apiKey`: Your OpenAI API key.
- `github-token`: Your GitHub personal access token.
- `max-prompt-tokens`: The maximum number of tokens allowed in the prompt.
- `ignore-paths`: Comma-separated list of file paths or patterns to ignore.
- `model`: The OpenAI language model to use for generating explanations.
- `temerature`: Parameter that controls how much randomness is in the output.
- `max-response-tokens`: The maximum number of tokens allowed in the response.
- `top_p`: Controls how many of the highest-probability words are selected to be included in the generated text.
- `frequency-penalty`: Reduces the probability of words that have already been generated.
- `presence-penalty`: Reduces the probability of a word if it already appeared in the predicted text.

## Action.yml

The `action.yml` file contains the metadata for the action.

```yaml
name: 'ChatGPT Comment'
description: 'Autogenerated a PR comment based on the code changes'
inputs:
  github-token:
    description: 'GitHub Token'
    required: true
  open-api-key:
    description: 'OpenAPI Token'
    required: true
  custom-prompt:
    description: 'Prompt to feed to ChatGPT'
    required: false
    default: 'Given the diff of all parts. Summarize the changes in 300 words or less'
  max-prompt-tokens:
    description: 'The maximum number of tokens to use for the prompt'
    required: false
    default: '10000'
  ignore-paths:
    description: 'comma separated list of paths and files'
    required: false
  model: 
    description: 'The model to use for the AI'
    required: false
    default: 'text-davinci-003'
  temperature:
    description: 'Parameter that controls how much randomness is in the output'
    required: false
    default: 1
  max-response-tokens:
    description: 'The maximum number of tokens allowed in the response'
    required: false
    default: 256
  top_p:
    description: 'Controls how many of the highest-probability words are selected to be included in the generated text'
    required: false
    default: 1
  frequency-penalty:
    description: 'Reduces the probability of words that have already been generated'
    required: false
    default: 0
  presence-penalty:
    description: 'Reduces the probability of a word if it already appeared in the predicted text'
    required: false
    default: 0
runs:
  using: 'node16'
  main: 'dist/index.js'
```

## Index.js

The `index.js` file contains the main logic for the action.

```javascript
// Import required packages and libraries
const axios = require('axios');
const core = require('@actions/core');
const github = require('@actions/github');
const { encode, decode } = require('gpt-3-encoder')

const { Configuration, OpenAIApi } = require("openai");
const { Octokit } = require('@octokit/rest');
const { context: githubContext } = require('@actions/github');

// ... (rest of the code)
```

## Reference example

The following workflow sets many of the action's inputs for reference purposes.
Check the [defaults](#action-inputs) to avoid setting inputs unnecessarily.

See below for the use cases.

```yml
on:
  pull_request:
    branches:
      - main
  workflow_dispatch:

jobs:
  chatgptComment:
    runs-on: ubuntu-latest
    name: Add Comment
    steps:
      - name: Add Comment
        uses: ps-aartread-org/chatgpt-comment-pull-request@main
        with:
          github-token: ${{ secrets.GITHUB_TOKEN }}
          open-api-key: ${{ secrets.CHATGPT_API_KEY }}
          max-prompt-tokens: '10000'
          ignore-paths: '.github/*, src/, package*.json, .env*'
          model: 'text-davinci-003'
          temperature: 1
          max-response-tokens: 256
          top_p: 1
          frequency-penalty: 0
          presence-penalty: 1

```

An example based on the above reference configuration adds comment that look like this:

<img width="961" alt="image" src="https://github.com/ps-aartread-org/chatgpt-comment-pull-request/assets/56265677/729f6d2b-3d2c-4549-b37b-fbb8c411aec4">


## License

This project is licensed under the [MIT License](LICENSE).

## Contributions

Contributions to this project are welcome. Feel free to open issues or submit pull requests to improve the script.

## Credits

This script utilizes the following packages and libraries:

- axios
- @actions/core
- @actions/github
- gpt-3-encoder
- openai
- @octokit/rest
