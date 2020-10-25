# drone-lambda-demo

How to deploy lambda application using [GitHub Actions](https://github.com/features/actions), [Drone](https://cloud.drone.io) or [GitLab CI](https://about.gitlab.com/stages-devops-lifecycle/continuous-integration/).

## Deploy with GitHub Actions

```yaml
name: deploy to lambda
on: [push]
jobs:

  deploy_zip:
    name: deploy lambda function from zip
    runs-on: ubuntu-latest
    strategy:
      matrix:
        go-version: [1.15.x]
    steps:
      - name: checkout source code
        uses: actions/checkout@v1
      - name: Install Go
        uses: actions/setup-go@v1
        with:
          go-version: ${{ matrix.go-version }}
      - name: Build binary
        run: |
          cd example && GOOS=linux go build -v -a -o main main.go && zip deployment.zip main
      - name: deploy zip
        uses: appleboy/lambda-action@v0.0.8
        with:
          aws_access_key_id: ${{ secrets.AWS_ACCESS_KEY_ID }}
          aws_secret_access_key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          aws_region: ${{ secrets.AWS_REGION }}
          function_name: gorush
          zip_file: example/deployment.zip
          debug: true
```

## Deploy with Drone

```yaml
---
kind: pipeline
name: testing

platform:
  os: linux
  arch: amd64

steps:
- name: build
  image: golang:1.15
  commands:
  - apt-get update && apt-get -y install zip
  - cd example && GOOS=linux go build -v -a -o main main.go && zip deployment.zip main

- name: deploy-lambda
  image: appleboy/drone-lambda
  settings:
    pull: true
    access_key:
      from_secret: AWS_ACCESS_KEY_ID
    secret_key:
      from_secret: AWS_SECRET_ACCESS_KEY
    region:
      from_secret: AWS_REGION
    function_name: gorush
    zip_file: example/deployment.zip
    debug: true
```
