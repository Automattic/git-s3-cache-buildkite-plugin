# **Git S3 Cache Buildkite Plugin**

Adds a `pre-checkout` hook that downloads git mirrors or bare repos from S3 and uses it as part of the `checkout` process.
Can also clone the repos during `post-checkout` and clean them up in `pre-exit`.

- [**Git S3 Cache Buildkite Plugin**](#git-s3-cache-buildkite-plugin)
  - [**Pipeline Usage**](#pipeline-usage)
  - [**Configuration**](#configuration)
  - [**Examples**](#examples)
    - [Single Cached Repo](#single-cached-repo)
    - [Two Cached Repos](#two-cached-repos)
    - [Cache Primary Build Repo](#cache-primary-build-repo)
    - [Cache Single Repo for Multiple Steps on Multiple Agents](#cache-single-repo-for-multiple-steps-on-multiple-agents)
  - [**Developing**](#developing)
  - [**Contributing**](#contributing)
## **Pipeline Usage**

Add the following to your `pipeline.yml`:

```yml
steps:
  - command: <generic command>
    label: "Caching Reference Repos"
    plugin:
      - automattic/git-s3-cache#v1.1.6:
          bucket: "my-s3-bucket"
          repos: "repo-name or repo-names"
          path: "path/to/repo"
          git_clone_flags: false
          auto_clone: true
          github_url: "full://github/path/with_org"
          auto_clean: true

```
(*More examples below*)

## **Configuration**

Before using this plugin, you should have an S3 bucket set up in a format similar to:

```
my-bucket/
├── my-project/
│   ├── 2000-01-01
│       ├── my-project.git.tar
│   └── 2001-01-01
│       ├── my-project.git.tar

```
>**Repos must be in tar format (eg `tar -cf <filename>.tar <source>`).**

Git mirrors/bare repositories must end in `.git`, and should be sorted alphabetically with the most recent mirror appearing last. For faster builds, we recommend **NOT** compressing your repos.
<br><br>


Key | Required | Type | Description
:---------|:----------:|:---------:|:---------
`bucket` | **Required** | string | The S3 bucket containing your mirrors.
`repos` | **Required** | string | Names of the repo or repo's you want to cache, separated by a space. Repos can include path info eg. `android/android.git`<p><p>**NOTE**: If you include path information, the plugin will automatically add the snapshot date in bewteen. (eg. `android/2021-07-08/android.git`).<p><p>*(This currently only supports one additional path. This will need to updated to handle unknown numbers of paths.)* <p>
`path` | **Required** | string | The primary path in S3 to your repo(s).  For multi-repos, this should be common for them all
`git_clone_flags` | Optional | booleans | Changes the behavior of the Buildkite Agent when it clones the build repo.<p><p> If specified to `true` this will alter the git clone functionality for the entire build to `-v --reference-if-able <reference_repo_destination>`.<p>*Works only with one repo and, should be your repo that is running the pipeline.*
`auto-clone` | optional | boolean | If `true` will attempt to clone the repos you specified into the workspace. Set this to false if you want to control when and where your repos get cloned manually.<p><p>***Requires**:* complete `github_url` path to repo.
`github_url` | optional | string | Needed for `auto-clone` functionality. Should be full path up to repo, do not include trailing slash.<p><p> *Should work with ssh links as well*.
`auto-clean` | optional | boolean | If `true` it will delete the build directory in `/tmp/${BUILDKITE_PIPELINE_NAME}`. <p><p>Use for systems that are not ephemeral.<p><p>*This runs at the end of EVERY STEP in a pipeline. It is most helpful with parallel builds.*

## **Examples**
<p><br><p>

### Single Cached Repo

 ```yml
steps:
  - command: test_repo.sh
    label: "Caching Reference Repos"
    plugin:
      - automattic/git-s3-cache#v1.1.6:
          bucket: "my-s3-bucket"
          repos: "jetpack"
          path: "wordpress"
          git_clone_flags: false
          auto_clone: true
          github_url: "https://github.com/automattic"
          auto_clean: true
```

- Cache `jetpack.git` from `s3://my-s3-bucket/wordpress/<latest_date>/`
- Clone this repo into the build working directory
- Run `test_repo.sh`
- Delete the cached repo at the end of the step
<p><br><p>

### Two Cached Repos

 ```yml
steps:
  - command: test_repo.sh
    label: "Caching Reference Repos"
    plugin:
      - automattic/git-s3-cache#v1.1.6:
          bucket: "my-s3-bucket"
          repos: "jetpack/jetpack calypso/wp-calypso"
          path: "wordpress"
          git_clone_flags: false
          auto_clone: true
          github_url: "https://github.com/automattic"
          auto_clean: true
```

- Assumes pipeline is running in a separate repo
 (eg. github.com/wordpress/sensei)
- Cache `jetpack.git` from `s3://my-s3-bucket/wordpress/jetpack/<latest_date>/`
- Cache `wp-calypso.git` from `s3://my-s3-bucket/**wordpress**/calypso/<latest_date>/`
- Clone both repos into the build working directory
- Run `test_repo.sh`
- Delete the cached repo at the end of the step
<p><br><p>

### Cache Primary Build Repo

 ```yml
steps:
  - command: test_repo.sh
    label: "Caching Reference Repo"
    plugin:
      - automattic/git-s3-cache#v1.1.6:
          bucket: "my-s3-bucket"
          repos: "jetpack"
          path: "wordpress"
          git_clone_flags: true
          auto_clone: false
          auto_clean: false
```


- Assumes pipeline is running in Jetpack repo (eg. github.com/wordpress/jetpack)
- Cache `jetpack.git` from `s3://my-s3-bucket/wordpress/<latest_date>/`
- During the `Preparing working directory` step, it will clone using the reference repo.
- Run `test_repo.sh`
- Leave the cached repo on agent
<p><br><p>

### Cache Single Repo for Multiple Steps on Multiple Agents

```yml
git-s3-cache: &git-s3-cache
  plugin:
    - automattic/git-s3-cache#v1.1.6:
        bucket: "my-s3-bucket"
        repos: "wp/jetpack"
        path: "wordpress"
        github_url: "https://github.com/automattic"
        git_clone_flags: false
        auto_clone: true
        auto_clean: true

  - label: "step 1"
    <<: *git-s3-cache
    command: <your command>
  - label: "step 2"
    <<: *git-s3-cache
    command: <your command>
```

- Assumes pipeline is running in a separate repo
 (eg. github.com/wordpress/sensei)
- *Runs both steps concurrently on two available agents*
- Cache `jetpack.git` from `s3://my-s3-bucket/wordpress/wp/jetpack/<latest_date>/`
- Clone the repo into the build working directory
- Run `<your command>`
- Delete the cached repo from each agent at the end of the step
<p><br><p>

## **Developing**

To run the tests:

```shell
docker-compose run --rm lint
```

## **Contributing**

1. Fork the repo
2. Make the changes
3. Run the tests
4. Commit and push your changes
5. Send a pull request
