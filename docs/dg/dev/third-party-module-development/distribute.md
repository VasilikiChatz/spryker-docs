---
title: How to Publish a Module on Packagist
description: How to Publish a Module on Packagist
last_updated: Jun 7, 2024
template: howto-guide-template

---

# How to Publish a Module on Packagist

## Prerequisites
1. **PHP installed** on your machine.
2. **Composer installed** on your machine.
3. A **Git repository** created in the previous step.

## Step-by-Step Instructions

### 2. Initialize Composer
- Navigate to your package directory (Git repository directory created on the previous page) in the terminal:
  ```bash
  cd my-package
  ```

- Run `composer init` to create a `composer.json` file and follow the prompts:
  ```bash
  composer init
  ```
  You'll be prompted to fill out details like:
    - Package name (e.g., `vendor/package-name`)
    - Description
    - Author
    - Minimum stability
    - Package type
    - License
    - Dependencies (you can skip if not needed)

### 3. Add the Package Metadata
Ensure your `composer.json` file includes all necessary metadata. Here’s an example of what it might look like:

{% info_block infoBox %}
   Make sure that all the module dependencies are mentioned in `require` section.
{% endinfo_block %}

```json
{
    "name": "vendor/package-name",
    "description": "A brief description of your package",
    "type": "library",
    "license": "MIT",
    "authors": [
        {
            "name": "Your Name",
            "email": "your-email@example.com"
        }
    ],
    "require": {
        "php": ">=8.1"
    },
    "autoload": {
        "psr-4": {
            "Vendor\\Package\\": "src/"
        }
    }
}
```

### 4. Commit Your Package to a Git Repository
- In the next steps we assume that you already have a git repository with the module:

- Make sure that you've ignored all the non-module files by creating `.gitignore` file with the following minimal content:

  ```text
  # tooling
  vendor/
  composer.lock
  ```

- Add all files to the repository:
  ```bash
  git add .
  ```

- Commit your files:
  ```bash
  git commit -m "Added composer.json"
  ```

- Push your changes to the remote repository:
  ```bash
  git push -u origin master
  ```

- Add a new tag
  This can be done in several ways:
  1. Using github interface https://docs.github.com/en/repositories/releasing-projects-on-github/managing-releases-in-a-repository by creating a new release. (Recommended)
  2. Manually
      ```bash
      git tag v1.0.0
      git push origin v1.0.0
      ```

### 5. Submit Your Package to Packagist
- Go to [Packagist](https://packagist.org/).
- Log in with your GitHub account or create an account on Packagist.

- Once logged in, click on "Submit".

- Enter the URL of your Git repository and click "Check".

- After verification, click "Submit".

### 6. Maintain Your Package
- Each time you make changes to your package, remember to push the changes to your Git repository and create a new tag (release).

### 7. Update Composer Metadata (Optional)
As your package evolves, you might need to update your `composer.json` file with new dependencies or other metadata. After making changes, ensure you commit and push these updates to your Git repository.

### 8. Install package via composer
Run `composer require vendor/package-name` to install the published package to your project.

### 9. Monitor and Respond to Issues
Monitor the issues reported on your Git repository hosting service and respond to feedback from users to improve your package.

## Summary
By following these steps and referring to the screenshots provided for guidance, you can successfully create, manage, and publish a PHP package on Packagist, making it available for others to use via Composer. Remember to keep your repository well-documented and maintain good version control practices to help users understand and effectively use your package.
