---
layout: post
status: publish
published: true
title: GitHub and dependabot version updates
author: Manuel Molina
about_author: "/us/manuel/readme.html"
author_email: mmc@pocosmhz.org
date: '2023-08-08 13:48:00 +0100'
tags:
- CI/CD
comments: true
---
In this post I'll discuss the different options that GitHub offers to the development teams in order to manage your project security and dependencies.

Under the *Code Security and Analysis* tab of the *Security* section of your project's settings, you can enable different levels of code analysis.

You have the main documentation here: [Keeping your supply chain secure with Dependabot](https://docs.github.com/en/code-security/dependabot).

However, I'll try to detail the three main settings I suggest you to enable in all your projects.

# Dependabot alerts
The first setting you can enable in all cases is *Dependabot alerts*:

> Enable Dependabot alerts to be generated when a new vulnerable dependency or malware is found in one of your repositories.

And what will you be warned about?

> A vulnerability is a problem in a project's code that could be exploited to damage the confidentiality, integrity, or availability of the project or other projects that use its code. Vulnerabilities vary in type, severity, and method of attack.

> Dependabot scans code when a new advisory is added to the GitHub Advisory Database or the dependency graph for a repository changes. When vulnerable dependencies or malware are detected, Dependabot alerts are generated.

Go [here](https://docs.github.com/en/code-security/dependabot/dependabot-alerts/configuring-dependabot-alerts#managing-dependabot-alerts-for-your-repository) to see how to enable it for one repository. In the same page you'll find directions on how to do it for your personal account or organization.

Once you have performed this step, you'll see that a new setting has been automatically enabled: *Dependency graph*. You can learn more about the dependency graph [here](https://docs.github.com/en/code-security/supply-chain-security/understanding-your-software-supply-chain/about-the-dependency-graph).

Once you have it enabled, you can go to your project's page and browse *Insights* -> *Dependency graph* to get more information about the current dependency list detected in your project. Also, from there you will also see which of them have a current security issue, with the severity of the issue.

Please see the next section.

# Dependabot security updates
In order to know and proactively treat the security issues GitHub finds in your repository, please enable the option next to *Dependabot alerts*, that is, *Dependabot security updates*.

> Dependabot can fix vulnerable dependencies for you by raising pull requests with security updates.

Read more about this setting [here](https://docs.github.com/en/code-security/dependabot/dependabot-security-updates/about-dependabot-security-updates).

It is important to notice that pull requests are open by creating a new branch, and to be merged onto the main branch of the repository. **However, you can edit the PR and merge to a different branch for your convenience**.

I strongly suggest that, in order to better trace the changes with your own branch naming conventions and third party tools like Jira.

Once you've enabled the setting, you can either check the newly created PRs when there's one, in the *Pull requests* section of your GitHub repository's main section.

At any moment, you can also check the current list of alerts issued by Dependabot by browsing *Security* -> *Vulnerability alerts* -> *Dependabot* from your repository's main page.

Let's see a graphic example for this:

![Vulnerability alert list](/content/images/2023-08-08-github-dependabot-version-updates/dependabot-alert-list.png)

# Dependabot version updates
The last step I strongly recommend you to take for all your code repositories is to enable [Dependabot version updates](https://docs.github.com/en/code-security/dependabot/dependabot-version-updates):

> You can use Dependabot to automatically keep the dependencies and packages used in your repository updated to the latest version, even when they don’t have any known vulnerabilities.

I find this specially usefull for development teams that don't have time, but need to be up to date about the bigger picture of the dependency list of their code base.

Let me extend on this. When your project grows you tend to prioritize feature releases and other tasks. However, as you make extensive use of libraries and modules of any kind, the chances of not noticing the release of newer versions increases. Moreover, a deprecated library would go unnoticed if there is no security issue involved.

For these reasons, and by enabling this setting, you'll be able to get that bigger picture by telling GitHub to specifically look at particular files related to your [package ecosystem](https://docs.github.com/en/code-security/dependabot/dependabot-version-updates/configuration-options-for-the-dependabot.yml-file#package-ecosystem). 

As this feature is really powerful, in order to configure it you have to manually craft a descriptive file named `.github/dependabot.yml` for your repository. See available options [here](https://docs.github.com/en/code-security/dependabot/dependabot-version-updates/configuration-options-for-the-dependabot.yml-file). You can name more than one ecosystem inside the same repository. This is useful if you have different ecosystems involved, like a Ruby on Rails application that involves a JavaScript frontend app served as static content.

One sample `dependabot.yml` file for a single ecosystem looks like this:

{% highlight YAML %}
# To get started with Dependabot version updates, you'll need to specify which
# package ecosystems to update and where the package manifests are located.
# Please see the documentation for all configuration options:
# https://docs.github.com/github/administering-a-repository/configuration-options-for-dependency-updates

version: 2
updates:
  - package-ecosystem: "bundler" # See documentation for possible values
    directory: "/" # Location of package manifests
    schedule:
      interval: "weekly"
{% endhighlight %}

Please check the aforementioned documentation in order to know which files are checked for every package ecosystem. There are plenty of them supported, like `bundler`, `pip`, `yarn`, or `npm`.

Once configured, you can browse to *Insights* -> *Dependency graph* -> *Dependabot* and click to see the files involved:

![Monitored dependency files](/content/images/2023-08-08-github-dependabot-version-updates/monitored-dependency-files.png)

# Other automated security measures
Depending on your GitHub plan and eligibility of your repository, you can opt to additional automated security measures.

I'll describe them briefly below.

## Code scanning
This feature is only available for [eligible](https://docs.github.com/en/code-security/code-scanning/automatically-scanning-your-code-for-vulnerabilities-and-errors/configuring-default-setup-for-code-scanning-at-scale#eligible-repositories-for-codeql-default-setup) repositories only:

> Code scanning is available for all public repositories on GitHub.com. Code scanning is also available for private repositories owned by organizations that use GitHub Enterprise Cloud and have a license for GitHub Advanced Security. For more information, see "About GitHub Advanced Security."

Once you enable this feature with the default settings, you'll be offered a dialog like this one:

![CodeQL default configuration](/content/images/2023-08-08-github-dependabot-version-updates/codeql-default-configuration.png)

You click *Enable CodeQL* and you're good to go. It will take some time to take effect if your code base is big, you'll be warned.

When it's done, you can check the current list of code scanning alerts issued by browsing *Security* -> *Vulnerability alerts* -> *Code scanning* from your repository's main page.

## Secret scanning
Best practices tell you that you must avoid including secrets in any clear text way in your code repositories.

You can make GitHub watch your repositories for leaked secrets in code in two ways.

### Secret scanning alerts for partners
This feature is intended to organizations or users that provide a public library or module, used by others.

If you or some organization happen to publish a module that contains a secret matching one of the patterns, the partner organization will get an alert from GitHub. They can decide what to do with the security leak.

This is clearly useful if you happen to provide API keys for your customers and one of them goes in the wild in a public repo. You'll have the option to deal with the security incident with your customer first hand, or **even automatically revoke the secret if so you decide**.

You can read more about secret scanning alerts for partners [here](https://docs.github.com/en/enterprise-cloud@latest/code-security/secret-scanning/about-secret-scanning#about-secret-scanning-alerts-for-partners).

Please see more about the supported secret patterns [here](https://docs.github.com/en/enterprise-cloud@latest/code-security/secret-scanning/secret-scanning-patterns#supported-secrets).

### Secret scanning alerts for users
This feature is available for all public repositories and for private and internal repositories that are owned by organizations using GitHub Enterprise Cloud with a license for GitHub Advanced Security.

It will analyze not only code but also comments and issue descriptions. If a secret pattern is found, you'll be notified.

If some code is pushed and secrets are detected, it triggers an alert as well.

Same secret patterns from previous section apply here.

# Wrapping up
After this post I hope you have a better understanding of the free tools GitHub provides for security and code coverage.

Most of it can be quickly implemented and help every team to better manage a project. On the other hand, it makes no harm to use it together with other tools of your choice.
