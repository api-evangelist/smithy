---
title: "Introducing Smithy for Python"
url: "https://aws.amazon.com/blogs/developer/introducing-smithy-for-python/"
date: "Fri, 04 Aug 2023 15:18:08 +0000"
author: "Jordon Phillips"
feed_url: "https://aws.amazon.com/blogs/developer/tag/smithy/feed/"
---
<p>AWS is excited to announce a preview of <a href="https://github.com/smithy-lang/smithy-python">Smithy client generation</a> for Python. This tooling will enable developers to generate clients in type-hinted Python in the same model-driven manner that AWS has used to develop its services for more than a decade. Writing and maintaining hand-written clients for a web service is both time-consuming and error-prone, problems which are both solved by generating clients with an unambiguous specification. We have made <a href="https://github.com/smithy-lang/smithy-python">Smithy for Python</a> open-source to allow any developer to confidently and quickly generate their own clients.</p> 
<h2>What is Smithy?</h2> 
<p><a href="http://smithy.io/">Smithy</a> is an open-source toolchain that we’ve standardized on to build new AWS SDKs, model new AWS services and features, and to generate clients. Smithy includes a protocol-agnostic Interface Definition Language (IDL) for generating clients, servers, documentation, and other artifacts. To learn more, check out <a href="http://smithy.io/">smithy.io</a>, and please watch the <a href="https://www.youtube.com/watch?v=3GpZzu4guTE">introductory talk</a> from Michael Dowling, Smithy’s Principal Engineer.</p> 
<h2>What’s included?</h2> 
<p>A code generated Python client will have the following features:</p> 
<ul> 
 <li>Code generated and type hinted – Code for clients, operations, and all the data they contain will be code generated into python classes. Along with type hints, this will allow customers to easily lint and debug their code using tools like mypy or other type analysis tooling.</li> 
 <li>Interceptors – Interceptors are the new extensibility feature that replace the <a href="https://botocore.amazonaws.com/v1/documentation/api/latest/topics/events.html">botocore event system</a>. They’re designed to be more reliable and to be consistent across all of Smithy’s supported languages.</li> 
 <li>Async first – Async is <a href="https://www.jetbrains.com/lp/devecosystem-2021/python/#Python_which-of-the-following-frameworks-libraries-do-you-use-in-addition-to-python">increasingly important</a> in the Python ecosystem, and since clients spend much of their time blocked on IO they’re the perfect use case for it. All operations will therefore be generated as async functions.</li> 
 <li>Configurable components – All of the core components of the client will be fully configurable. If you want to use tornado as the underlying http library, for example, you can easily do that.</li> 
</ul> 
<p>Currently protocol support is limited to the <a href="https://smithy.io/2.0/aws/protocols/aws-restjson1-protocol.html#aws-protocols-restjson1-trait">restJson1</a> protocol, which makes use of <a href="https://smithy.io/2.0/spec/http-bindings.html">http bindings</a> with a JSON body. It’s widely used among Amazon’s services, such as API Gateway and Lambda. For authentication, <a href="https://smithy.io/2.0/spec/authentication-traits.html#smithy-api-httpapikeyauth-trait">httpApiKey</a> auth is currently supported.</p> 
<h2>Getting started</h2> 
<p>There are three prerequisites to client generation. The first prerequisite is Python, which has a minimum required version of 3.11. The second prerequisite is Java 17, which is only required to build the generator. In the future, when the generator is published to a package repository, this will not be required. It is also not required to run the generated Python client. The last prerequisite you’ll need is the <a href="https://smithy.io/2.0/guides/smithy-cli/cli_installation.html">Smithy CLI</a>.</p> 
<p>To build the generator, clone the <a href="https://github.com/awslabs/smithy-python">smithy-python GitHub repo</a> and run <code>make install-java-components</code>. This will build the generator and make it available on your local machine.</p> 
<p>Now that you’ve built the generator, you need a Smithy model. In this example, you’ll use a small model for a service that accepts a string message in a JSON body and returns it unmodified. Save the following model contents to a file called <code>main.smithy</code> in a directory called <code>model</code>.</p> 
<pre><code class="lang-smithy">$version: "2.0"

namespace com.example

use aws.protocols#restJson1

@restJson1
service EchoService {
    operations: [EchoMessage]
}

@http(
    method: "POST"
    uri: "/echo"
)
operation EchoMessage {
    input := {
        message: String
    }
    output := {
        message: String
    }
}</code></pre> 
<p>Now you need to create a configuration file named <code>smithy-build.json</code>. This configuration file lets you specify dependencies, configure plugins, and create projections, which are different views of your model catered to a specific purpose. Additional information on what you can do with this configuration file and projections is provided in the <a href="https://smithy.io/2.0/guides/building-models/build-config.html">smithy-build docs</a>. The following <code>smithy-build.json</code> file adds a few necessary dependencies and creates a projection to build the python client in. Create your <code>smithy-build.json</code> file in the same directory as your model folder with the following contents:</p> 
<pre><code class="lang-json">{
    "version": "1.0",
    "sources": ["model"],
    "maven": {
        "dependencies": [
            "software.amazon.smithy:smithy-model:1.34.0",
            "software.amazon.smithy:smithy-aws-traits:1.34.0",
            "software.amazon.smithy.python:smithy-python-codegen:0.1.0"
        ]
    },
    "projections": {
        "python-client": {
            "plugins": {
                "python-client-codegen": {
                    "service": "com.example#EchoService",
                    "module": "echo",
                    "moduleVersion": "0.1.0"
                }
            }
        }
    }
}</code></pre> 
<p>Your file structure now should now look like this:</p> 
<div class="hide-language"> 
 <pre><code class="lang-bash">.
├── model
│   └── main.smithy
└── smithy-build.json</code></pre> 
</div> 
<p>The only step left to generate your client is to run <code>smithy build</code>. Your new client will now be available as an installable package in <code>build/python-client/python-client-codegen</code>. To use it, you’ll need to install a few Python dependencies by running <code>make install-python-components</code> from the root of the smithy-python repository. The following snippet shows how you can use your new client:</p> 
<pre><code class="lang-python">import asyncio

from echo.client import EchoService
from echo.config import Config
from echo.models import EchoMessageInput


async def main() -&gt; None:
    client = EchoService(Config(endpoint_uri="https://example.com/"))
    response = await client.echo(EchoMessageInput(message="spam"))
    print(response.message)


if __name__ == "__main__":
    asyncio.run(main())</code></pre> 
<h2>What’s next for Smithy for Python?</h2> 
<p>We will be delivering new features to the code generator to make generating custom clients easier. We will also be delivering support for more protocols and features to enable generation for more kinds of services.</p> 
<h2>Feedback</h2> 
<p>We encourage you to try out Smithy for Python to build your own clients. If you have any question, comments, concerns, or ideas, please open an issue or bug report on the <a href="https://github.com/smithy-lang/smithy-python">smithy-python GitHub repository</a>. If you discover a potential security issue, we ask that you notify AWS/Amazon Security via our <a href="http://aws.amazon.com/security/vulnerability-reporting/">vulnerability reporting page</a> instead.&nbsp;We appreciate any feedback we receive, and will use it to make the next versions of the clients better.</p>
