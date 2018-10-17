# EVM Solidity Benchmark

Benchmark for Solidity code analysis tools.

*Just wondering which EVM hacking tool is most 1337? **[See results here.](https://TODO)***

<!-- markdown-toc start - Don't edit this section. Run M-x markdown-toc-refresh-toc -->
**Table of contents**

- [EVM Solidity Benchmark](#evm-benchmark)
    - [Introduction](#introduction)
    - [Cloning](#cloning)
    - [Benchmarks have changed?](#benchmarks-have-changed)
    - [Project setup](#project-setup)
    - [Analysers setup](#analysers-setup)
    - [About Python Code to Run Benchmarks and Create Reports](#about-python-code-to-run-benchmarks-and-create-reports)
    - [Adding additional analyser to benchmark](#adding-additional-analyser-to-benchmark)
    - [See also](#see-also)

<!-- markdown-toc end -->

## Introduction

This repo is a collection of benchmarks for evaluating Solidity EVM code analysis tools' precision.

Particularly, it's a fork of Bernhard Mueller's fork of Suhabe Bugara's excellent benchmark suite.

Bernhard had added Trail of Bits' [(Not So) Smart Contracts](https://github.com/trailofbits/not-so-smart-contracts) as a submodule, and some runners for the whole thing.

Reports from running `runner/run.py` and `runner/report.py` are [**here**](https://TODO).

## Cloning

Be mindful to use the `--recurse-submodules` option.

```console
$ git clone --recurse-submodules https://github.com/gingeleski/evm-benchmark.git
```

If benchmarks change and you want to pull in the new benchmark code, use `git submodule update`.

## Vagrant setup

*(Not yet implemented 10/16/2018)*

The easiest way to set everything up - benchmarks *and* analyzers - is with the provided Vagrant box.

Follow the steps below and the heavy lifting is done.

```console
$ cd evm-benchmark/vagrant
$ vagrant up
$ vagrant ssh
```

## Manual setup

#### Runners

Reports programs are written in Python 3.6 or better.

To install the Python dependencies, do the following, preferably in a [**virtualenv**](https://docs.python.org/3/tutorial/venv.html).

```console
$ pip install -r requirements.txt
```

#### Analysers

Analysers are not part of project dependencies and they should be installed manually. The reason for this was to make setup not dependent on analysers failures (there might be some) and to make it possible for user to select specific analysers to benchmark, instead of installing all of them.  

Below, you will find a list of supported analysers with installation instructions and known bugs that prevents installation or makes analyser unworkable.

##### [Mythril](https://github.com/ConsenSys/mythril)

Available in PyPi

```console
$ pip install mythril
```

##### [Manticore](https://github.com/trailofbits/manticore)

Available in PyPi

```console
$ pip install manticore
```

Known bugs:

- `ValueError: not allowed to raise maximum limit`
  * Description: Latest version in PyPi - `0.2.0` fails during analyser execution
  * Workaround: Source code in master branch already contains fix. Thus, while the new version for PyPi is not released manticore must be installed manually:
  ```console
  $ git clone https://github.com/trailofbits/manticore.git
  $ cd manticore/
  $ pip install .
  ```

- Installation fails on MacOS
  * Description: https://github.com/trailofbits/manticore/issues/1075
  * Workaround: n/a. On some systems with MacOS it was possible to successfully install it, therefore try to install at firsts.

#### About Python runner and reporting code

We assume the benchmark repositories are set up with Git via the `--recurse-submodules` switch described earlier.

With this in place, the two Python programs are run in sequence to

* run an analyzer over a benchmark suite, and
* generate HTML reports for a benchmark suite that we have gathered data for in the previous step

##### [runner/run.py](https://github.com/gingeleski/evm-benchmark/blob/master/runner/run.py)

Executes specified benchmark suite.

Input arguments:

- `-s`, `--suite`       Benchmark suite name. Default `Suhabe`. Currently supported: `Suhabe`, `nssc`
- `-a`, `--analyser`    Analyser to benchmark. If not set all supported analysers will be benchmarked.
                        Currently supported: `Mythril`, `Manticore`
- `-v`, `--verbose`     More verbose output; use twice for the most verbose output
- `-t`, `--timeout`     Maximum time allowed on any single benchmark. Default 7 seconds
- `--files`             Print list of files in benchmark and exit

**Description:**

`runner/run.py` takes a number of command-line arguments; one of them is the name of a benchmark suite. From that it reads two YAML configuration files for the benchmark. The first YAML file has information about the benchmark suite: the names of the files in the benchmarks, whether the benchmark is supposed to succeed or fail with a vulnerability, and possibly other information. An example
of such a YAML file is [benchconf/Suhabe.yaml](https://github.com/gingeleski/evm-benchmark/blob/master/benchconf/Suhabe.yaml). The
other YAML input configuration file is specific to the analyzer. For Mythril on the Suhabe benchmark, it is called [benchconf/Suhabe-Mythril.yaml](https://github.com/gingeleski/evm-benchmark/blob/master/benchconf/Suhabe-Mythril.yaml)

For each new Benchmark suite, these two YAML files will need to exist. The second one you can start out with an empty file.

The output is a YAML file which is stored in the folder [`benchdata`](https://github.com/gingeleski/evm-benchmark/tree/master/benchdata) with a subfolder under that with the name of the benchmark. For example the output of `run.py` for the Suhabe benchmark suite will be a file called `benchdata/Suhabe/Mythril.yaml`.

##### [runner/report.py](https://github.com/gingeleski/evm-benchmark/blob/master/runner/report.py)

Takes the aforementioned data YAML files and creates a HTML report from that.

Input arguments:

- `-s`, `--suite`       Benchmark suite name. Default `Suhabe`,

Here is an example of complete report generation using Mythril on the Suhabe benchmark giving Mythril 5 minutes maximum to analyze a single benchmark:

```console
$ python runner/run.py --timeout 300 --suite Suhabe --analyser Mythril
$ python runner/report.py --suite Suhabe
```

## Adding an additional analyser to benchmark

Source code related to analysers is located in `runner/analysers/` module. In order to add support of a new analyser:

* Implement new class inherited from `BaseAnalyser`
* New class must be **imported** in `analyser/__init__.py`
* Create configuration files with expected output in `benchconf/`

Please check existing analysers as an example.

## See also

Suggestions and pull requests are welcome.

Please create a [new issue](https://github.com/gingeleski/evm-benchmark/issues/new) for idea discussion.
