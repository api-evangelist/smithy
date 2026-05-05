---
title: "Introducing the Smithy CLI"
url: "https://aws.amazon.com/blogs/developer/introducing-the-smithy-cli/"
date: "Thu, 27 Apr 2023 16:43:16 +0000"
author: "Hayden Baker"
feed_url: "https://aws.amazon.com/blogs/developer/tag/smithy/feed/"
---
<p>The Smithy team is excited to announce the official release of the <a href="https://smithy.io/2.0/guides/smithy-cli/index.html">Smithy CLI</a>.&nbsp;<a href="https://smithy.io">Smithy</a> is an open-source Interface Definition Language (IDL) for web services created by AWS. AWS uses Smithy to model services, generate server scaffolding and rich clients in multiple languages, and generate the <a href="https://aws.amazon.com/developer/tools/">AWS SDKs</a>. Smithy enables large-scale collaboration on APIs through its extensible meta-model and pluggable design. It is purpose-built for enabling code generation in multiple languages, can be extended with custom traits, enables automatic API standards enforcement, and is protocol-agnostic. Smithy’s design is rooted in our experience building thousands of service APIs and developing complex SDKs within Amazon. To learn more, check out <a href="https://smithy.io">smithy.io</a>, and please watch the <a href="https://www.youtube.com/watch?v=3GpZzu4guTE">introductory talk</a> from Michael Dowling, Smithy’s Principal Engineer.</p> 
<p>Currently, most developers build their Smithy models using <a href="https://gradle.org/">Gradle</a> and the&nbsp;<a href="https://smithy.io/2.0/guides/building-models/gradle-plugin.html">smithy-gradle plugin</a>. However, a developer would need to have an installation of <a href="https://en.wikipedia.org/wiki/Java_(programming_language)">Java</a>, and knowledge of how Gradle tooling works just to build their models. This is overly complex for developers who aren’t already familiar with these tools, and detracts from the intended experience of modeling with Smithy.</p> 
<p>With the Smithy CLI, developers can build their models quicker and with a single command, without any knowledge of Java or Gradle. The Smithy CLI is now available to download on MacOS, Linux, and Windows platforms.</p> 
<h2 id="temp:C:XVZ078122a694e9416a9b35bf908">Getting Started</h2> 
<p>The Smithy CLI enables you to quickly iterate on your Smithy models. With this tool, you can easily build your models, run ad-hoc validation on your models, compare models for differences, and query them.&nbsp;To install the Smithy CLI on MacOS with <a href="https://brew.sh/">Homebrew</a>, you can run the following commands:</p> 
<pre><code class="lang-bash">$&nbsp;brew tap smithy-lang/tap
$&nbsp;brew install smithy-cli</code></pre> 
<p>For install instructions on other platforms or for detailed instructions on installation, you can view the <a href="https://smithy.io/2.0/guides/smithy-cli/cli_installation.html">installation guide</a>. Now that you have the Smithy CLI installed, you can view the ‘help’ information by using the <code>--help</code> flag:</p> 
<pre><code class="lang-bash">$ smithy --help
Usage: smithy [-h | --help] [--version] &lt;command&gt; [&lt;args&gt;]
 
Available commands:
    validate    Validates Smithy models.
    build       Builds Smithy models and creates plugin artifacts for each projection found in smithy-build.json.
    diff        Compares two Smithy models and reports differences.
    ast         Reads Smithy models in and writes out a single JSON AST model.
    select      Queries a model using a selector.
    clean       Removes Smithy build artifacts.
    migrate     Migrate Smithy IDL models from 1.0 to 2.0 in place.</code></pre> 
<p>You can also call a command with the&nbsp;<code>--help</code> flag appended to view command-specific information:</p> 
<pre><code class="lang-bash">$ smithy build --help
Usage: smithy build [--help | -h]
                    [--debug] [--quiet] [--no-color]
                    [--force-color] [--stacktrace]
                    [--logging LOG_LEVEL]
                    [--config | -c CONFIG_PATH...]
                    [--no-config] [--severity SEVERITY]
                    [--allow-unknown-traits]
                    [--output OUTPUT_PATH]
                    [--projection PROJECTION_NAME]
                    [--plugin PLUGIN_NAME] [&lt;MODELS&gt;]

Builds Smithy models and creates plugin artifacts for each projection found in
smithy-build.json.
...</code></pre> 
<p>Throughout this post, we’ll be using the Smithy CLI on our <a href="https://smithy.io/2.0/quickstart.html#complete-example">example Weather model</a> from the official Smithy documentation. You can follow along with the example or use your own models. Your local workspace should look like:</p> 
<pre><code class="lang-bash">~/weather $ tree .
.
├── model
│   ├── weather.smithy
├── smithy-build.json</code></pre> 
<h3 id="temp:C:XVZe42b74d1fa844c609c61d0464">Building Models</h3> 
<p><a href="https://smithy.io/2.0/spec/model.html">Models</a> are at the core of the Smithy language, and building models is the core functionality of the Smithy CLI. The build command performs validation and generates build artifacts, such as the <a href="https://smithy.io/2.0/spec/json-ast.html">JSON abstract syntax tree (AST)</a>, other models, code, and more.</p> 
<p>Let’s build our basic example weather model:</p> 
<pre><code class="lang-bash">~/weather $ smithy build model/

SUCCESS: Validated 240 shapes

Validated model, now starting projections...

──  source  ────────────────────────────────────────────────────────────────────
Completed projection source (240): weather/build/smithy/source

Summary: Smithy built 1 projection(s), 3 plugin(s), and 4 artifacts</code></pre> 
<p>Looking into the build artifacts (under <code>weather/build/smithy/source</code>), we’ll find several files based on our build configuration (<code>smithy-build.json</code>):</p> 
<pre><code class="lang-bash">~/weather $ tree .
.
├── build
│   └── smithy
│       └── source
│           ├── build-info
│           │   └── smithy-build-info.json   -- build metadata (projections, validation, ...)
│           ├── model
│           │   └── model.json               -- JSON AST
│           └── sources
│               ├── manifest                 -- an inventory of the build's smithy models
│               └── weather.smithy           -- a copy of our example weather model
├── model
│   └── weather.smithy
└── smithy-build.json</code></pre> 
<p>In this case, no build behaviors outside of the defaults were configured in our <code>smithy-build.json</code>, so building our example weather model rendered only the JSON AST of the model. For more detailed information on configuring builds and build artifacts, check out the <a href="https://smithy.io/2.0/guides/building-models/build-config.html">smithy-build</a> guide.</p> 
<h3 id="temp:C:XVZb110384cb8ca48f7a6a9001aa">Linting and Validating Models</h3> 
<p>Code validation tools, such as <a href="https://checkstyle.org/">Checkstyle</a> and <a href="https://spotbugs.github.io/">SpotBugs</a>, help developers avoid common pitfalls and bugs, while also ensuring that the code automatically adheres to standards of an organization. In Smithy, validators help prevent style issues and common mistakes, so that your team can focus on more important design considerations.</p> 
<p>We can use a&nbsp;<a href="https://smithy.io/2.0/spec/selectors.html">selector</a> to choose which shapes to perform some custom validation on. As a best practice, we want to enforce documentation on all our operations, so we will add our selector to a validator in our model file:</p> 
<pre><code class="lang-smithy">// --- model/weather.smithy ---
$version: "2"

metadata validators = [{
    name: "EmitEachSelector"
    id: "OperationMissingDocumentation"
    message: "This operation is missing documentation"
    namespaces: ["example.weather"]
    configuration: {
        selector: """
            operation:not([trait|documentation])
        """
    }
}]

namespace example.weather
...</code></pre> 
<p>We’ll then run the validation on our weather model with the&nbsp;<code>validate</code> command:</p> 
<pre><code class="lang-bash">~/weather $ smithy validate model/

──  DANGER  ────────────────────────────────────── OperationMissingDocumentation
Shape: example.weather#GetCity
File:  model/weather.smithy:46:1

45| @readonly
46| operation GetCity {
  | ^

This operation is missing documentation


──  DANGER  ────────────────────────────────────── OperationMissingDocumentation
Shape: example.weather#ListCities
File:  model/weather.smithy:92:1

88| // The paginated trait indicates that the operation may
89| // return truncated results.
90| @readonly
91| @paginated(items: "items")
92| operation ListCities {
  | ^

This operation is missing documentation
...

FAILURE: Validated 240 shapes (DANGER: 4)</code></pre> 
<p>With the ability to define custom validation rules like this, you can enforce a common standard for Smithy models, and be assured that best practices are upheld. For more information on validation in Smithy, please check out the&nbsp;<a href="https://smithy.io/2.0/spec/model-validation.html">Validation</a> and <a href="https://smithy.io/2.0/guides/model-linters.html">Linting</a> guides.</p> 
<h3 id="temp:C:XVZb4b7a3194d6244e4bd471369f">Differencing Models</h3> 
<p>If we make changes to our model, we’ll check for backward compatibility issues using the smithy <code>diff</code> command. For more information on the diff’ing process, see <a href="https://github.com/awslabs/smithy/tree/main/smithy-diff">smithy-diff</a>.</p> 
<p>Let’s modify the <code>time</code> member in the&nbsp;<code>GetCurrentTimeOutput</code> shape in the example model. First, copy the model and rename the copied model (<code>weather-old.smithy</code>):</p> 
<pre><code class="lang-bash">$ cp model/weather.smithy weather-old.smithy</code></pre> 
<p>Now, make the change to the shape in the “new” model file:</p> 
<pre><code class="lang-smithy">// --- model/weather.smithy ---
...
@output
structure GetCurrentTimeOutput {
    @required
    time: String
}
...</code></pre> 
<p>Now, let’s perform a compatibility check against this change:</p> 
<pre><code class="lang-bash">~/weather $ smithy diff --old weather-old.smithy --new model/weather.smithy

──  DIFF  ERROR  ─────────────────────────────────────────── ChangedMemberTarget
Shape: example.weather#GetCurrentTimeOutput$time
File:  model/weather.smithy:138:5

136| structure GetCurrentTimeOutput {
···|
138|     time: String
   |     ^

The shape targeted by the member example.weather#GetCurrentTimeOutput$time
changed from smithy.api#Timestamp (timestamp) to smithy.api#String (string).
The type of the targeted shape changed from timestamp to string.

FAILURE: Validated 240 shapes (ERROR: 1)</code></pre> 
<p>We made a breaking change (<code>ERROR</code>) by changing the datatype of a shape that already had a previous definition. This is dangerous because of the break in backwards-compatibility between versions of your model. A client using your old model would no longer be able to safely call the&nbsp;<code>GetCurrentTime</code> operation. For more information on safely evolving your models, see the&nbsp;<a href="https://smithy.io/2.0/guides/evolving-models.html">model evolution</a> guide.</p> 
<h3 id="temp:C:XVZ305327f9436e4729bd483554a">Querying Models</h3> 
<p><a href="https://smithy.io/2.0/spec/selectors.html">Selectors</a> are a powerful way to query your model for different shapes. They can be used to build custom model validation logic through <a href="https://smithy.io/2.0/spec/model-validation.html#built-in-validators">validators</a>, or to define where certain <a href="https://smithy.io/2.0/spec/model.html#trait-trait">traits</a> can be applied. With the Smithy CLI, the process of working with and developing selectors is streamlined.</p> 
<p>To query all of the shapes in our weather model, we can use the following statement:</p> 
<pre><code class="lang-bash">~/weather $ smithy select --selector '[id|namespace = "example.weather"]' model/
example.weather#City
example.weather#CityCoordinates
example.weather#CityCoordinates$latitude
example.weather#CityCoordinates$longitude
...
example.weather#Weather</code></pre> 
<p>What if you want to find all undocumented operations in our model? We can answer this question by using the following statement:</p> 
<pre><code class="lang-bash">~/weather $ smithy select --selector 'operation:not([trait|documentation])' model/
example.weather#GetCity
example.weather#GetCurrentTime
example.weather#GetForecast
example.weather#ListCities</code></pre> 
<p>You can iterate on this process as much as needed to answer questions about your model – once you have your selector, you can then use it in validation. For a greater understanding of Smithy selectors, please read through the <a href="https://smithy.io/2.0/spec/selectors.html">Selectors</a> guide.</p> 
<h3 id="temp:C:XVZb7b7d4e95a6347248d6b0465d">Customizing Builds</h3> 
<p>One of the most powerful features of the build process is the extensibility. You can customize your build with the&nbsp;<code>smithy-build.json</code> file, adding projections or plugins as you desire. Let’s use the Smithy CLI to generate a client in TypeScript by configuring it to use the <a href="https://github.com/awslabs/smithy-typescript">smithy-typescript code-generator</a>.&nbsp;For a deeper dive into code-generation, check out the&nbsp;<a href="https://smithy.io/2.0/guides/using-code-generation/index.html">code generation guide</a>.</p> 
<p>Let’s make a new and more simple service model for demonstration purposes. Our workspace should have the following structure:</p> 
<div class="hide-language"> 
 <pre><code class="lang-bash">time
├── model
│   └── time.smithy      -- our new model file for the time service
└── smithy-build.json    -- a new smithy-build.json file</code></pre> 
</div> 
<p>Our new model file, <code>time.smithy</code>, should have the following code:</p> 
<pre><code class="lang-smithy">// --- model/time.smithy ---
$version: "2"

namespace example.time

service Time {
    version: "0.0.1"
    operations: [GetCurrentTime]
}

/// An operation for getting the current time
@readonly
@http(code: 200, method: "GET", uri: "/time",)
operation GetCurrentTime {
    output := { 
        @required 
        @timestampFormat("date-time") 
        time: Timestamp 
    }
}</code></pre> 
<p>To generate the typescript client for the time service, our build configuration file should contain the TypeScript plugin and parameters to produce code for the time model:</p> 
<pre><code class="lang-json">// --- smithy-build.json ---
{
    "version": "1.0",
    "projections": {
        "source": {
            "plugins": {
                "typescript-codegen": {
                    "service": "example.time#Time",
                    "package": "@example/time",
                    "packageVersion": "0.0.1"
                }
            }
        }
    },
    "maven": {
        "dependencies": [
            "software.amazon.smithy:smithy-model:1.30.0",
            "software.amazon.smithy.typescript:smithy-typescript-codegen:0.14.0"
        ]
    }
}</code></pre> 
<p>The build will apply the typescript code-generator plugin to generate code, and will resolve the dependencies for the generator from Maven, as indicated by the&nbsp;<code>maven</code> section in the configuration. Let’s build our model and generate the code:</p> 
<pre><code class="lang-bash">~/time $ smithy build model/

SUCCESS: Validated 378 shapes

Validated model, now starting projections...

[WARNING] Unable to find a protocol generator for example.time#Time: Unable to derive the protocol setting of the service `example.time#Time`
   because no protocol definition traits were present. You need to set an explicit `protocol` to generate in smithy-build.json to generate this service.
──  source  ────────────────────────────────────────────────────────────────────
Completed projection source (378): time/build/smithy/source

Summary: Smithy built 1 projection(s), 4 plugin(s), and 22 artifacts</code></pre> 
<p>Several TypeScript source files and configuration files are generated for the time client under the&nbsp;<code>build/smithy/source/typescript-codegen</code> directory in the workspace. A warning was printed because our <code>Time</code> service does not specify a protocol, but we can safely ignore this for demonstration purposes. Let’s take a look at a small snippet of the generated code in the&nbsp;<code>build/smithy/source/typescript-codegen/src/Time.ts</code> file:</p> 
<pre><code class="lang-typescript">// smithy-typescript generated code
import { TimeClient } from "./TimeClient";
import {
  GetCurrentTimeCommand,
  GetCurrentTimeCommandInput,
  GetCurrentTimeCommandOutput,
} from "./commands/GetCurrentTimeCommand";
import { HttpHandlerOptions as __HttpHandlerOptions } from "@aws-sdk/types";

export class Time extends TimeClient {
  /**
   * An operation for getting the current time
   */
  public getCurrentTime(
    args: GetCurrentTimeCommandInput,
    options?: __HttpHandlerOptions,
  ): Promise&lt;GetCurrentTimeCommandOutput&gt;;
  public getCurrentTime(
    args: GetCurrentTimeCommandInput,
    cb: (err: any, data?: GetCurrentTimeCommandOutput) =&gt; void
  ): void;
  public getCurrentTime(
    args: GetCurrentTimeCommandInput,
    options: __HttpHandlerOptions,
    cb: (err: any, data?: GetCurrentTimeCommandOutput) =&gt; void
  ): void;
  public getCurrentTime(
    args: GetCurrentTimeCommandInput,
    optionsOrCb?: __HttpHandlerOptions | ((err: any, data?: GetCurrentTimeCommandOutput) =&gt; void),
    cb?: (err: any, data?: GetCurrentTimeCommandOutput) =&gt; void
  ): Promise&lt;GetCurrentTimeCommandOutput&gt; | void {
    const command = new GetCurrentTimeCommand(args);
    if (typeof optionsOrCb === "function") {
      this.send(command, optionsOrCb)
    } else if (typeof cb === "function") {
      if (typeof optionsOrCb !== "object")
        throw new Error(`Expect http options but get ${typeof optionsOrCb}`)
      this.send(command, optionsOrCb || {}, cb)
    } else {
      return this.send(command, optionsOrCb);
    }
  }
}</code></pre> 
<p>From the preceding code snippet, we can observe the <code>GetCurrentTime</code> operation in our model was used in the code-generator to create a method, <code>getCurrentTime</code>, in the TypeScript client for the time service. If this were a real service, a customer could use this method to make the request to our service in their own TypeScript packages.&nbsp;Using just the Smithy CLI, we were able to build our simple time model, and generate some basic client code in TypeScript – to do this before, you would have needed to use Gradle and be familiar with the Gradle ecosystem to manage your project.</p> 
<h3 id="temp:C:XVZ18848390dd8945ccbd030ad1d">What’s Next?</h3> 
<p>Start using Smithy CLI today to streamline your experience when building models with Smithy. Smithy CLI makes it easy to build, validate, and rapidly iterate on your models.</p> 
<p>We will be continuously making improvements to the CLI to enhance the developer experience, so tell us how you like using the Smithy CLI by leaving a comment or by contacting us on&nbsp;<a href="https://github.com/awslabs/smithy">GitHub</a>. Please don’t hesitate to create an&nbsp;<a href="https://github.com/awslabs/smithy/issues">issue</a> or a&nbsp;<a href="https://github.com/awslabs/smithy/pulls">pull request</a> if you have ideas for improvements. Check out the Smithy documentation and user-guides over at <a href="https://smithy.io">smithy.io</a> to learn more about how to leverage the full potential of Smithy.</p> 
<p><strong>About the author:</strong></p> 
<footer> 
 <div class="blog-author-box"> 
  <div class="blog-author-image">
   <img alt="Hayden Baker" class="alignleft wp-image-11636" height="160" src="https://d2908q01vomqb2.cloudfront.net/0716d9708d321ffb6a00818614779e779925365c/2023/04/24/Hayden-headshot-black.png" width="120" />
  </div> 
  <h3 class="lb-h4">Hayden Baker</h3> 
  <p style="text-align: left;">Hayden is a software development engineer on the Smithy team at AWS. He enjoys working on projects and tools that aim to improve the developer experience. You can find him on GitHub @haydenbaker.</p> 
 </div> 
</footer>
