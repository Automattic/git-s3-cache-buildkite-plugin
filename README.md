# Git S3 Cache Buildkite Plugin

Adds a `pre-checkout` hook that downloads a git mirror from S3 and uses it as part of the `checkout` process.

## Example

Add the following to your `pipeline.yml`:

```yml
steps:
  - command:
      - "yarn install"
      - "yarn run test"
    plugins:
      - automattic/git-s3-cache#v1.0.0:
          bucket: "my-s3-bucket"
          repo: "path-to-repo-in-s3/"
```

## Configuration

Before using this plugin, you should have an S3 bucket set up in a format similar to:

```
my-bucket/
├── my-project/
│   ├── 2020-01-01-my-project.git
│   ├── 2020-01-03-my-project.git
│   ├── 2020-01-05-my-project.git
```

Or:

```
my-bucket/
├── my-project/
│   ├── 2000-01-01
│       ├── my-project.git
│   └── 2001-01-01
│       ├── my-project.git

```

Git mirrors must end in `.git`, and should be sorted alphabetically with the most recent mirror appearing last.

### `bucket` (Required, string)

The S3 bucket containing your mirrors.

### `repo` (Required, string)

The S3 path to the repo. For example `my-org/my-project`.

## Developing

To run the tests:

```shell
docker-compose run --rm lint
```

## Contributing

1. Fork the repo
2. Make the changes
3. Run the tests
4. Commit and push your changes
5. Send a pull request
