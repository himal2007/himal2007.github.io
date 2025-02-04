---
layout: distill
title: Testing changes in your Python Packages with GitHub Actions Workflow
description: Learning how to design a GitHub Actions workflow to test for changes in a Python package.
tags: github-actions, python, testing, automation
categories: github github-actions python testing automation
giscus_comments: true
date: 2025-01-26
featured: true
mermaid:
  enabled: true
  zoomable: true
code_diff: true
map: true
chart:
  chartjs: true
  echarts: true
  vega_lite: true
tikzjax: true
typograms: true

authors:
  - name: Himal Shrestha
    url: "https://himal2007.github.io"
    affiliations:
      name: MDU-PHL, The University of Melbourne

# Optionally, you can add a table of contents to your post.
# NOTES:
#   - make sure that TOC names match the actual section names
#     for hyperlinks within the post to work correctly.
#   - we may want to automate TOC generation in the future using
#     jekyll-toc plugin (https://github.com/toshimaru/jekyll-toc).
toc:
  - name: Introduction
  - name: What are GitHub Actions?
  - name: Key Concepts
  - name: Creating Your First Workflow
    subsections:
      - name: Setting Up the Workflow File
      - name: Defining the Job
      - name: Setting Up the Environment
      - name: Installing Dependencies
  - name: Testing Strategy
  - name: Result Comparison and Reporting
  - name: Best Practices and Tips
  - name: Resources for Further Learning

# Below is an example of injecting additional post-specific styles.
# If you use this post as a template, delete this _styles block.
_styles: >
  .fake-img {
    background: #bbb;
    border: 1px solid rgba(0, 0, 0, 0.1);
    box-shadow: 0 0px 4px rgba(0, 0, 0, 0.1);
    margin-bottom: 12px;
  }
  .fake-img p {
    font-family: monospace;
    color: white;
    text-align: left;
    margin: 12px 0;
    text-align: center;
    font-size: 16px;
  }
---

# Testing changes in your Python Packages with GitHub Actions Workflow

Working in public health, I've learned that maintaining bioinformatics tools is a bit like being a gardener – you need to constantly tend to your tools while ensuring they continue to work reliably. As I make updates to improve these tools, one of the challenges has been making sure that newer versions produce consistent results. After all, when these tools are part of critical public health pipelines, unexpected changes in output could have significant downstream effects.

Like many developers, I used to spend countless hours manually testing for changes between versions – a process that was not only time-consuming but prone to human error. That was until I discovered GitHub Actions, a game-changing automation tool that transformed my development workflow. Think of it as having a dedicated assistant who tirelessly checks your work, making sure everything runs smoothly across different versions.

GitHub Actions has become my reliable partner in ensuring code quality and consistency. It's a powerful platform that lets you define, manage, and automatically execute tasks directly within your repository. The best part? It catches potential issues before they can impact production environments, saving valuable time and reducing headaches.

In this blog post, I'll walk you through how to harness the power of GitHub Actions to create custom workflows that automatically test for changes that you make on your projects. We'll use a real-world example of a bioinformatics tool called *hicap*, which we use for serotyping *Haemophilus influenzae*. Whether you're working in bioinformatics or another field, you'll learn how to implement these automated testing workflows to make your development process more efficient and reliable.

## What are GitHub Actions?

GitHub Actions is a continuous integration and continuous delivery (CI/CD) platform that automates your software development workflows right in your GitHub repository. Think of it as your personal robot assistant that can:
- Run your tests automatically
- Build your packages
- Deploy your applications
- And much more!

## Key Concepts

Before diving in, let's understand some basic terminology:

1. **Workflow**: A configurable automated process made up of one or more jobs. It's defined in a YAML file in your repository's `.github/workflows` directory.

2. **Event**: A specific activity that triggers a workflow, like:
   - Push to a repository
   - Pull request creation
   - Release publication
   - Manual trigger (workflow_dispatch)
You can read more about all the available events in the [GitHub Actions documentation](https://docs.github.com/en/actions/reference/events-that-trigger-workflows).

3. **Job**: A set of steps that execute on the same runner (virtual machine).

4. **Step**: An individual task that can run commands or actions.

5. **Action**: A reusable unit of code that can be used in your workflow.

## Creating Your First Workflow: A Step-by-Step Guide

Let's break down how to create a testing workflow using `hicap` as our example.

### Step 1: Setting Up the Workflow File

Create a new file in your repository at `.github/workflows/test.yml`. The basic structure looks like this:

```yaml
name: Hicap Test Workflow
on:
  workflow_dispatch:
  release:
    types: [created, published]
```

This header section defines:
- The workflow name
- When it should run. The above configuration triggers the workflow both manually (on manual trigger) and automatically (when a release is created or published).

### Step 2: Defining the Job 

```yaml
jobs:
  test-and-compare:
    runs-on: ubuntu-latest
```

Here we:
- Define a job named "test-and-compare"
- Specify it should run on the latest Ubuntu virtual machine. 

Github provides various virtual machines for running workflows, such as `windows-latest`, `macos-latest`, and `ubuntu-latest` or with specific versions `ubuntu-20.04`, `windows-2019`, etc.

### Step 3: Setting Up the Environment

Let's look at the key steps for environment setup:

```yaml
steps:
  - name: Checkout current code
    uses: actions/checkout@v4

  - name: Set up Conda with Mamba
    uses: conda-incubator/setup-miniconda@v3
    with:
      miniforge-variant: Miniforge3
      miniforge-version: latest
      use-mamba: true
      activate-environment: test
```

These steps:
1. Check out your repository code
2. Set up Conda with Mamba for package management

<details>
  <summary><strong>More about actions</strong></summary>

  GitHub Actions are custom applications that can be used to automate tasks in your software development lifecycle. You can create and publish your own actions, or use actions shared by the community from the [GitHub Marketplace](https://github.com/marketplace?type=actions).

  <details>
    <summary><strong>Creating and Publishing Actions</strong></summary>
    You can create your own actions by defining them in a repository. Actions can be written in JavaScript or as Docker containers. Once created, you can publish your actions to the GitHub Marketplace to share them with the community.
  </details>

  <details>
    <summary><strong>Using Actions from the Marketplace</strong></summary>
    To use an action from the GitHub Marketplace, you can reference it in your workflow file using the <code>uses</code> keyword. For example: 

    ```yaml
    steps:
      - name: Use a Marketplace action
        uses: actions/checkout@v4
    ```

  </details>

  <details>
    <summary><strong>Using Actions from Other Repositories</strong></summary>
    You can also use actions from other repositories by specifying the repository name and version. For example:

    ```yaml
    steps:
      - name: Use an action from another repository
        uses: owner/repo@v1
    ```

  </details>

</details>

### Step 4: Installing Dependencies

```yaml
  - name: Create Conda environment from YAML
    shell: bash -el {0}
    run: |
      mamba env create -f environment.yml
```

This step creates a Conda environment using your `environment.yml` file, which should list all your package dependencies.

💡**Pro Tip**: I export the environment file from my development environment using `conda env export --no-builds > environment.yml`. The `--no-builds` flag ensures that the environment is platform-independent by excluding build-specific information.  

## Testing Strategy: The Two-Phase Approach

In our example, we use two-phase testing strategy, one with the current release and another with the new changes made in the repository.

1. **Phase 1: Test Current Release**
```yaml
  - name: Run hicap test (before update)
    shell: bash -el {0}
    run: |
      source $CONDA/bin/activate hicap-env
      mkdir -p test_old
      hicap -q tests/data/type_a.fasta -o test_old
```

Sometimes we have to run `source $CONDA/bin/activate hicap-env` to activate the environment as it bypasses the need for conda initialisation in the shell session. The `shell` attribute is used to specify the shell to run the commands in. The `-e` flag causes the shell to exit immediately if any command exits with a non-zero status and `-l` flag ensures that the shell is a login shell. `{0}` is a placeholder for the scripts and commands to be run.

2. **Phase 2: Test New Changes**
```yaml
  - name: Install editable hicap package
    shell: bash -el {0}
    run: |
      source $CONDA/bin/activate hicap-env
      pip install -e .

  - name: Run hicap test (after update)
    shell: bash -el {0}
    run: |
      conda activate hicap-env
      mkdir -p test_new
      python3 hicap-runner.py -q tests/data/type_a.fasta -o test_new
```

This approach allows us to:
- Run tests on both the current release and the new changes
- Catch unexpected changes
- Ensure backward compatibility

`pip install -e .` installs the package in editable mode, allowing changes to the source code to be reflected immediately without needing to reinstall the package.

## Result Comparison and Reporting

```yaml
  - name: Compare test results
    id: compare
    shell: bash -el {0}
    run: |
      if ! diff -r test_old test_new > diff_output.txt; then
        echo "DIFF_DETECTED=true" >> $GITHUB_ENV
      else
        echo "DIFF_DETECTED=false" >> $GITHUB_ENV
      fi

  - name: Print hicap test output
    if: env.DIFF_DETECTED == 'true'
    shell: bash -el {0}
    run: |
      echo "Differences found between test_old and test_new:"
      cat diff_output.txt
```

This section:
- Compares the output of both test runs
- Reports any differences found
- Provides detailed output for debugging

Here, `diff -r` recursively compares the contents of the two directories and writes the output to `diff_output.txt`. The `if` condition checks if differences were detected and if so, it sets an environment variable `DIFF_DETECTED` to `true` by appending to the `GITHUB_ENV` and vice versa. We can access custom environment variables in a workflow run using the `env` attribute as done in `env.DIFF_DETECTED`. The purpose of `id` field is to give a unique identifier to the step which can be used to refer to the step in the workflow. For example, in this workflow with `id: compare`, we can refer to this step as `steps.compare.outputs.<variable_name>`.

In this example, we've shown how to set up a testing workflow for a Python package using GitHub Actions using the `hicap` tool as an example. You can adapt this workflow to test your own Python packages by following the same principles. You can view the results of the workflow in the Actions tab of the repository. For an example of the workflow run where I deliberately introduced a change with the output that would print `hicap_version` as one of the columns, you can view the [workflow run](https://github.com/MDU-PHL/hicap/actions/runs/13092188787/job/36530138988) where you can open the `Print hicap test output` step to see the differences detected. The differences are in both lines of the output where first line has extra column `hicap_version` and the second line has the version number of the tool. Also the order of `IS1016_hits` i.e. `bexA, bexB, bexD, bexC` is different in the two outputs.

This is just a tip of the iceberg of what you can do with GitHub Actions. I am sure there are better ways to design the workflow and the action to make it more efficient and effective, even just for testing changes in the tool. This blog is written to make ourselves familiar with the concepts of GitHub Actions and how to design a workflow around it.

In summary, GitHub Actions provides a powerful way to automate your Python package testing. By following this guide and examining the `hicap` example, you now have the foundation to create your own testing workflows. Remember to start simple and gradually add complexity as needed.

## Best Practices and Tips

1. **Use Specific Versions**
   - Always specify versions for actions (e.g., `actions/checkout@v4`)
   - This prevents breaking changes from affecting your workflow

2. **Error Handling**
   - Use `set -euo pipefail` in bash scripts
   - This ensures your workflow fails fast on errors

3. **Environment Activation**
   - Always use `shell: bash -el {0}` when working with Conda environments
   - This ensures proper environment activation

4. **Clear Step Names**
   - Use descriptive names for steps
   - Makes it easier to debug when things go wrong


## Resources for Further Learning

1. [GitHub Actions Documentation](https://docs.github.com/en/actions)
2. [GitHub Actions for Python](https://docs.github.com/en/actions/automating-builds-and-tests/building-and-testing-python)
3. [Conda with GitHub Actions](https://github.com/conda-incubator/setup-miniconda)
4. [GitHub Actions Marketplace](https://github.com/marketplace?type=actions)
