This is an action which is used to validate that the Ecosystem model defined in the calling repository eco.py file is validate, 
authorized to make the changes, passes all policies from GovernanceZones and is backwards compatible. Any issues will be visible in the
pull request logs.

This should be called from a workflow in the repository similar to this:

```yaml
name: DataSurface Model Validator

on:
  pull_request:
    types: [opened, synchronize, reopened]

jobs:
  check:
    runs-on: ubuntu-latest

    steps:
    - name: DataSurface validator
      uses: billynewport/datasurface-validator@main
      with:
          github_token: ${{ secrets.GITHUB_TOKEN }}      
          base_repository: ${{ github.repository }}
          head_repository: ${{ github.event.pull_request.head.repo.full_name }}
          head_branch: ${{ github.event.pull_request.head.ref }}          

```

The 4 inputs must be passed so that the datasurface-validator action can perform the task. The github_token allows
the validator to access the repositories. The base_repository is the current live model taken from that repository.
The head_repository is where the proposed new ecosystem model is coming from and the branch is head_branch. 

Together, the head_repository and head_branch must be authorized in the model for any changes found when comparing the model in the 
head against the base model.

The validator uses the datasurface module defined in the main repository requirements.txt.

The following check-files-changes.yml should also be added to a repositories .github/workflow folder:

```yml
name: Check Changed Files are all python files
on: [push, pull_request]

# Idea here is to prevent any changes which can prevent repository workflows from running

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
    - name: Checkout repository
      uses: actions/checkout@v2
      with:
        fetch-depth: 0  # Fetch all history so we can compare with the previous commit

    - name: Check changed files
      run: |
        changed_files=$(git diff --name-only HEAD^ HEAD)
        for file in $changed_files; do
          if [[ ! $file =~ .*\.py$ ]] && [[ ! $file =~ .*\.md$ ]] && [[ ! $file == .gitignore ]]; then
            echo "Error: $file cannot be changed. Only python files, markdown files, and .gitignore are allowed."
            exit 1
          fi
        done
```

So, a DataSurface model repository would have 2 yml files. One like the validator one above and this file checker as the second one. It doesn't
seem possible to handle this in a single action.
