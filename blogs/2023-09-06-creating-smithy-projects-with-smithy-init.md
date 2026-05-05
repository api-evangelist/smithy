---
title: "Creating Smithy Projects with Smithy Init"
url: "https://aws.amazon.com/blogs/developer/creating-smithy-projects-with-smithy-init/"
date: "Wed, 06 Sep 2023 21:57:37 +0000"
author: "Andrew Foss"
feed_url: "https://aws.amazon.com/blogs/developer/tag/smithy/feed/"
---
<p>The Smithy team is excited to announce the release of the <code>init</code> command in <a href="https://smithy.io/2.0/guides/smithy-cli/index.html">Smithy CLI</a>. This command enables developers to create new Smithy projects quickly and easily.</p> 
<p>Before the Smithy <code>init</code> command was introduced, developers had to carefully follow along with a developer guide or blog post to setup their Smithy projects. This involves copying and pasting code snippets, as well as creating various files and directories. With the introduction of the <code>init</code> command, developers can use a single command to create a project with all the necessary Smithy files, configuration files, and directories, all tailored to their use case. The new <code>init</code> command simplifies developer experience of getting started with Smithy, and it is less error-prone as it creates projects based on templates. The templates provided by the <code>init</code> command cater to different project needs as well as to serve as great examples. Developers can use these templates to explore different project setups and learn about various Smithy features.</p> 
<h2>What is Smithy?</h2> 
<p><a href="https://smithy.io/2.0/index.html">Smithy</a> is an open-source Interface Definition Language (IDL) and set of tools for building web services, created by AWS. AWS uses Smithy to model services, generate server scaffolding, generate SDKs for multiple languages, and generate AWS SDKs. Smithy enables large-scale collaboration on API’s through its extensible meta-model and pluggable design. It is purpose-built to support code generation in multiple languages, enables automatic API standards enforcement, and is protocol-agnostic. Smithy’s design is rooted in our experience from building thousands of service APIs and developing complex SDKs within Amazon. To learn more, check out the <a href="https://smithy.io/2.0/index.html">smithy.io</a> website, and please watch the <a href="https://www.youtube.com/watch?v=3GpZzu4guTE">introductory talk</a> from one of Smithy’s creators.</p> 
<h2>What is the Smithy CLI?</h2> 
<p>The <a href="https://smithy.io/2.0/guides/smithy-cli/index.html">Smithy CLI</a> allows developers to quickly iterate on their Smithy models. Using the Smithy CLI, developers can quickly initialize a Smithy project, build models, run validation, compare models for differences, query models, and more. The Smithy CLI is available to <a href="https://smithy.io/2.0/guides/smithy-cli/cli_installation.html">download</a> on macOS, Linux, and Windows platforms.</p> 
<h2>Getting Started</h2> 
<p>If you haven’t installed the Smithy CLI before, follow the <a href="https://smithy.io/2.0/guides/smithy-cli/cli_installation.html">installation guide</a>. If you have installed the Smithy CLI, run <code>smithy --version</code> in a terminal to verify that the version is <code>1.36.0</code> or above. If Homebrew was used to install the Smithy CLI, you can update it&nbsp;to the latest version by running <code>brew upgrade smithy-cli</code> in a terminal. Otherwise, please refer to the <a href="https://smithy.io/2.0/guides/smithy-cli/cli_installation.html">installation guide</a> to update.</p> 
<p>With the Smithy CLI installed, you can view the help information for the <code>init</code> command by using the <code>--help</code> flag:</p> 
<pre><code class="lang-bash">$ smithy init --help
Usage: smithy init [--help | -h] [--debug] 
                   [--quiet] [--no-color] 
                   [--force-color] [--stacktrace] 
                   [--logging LOG_LEVEL] 
                   [--template | -t quickstart-cli] 
                   [--url https://github.com/smithy-lang/smithy-examples.git] 
                   [--output | -o new-smithy-project] 
                   [--list | -l] 
Initialize a smithy project using a template</code></pre> 
<p>Next, we will demonstrate how the <code>init</code> command can be used to create a new Smithy project.</p> 
<h2>Create a new Smithy project</h2> 
<p>Let’s create a new project by calling the <code>init</code> command:</p> 
<pre><code class="lang-bash">$ smithy init --output ./new-smithy-project
Smithy project created in directory: new-smithy-project</code></pre> 
<p>This will create a new project in the <code>new-smithy-project</code> directory:</p> 
<pre><code class="lang-bash">$ tree ./new-smithy-project
./new-smithy-project
├── README.md
├── models
│   └── weather.smithy
└── smithy-build.json</code></pre> 
<p>By default, the <code>init</code> command line tool will create a new project using the <code>quickstart-cli</code> template. In the following section, we will view the available templates and how to access them.</p> 
<h2>Utilizing project templates</h2> 
<p>The <code>init</code> command offers a selection of project templates catering to different needs. Each template has its own <a href="https://smithy.io/2.0/guides/building-models/build-config.html">smithy-build.json</a> file and <a href="https://smithy.io/2.0/spec/model.html">model</a> files for serving different project needs. This is useful if you are new to Smithy and want to learn about different Smithy features. For example, you may want to use a template to create a project consisting a simple service modeled in Smithy.</p> 
<p>To view the available templates and their respective use cases, you can run the <code>init</code> command with the <code>--list</code> flag:</p> 
<pre><code class="lang-bash">$ smithy init --list   
────────────────────────────  ──────────────────────────────────────────────────────────────────────
NAME                          DOCUMENTATION
────────────────────────────  ──────────────────────────────────────────────────────────────────────
common-linting-configuration  Gradle project to create a package to define a common set of model
                              validations.
custom-annotation-trait       Gradle project for creating a package for a custom annotation trait.
custom-linter                 Gradle project to create a custom, configurable model linter.
custom-string-trait           Gradle project for creating a package for a custom string trait.
custom-structure-trait        Gradle project for creating a package for a custom structure trait.
custom-validator              Gradle project to create a custom model validator.
decorators                    Gradle project to create a custom  decorator for validation events.
filter-internal-projection    Projection that filters out internal shapes and traits.
quickstart-cli                Smithy Quick Start example weather service using the Smithy CLI.
quickstart-gradle             Smithy Quick Start example weather service using the
                              Smithy-Gradle-Plugin.
shared-model-package          Gradle project to create a shared package of common Smithy models.
smithy-to-openapi             Generate an OpenAPI spec from a Smithy model</code></pre> 
<p>Let’s try using the <code>quickstart-gradle</code> template to create a project for a <a href="https://smithy.io/2.0/quickstart.html#weather-service">weather service</a>. You can call the <code>init</code> command using the <code>--template</code> flag:</p> 
<pre><code class="lang-bash">$ smithy init --template quickstart-gradle --output ./my-new-service
Smithy project created in directory: my-new-service</code></pre> 
<p>This will create a project consisting of the Smithy model file, the <code>smithy-build.json</code> file and various Gradle config files within the <code>my-new-service</code> directory:</p> 
<pre><code class="lang-bash">$ tree ./my-new-service
./my-new-service
├── README.md
├── build.gradle.kts
├── gradle
│   └── wrapper
│       ├── gradle-wrapper.jar
│       └── gradle-wrapper.properties
├── gradle.properties
├── gradlew
├── gradlew.bat
├── models
│   └── weather.smithy
├── settings.gradle.kts
└── smithy-build.json</code></pre> 
<p>As you can see, the <code>init</code> command simplifies the initiation of a new Smithy project. We encourage you to try out the other templates to learn more about different project setups and Smithy features!</p> 
<h2>Templates repository</h2> 
<p>Behind the scenes, the <code>init</code> command checks the latest list of templates available from Smithy’s <a href="https://github.com/smithy-lang/smithy-examples">smithy-examples</a> Git repository. This open-source repository includes contributions from both AWS and community members.</p> 
<p>Developers have the flexibility to override the default repository with an alternative one. Developers may want to set up their own template repository to share Smithy models specific to their products or use case. Alternatively, they may want to create a set of private templates to be shared within an organization.</p> 
<p>The <code>--url</code> parameter specifies a custom repository:</p> 
<pre><code class="lang-bash">$ smithy init --template some-common-package --url https://github.com/user/repo.git
Smithy project created in directory: some-common-package</code></pre> 
<p>The <code>init</code> command expects a <code>smithy-templates.json</code> file at the root level of the repository. Here is an <a href="https://github.com/smithy-lang/smithy-examples/blob/main/smithy-templates.json">example</a> from smithy-examples repository for reference.</p> 
<h2>What’s Next?</h2> 
<p>With the Smithy <code>init</code> command now available, developers can quickly and easily initiate Smithy projects from a set of curated project templates. Developers can also utilize the templates to explore different project setups and learn about various Smithy features.</p> 
<p>We encourage you to try out the Smithy <code>init</code> command and tell us your thoughts by contacting us on <a href="https://github.com/smithy-lang/smithy">GitHub</a>. Please don’t hesitate to create an <a href="https://github.com/smithy-lang/smithy/issues">issue</a> or a <a href="https://github.com/smithy-lang/smithy/pulls">pull request</a> if you have ideas for improvements.</p>
