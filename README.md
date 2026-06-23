# supported-pythons

[![test](https://github.com/zacharyburnett/supported-pythons/actions/workflows/test.yml/badge.svg?branch=main)](https://github.com/zacharyburnett/supported-pythons/actions/workflows/test.yml)

retrieve active (not end-of-life) minor versions of Python supported by a Python package

| input         | description                                                                   |
| ------------- | ----------------------------------------------------------------------------- |
| `package`     | package name on PyPI, or URL to source repository containing `pyproject.toml` |
| `package-ref` | release version on PyPI, or tag / branch / commit of package source           |
| `no-eoas`     | also omit end-of-active-support versions of Python                            |

| output     | description                           |
| ---------- | ------------------------------------- |
| `versions` | Python minor versions as a JSON list  |
| `oldest`   | oldest supported Python minor version |
| `latest`   | latest supported Python minor version |

```yaml
jobs:
  supported-pythons:
    runs-on: ubuntu-latest
    steps:
      - id: supported-pythons
        uses: OpenAstronomy/supported-pythons@2.1.0
        with:
          package: romancal
          package-ref: 1.0.1
    outputs:
      versions: ${{ steps.supported-pythons.outputs.versions }}
      oldest: ${{ steps.supported-pythons.outputs.oldest }}
      latest: ${{ steps.supported-pythons.outputs.latest }}
  test:
    needs: supported-pythons
    runs-on: ubuntu-latest
    strategy:
      matrix:
        python-version: ${{ fromJSON(needs.supported-pythons.outputs.versions) }}
        include:
          - name: run tests (oldest Python)
            python-version: ${{ steps.supported-pythons.outputs.oldest }}
          - name: run tests (latest Python)
            python-version: ${{ steps.supported-pythons.outputs.latest }}
      fail-fast: false
    name: ${{ matrix.name || format('run tests (Python {0})', matrix.python-version) }}
    steps:
      - uses: actions/setup-python@v6
        with:
          python-version: ${{ matrix.python-version }}
      - uses: actions/checkout@v6
      - run: pip install .[test]
      - run: pytest
```
