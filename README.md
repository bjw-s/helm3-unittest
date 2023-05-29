# Unit Test plugin for Helm 3

[![Release Status](https://github.com/bjw-s/helm3-unittest/workflows/release/badge.svg)](https://github.com/bjw-s/helm3-unittest/actions?query=workflow%3Arelease)
[![Latest Release](https://img.shields.io/github/v/release/bjw-s/helm3-unittest)](https://github.com/bjw-s/helm3-unittest/releases)

This is a fork of <https://github.com/vbehar/helm3-unittest>.

## Documentation

If you are ready for writing tests, check the [DOCUMENT](./DOCUMENT.md) for the test API in YAML.

- [Install](#install)
- [Get Started](#get-started)
- [Test Suite File](#test-suite-file)
- [Usage](#usage)
  - [Flags](#flags)
- [Example](#example)
- [Snapshot Testing](#snapshot-testing)
- [Related Projects / Commands](#related-projects--commands)
- [Contributing](#contributing)

## Install

```
$ helm plugin install https://github.com/bjw-s/helm3-unittest
```

It will install the latest version of binary into helm plugin directory.

## Get Started

Add `tests` in `.helmignore` of your chart, and create the following test file at `$YOUR_CHART/tests/deployment_test.yaml`:

```yaml
suite: test deployment
templates:
  - deployment.yaml
tests:
  - it: should work
    set:
      image.tag: latest
    asserts:
      - isKind:
          of: Deployment
      - matchRegex:
          path: metadata.name
          pattern: -my-chart$
      - equal:
          path: spec.template.spec.containers[0].image
          value: nginx:latest
```

and run:

```
$ helm unittest $YOUR_CHART
```

Now there is your first test! ;)

## Test Suite File

The test suite file is written in pure YAML, and default placed under the `tests/` directory of the chart with suffix `_test.yaml`. You can also have your own suite files arrangement with `-f, --file` option of cli set as the glob patterns of test suite files related to chart directory, like:

```bash
$ helm unittest -f 'my-tests/*.yaml' -f 'more-tests/*.yaml' my-chart
```

Check [DOCUMENT](./DOCUMENT.md) for more details about writing tests.

## Usage

```
$ helm unittest [flags] CHART [...]
```

This renders your charts locally (without tiller) and runs tests
defined in test suite files.

### Flags

```
--color              enforce printing colored output even stdout is not a tty. Set to false to disable color
-f, --file stringArray   glob paths of test files location, default to tests/*_test.yaml (default [tests/*_test.yaml])
-h, --help               help for unittest
-u, --update-snapshot    update the snapshot cached if needed, make sure you review the change before update
```

## Example

Check [`__fixtures__/basic/`](./__fixtures__/basic) for some basic use cases of a simple chart.

## Snapshot Testing

Sometimes you may just want to keep the rendered manifest not changed between changes without every details asserted. That's the reason for snapshot testing! Check the tests below:

```yaml
templates:
  - deployment.yaml
tests:
  - it: pod spec should match snapshot
    asserts:
      - matchSnapshot:
          path: spec.template.spec
  # or you can snapshot the whole manifest
  - it: manifest should match snapshot
    asserts:
      - matchSnapshot: {}
```

The `matchSnapshot` assertion validate the content rendered the same as cached last time. It fails if the content changed, and you should check and update the cache with `-u, --update-snapshot` option of cli.

```
$ helm unittest -u my-chart
```

The cache files is stored as `__snapshot__/*_test.yaml.snap` at the directory your test file placed, you should add them in version control with your chart.

## Tests within subchart

If you have customized subchart (not installed via `helm dependency`) existed in `charts` directory, tests inside would also be executed by default. You can disable this behavior by setting `--with-subchart=false` flag in cli, thus only the tests in root chart will be executed. Notice that the values defined in subchart tests will be automatically scoped, you don't have to add dependency scope yourself:

```yaml
# parent-chart/charts/child-chart/tests/xxx_test.yaml
templates:
  - xxx.yaml
tests:
  - it:
    set:
      # no need to prefix with "child-chart."
      somevalue: should_be_scoped
    asserts:
      - ...
```

Check [`__fixtures__/with-subchart/`](./__fixtures__/with-subchart) as an example.
