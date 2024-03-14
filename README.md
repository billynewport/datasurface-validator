This is an action which is used to validate that the Ecosystem model defined in the calling repository eco.py file is validate, 
authorized to make the changes, passes all policies from GovernanceZones and is backwards compatible. Any issues will be visible in the
pull request logs.

This should be called from a workflow in the repository similar to this:

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

  The 4 inputs must be passed so that the datasurface-validator action can perform the task.
