## Export environment variables

### Description
Upload artifact with environment variables

### Usage
```yaml
- name: Export environment variables
  uses: ./.github/actions/variables/export
  with:
    variables: |
      ENVIRONMENT=dev
      VERSION=1.0.0
```


