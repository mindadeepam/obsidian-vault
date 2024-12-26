To trigger the serverless script in the ds-serverless repo from our project repositories, we can
send a workflow-dispatch from the producer(project-repo) and the consumer(ds-serverless) must have trigger on workflows enabled. 

**Sending workflow dispatch using HTTP request**
[Reference link](https://docs.github.com/en/rest/actions/workflows?apiVersion=2022-11-28) 
We use curl command to send a workflow dispatch to trigger the consumer workflow. 
This is an example of the producer.yml file ([link](https://github.com/FarMart-Tech/content_moderation/blob/b4e7c85f7ece778b67ff7f681bdbaea7c63c685c/.github/workflows/content_mod_repo.yml))-

```
name: trigger_ds_serverless_content_mod

on:
  push:
    branches: 
      - deepam_dev
      - dev
    paths:
      - '.github/workflows/content_mod_repo.yml'
      - 'content_moderation/**'
  workflow_dispatch:  # Add this event definition to enable manual triggering

jobs:
  trigger-ds-serverless:
    environment: ${{ github.ref_name }}
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - uses: dorny/paths-filter@v2
        id: filter
        with:
          base: ${{ github.ref_name }}
          filters: |
            src_files:
              - 'content_moderation/predict.py'
              - 'content_moderation/constants.py'


## GENERAL RUNS
      - name: Install Python
        uses: actions/setup-python@v2
        with:
          python-version: "3.9"

      - name: Print tokens
        run: |
          echo "PAT: ${{ secrets.PAT }}"
          echo "branch: ${{ github.ref }}"


## IF THE SRC DIRECTORY SUBDIRECTORY IS CHANGED, TRIGGER THE SERVERLESS REPO
      - name: Trigger Workflow in Serverless Repo
        if: steps.filter.outputs.src_files == 'true'
        env:
          REPO_OWNER: FarMart-Tech
          REPO_NAME: ds-serverless-functions
          WORKFLOW_NAME: content_moderation_serverless.yml
          TRIGGER_REF: ${{ github.ref }}
        run: |
          set -e 
          curl -X POST \
            -H "Authorization: Bearer ${{ secrets.PAT }}" \
            -H "Accept: application/vnd.github.v3+json" \
            -H "X-GitHub-Api-Version: 2022-11-28" \https://api.github.com/repos/$REPO_OWNER/$REPO_NAME/actions/workflows/$WORKFLOW_NAME/dispatches \
            -d '{
              "ref": "'"$TRIGGER_REF"'"
            }'
```

**In the consumer.yml file**, we just need to have the on workflow-dispatch flag: 
[Reference link](https://docs.github.com/en/actions/using-workflows/events-that-trigger-workflows#workflow_dispatch)

```
on:
	push:
		branches:
			- dev
			- deepam_dev
		paths:
			- content_moderation_serverless/**
	workflow_dispatch: # This flag will enable us to trigger this consumer file manually.
```

Checkout [this](https://github.com/FarMart-Tech/ds-serverless-functions/blob/deepam_dev/.github/workflows/content_moderation_serverless.yml) content moderation workflow file in the ds-serverless-functions repository for reference.

### Reference links:
1. https://docs.github.com/en/rest/actions/workflows?apiVersion=2022-11-28
2. https://docs.github.com/en/actions/using-workflows/events-that-trigger-workflows#workflow_dispatch