# ci-first-pipeline

[![Python Calculator CI](https://github.com/mustafa-yusuf-d/ci-first-pipeline/actions/workflows/python-ci.yml/badge.svg)](https://github.com/mustafa-yusuf-d/ci-first-pipeline/actions/workflows/python-ci.yml)

A small Python calculator with a GitHub Actions pipeline that lints and tests every change. I built it to learn how continuous integration works: what 
triggers a run, how a failure shows up, and how to read it.

## What the app does

app/calc.py has three functions: add, subtract and divide. divide raises a ValueError when the divisor is zero. The tests in tests/test_calc.py use 
unittest and cover each function, plus the divide-by-zero case.

## What the pipeline does

The workflow is in .github/workflows/python-ci.yml.

Trigger: every push to main and every pull request into main.
Matrix: one test job definition runs twice, once on Python 3.12 and once on 3.13, so each run has two jobs.
Steps, in order:
Check out the code (actions/checkout@v4).
Set up Python (actions/setup-python@v5).
Install the dependencies from app/requirements.txt.
Lint with ruff check ..
Run the tests with python3 -m unittest discover -v.

A failing step stops the job, so a lint error means the tests never run. The pipeline runs the same commands I run locally, so a local pass should match a 
pass in CI.

## Running it locally

pip install -r app/requirements.txt
ruff check .
python3 -m unittest discover -v

The ruff version is pinned in my requirements.txt file.

## What broke and how I fixed it

To see a failing pipeline, I changed the operator in add from + to - on a branch and opened a pull request. The Test with unittest step went red, and the 
log showed:

AssertionError: 5 != 15

The traceback pointed at the test file, not at calc.py. The failing assertion shows where a check failed, not where the fault is, so I looked at which 
code add(10, 5) calls and found the wrong operator. I fixed it with a new commit, and the checks went green. The failed run stayed in the Actions history.

A second thing surprised me. Before I narrowed the trigger to main, a push to my feature branch started a push run as well as the pull request run. A 
run uses the workflow file that exists in the commit that triggered it, and my feature branch still had the old trigger.

## What I would improve next

Split lint and tests into separate jobs linked with needs.
Cache the pip dependencies to speed up runs.
Add branch protection so a pull request with failing checks cannot be merged.
Restrict the workflow token with a permissions block.
Build a Docker image in the pipeline and push it to a registry.
