To create a package and automate its release to PyPI using Poetry, GitHub Actions, and CI/CD on Ubuntu, follow these steps:

1. **Initialize Poetry**: Navigate to your project directory (`/home/sam/github/myrepoutils`) and initialize Poetry if you haven't already. This creates a `pyproject.toml` file, which is crucial for Poetry to manage your package and its dependencies.

   ```bash
   cd /home/sam/github/myrepoutils
   poetry init
   ```

2. **Configure `pyproject.toml`**: Ensure your `pyproject.toml` includes details about your package such as name, version, description, authors, and dependencies. Here's an example snippet:

   ```toml
   [tool.poetry]
   name = "myrepoutils"
   version = "0.1.0"
   description = ""
   authors = ["Your Name <you@example.com>"]

   [tool.poetry.dependencies]
   python = "^3.8"

   [tool.poetry.dev-dependencies]
   pytest = "^6.2.4"
   ```

3. **Set Up GitHub Actions for CI/CD**:

   - Inside the `.github/workflows` directory, create a YAML file, e.g., `ci_cd.yml`.
   - Define steps for installing Poetry, caching dependencies, running tests, and publishing to PyPI only when a new tag is pushed. Use the examples from Source 0 and adapt them to include installing dependencies and running tests. Here's a simplified version:

     ```yaml
     name: Python Package CI/CD

     on:
       push:
         branches:
           - main
         tags:
           - "v*"

     jobs:
       build:
         runs-on: ubuntu-latest

         steps:
           - uses: actions/checkout@v2

           - name: Set up Python
             uses: actions/setup-python@v2
             with:
               python-version: 3.8

           - name: Install Poetry
             run: |
               curl -sSL https://raw.githubusercontent.com/python-poetry/poetry/master/get-poetry.py | python -
               echo "$HOME/.poetry/bin" >> $GITHUB_PATH

           - name: Install dependencies
             run: poetry install

           - name: Run tests
             run: poetry run pytest

           - name: Publish to PyPI
             if: startsWith(github.ref, 'refs/tags/')
             run: poetry publish --build -u __token__ -p ${{ secrets.PYPI_TOKEN }}
             env:
               PYPI_TOKEN: ${{ secrets.PYPI_TOKEN }}
     ```

   - Ensure you replace `PYPI_TOKEN` with your actual PyPI token added to GitHub Secrets.

4. **Add PyPI Token to GitHub Secrets**:

   - Go to your repository on GitHub.
   - Navigate to Settings > Secrets.
   - Click on "New repository secret".
   - Name it `PYPI_TOKEN` and paste your PyPI API token.

5. **Push Changes and Tag**:
   According to SemVer, the version number format is `MAJOR.MINOR.PATCH`, with:

   - `MAJOR` version when you make incompatible API changes,
   - `MINOR` version when you add functionality in a backward-compatible manner, and
   - `PATCH` version when you make backward-compatible bug fixes [0][4].

Based on the changes you've made since the last release, decide whether the next version will increment the major, minor, or patch number. If you are adding new features without breaking backward compatibility, it's a minor update. If you're fixing bugs without adding new features or breaking anything, it's a patch update. If the changes are not backward compatible, it's a major update [0][3][4].

6. **Tagging in Git**: Once you've determined the next version number, use the `git tag` command to tag the release. If you are tagging version 1.0.0, you would use:

   ```bash
   git tag v1.0.0
   ```

   Prefixing the version number with a `v` is a common practice but not required by SemVer. However, it's widely recognized and recommended for clarity [0][3].

7. **Push Tags to Remote Repository**: After tagging your release locally, push the tag to your remote repository to share it with others. Use the command:

   ```bash
   git push origin v1.0.0
   ```

   Replace `v1.0.0` with your actual version number. If you want to push all tags at once, you can use `git push --tags` [3].

8. **Automating with CI/CD**: If you're using Continuous Integration/Continuous Deployment (CI/CD) pipelines, you can automate tagging and releasing based on specific triggers, such as a merge into the main branch. Tools like GitHub Actions, GitLab CI/CD, and others can be configured to handle versioning based on commit messages or other criteria, making the process more streamlined and reducing the potential for human error [1][2].

9. **Versioning Best Practices**: Always update your `README.md`, documentation, and any other relevant information to reflect the new version and changes made. This helps users and contributors understand what has changed and how it might affect them [4].
