---
title: "Smithy Kotlin client code generation now generally available"
url: "https://aws.amazon.com/blogs/developer/smithy-kotlin-client-code-generation-now-generally-available/"
date: "Thu, 02 Apr 2026 15:24:42 +0000"
author: "Omar Perez"
feed_url: "https://aws.amazon.com/blogs/developer/tag/smithy/feed/"
---
<p><a href="https://github.com/smithy-lang/smithy-kotlin" rel="noopener noreferrer" target="_blank">Smithy Kotlin</a>&nbsp;client code generation is now generally available. With Smithy Kotlin, you can keep client libraries in sync with evolving service APIs. By using client code generation, you can reduce repetitive work and instead, automatically create type-safe Kotlin clients from your service models. In this post, you will learn what Smithy Kotlin client generation is, how it works, and how you can use it.</p> 
<p>Modern service development increasingly relies on strong contracts, automation, and consistency. <a href="https://smithy.io/" rel="noopener noreferrer" target="_blank">Smithy</a>&nbsp;provides a model-driven approach to defining services and enables code generation from those definitions, helping you to produce reliable clients from a single source of truth.</p> 
<h2>How it works</h2> 
<p>At a high level, Smithy Kotlin client code generation transforms Smithy service models into strongly typed Kotlin clients. This process bridges the gap between API design and implementation, producing code that handles serialization, protocol details, and request/response lifecycles automatically.</p> 
<h3>Model-driven development</h3> 
<p>At the core of the workflow is modeling services using Smithy. You can define services, operations, and data shapes in a declarative format that captures structure, constraints, and protocol bindings. These models specify the canonical definition of the API surface. For example:</p> 
<div class="hide-language"> 
 <pre><code class="lang-php">namespace com.example

use aws.api#service
use smithy.protocols#rpcv2Cbor

@title("Coffee Shop Service")
@rpcv2Cbor
@service(sdkId: "CoffeeShop")
service CoffeeShop {
&nbsp;&nbsp; &nbsp;version: "2024-08-23"
&nbsp;&nbsp; &nbsp;operations: [
&nbsp;&nbsp; &nbsp; &nbsp; &nbsp;GetMenu
&nbsp;&nbsp; &nbsp;]
}

@http(method: "GET", uri: "/menu")
@readonly
operation GetMenu {
&nbsp;&nbsp; &nbsp;output := {
&nbsp;&nbsp; &nbsp; &nbsp; &nbsp;items: CoffeeItems
&nbsp;&nbsp; &nbsp;}
}
</code></pre> 
</div> 
<p>Smithy Kotlin consumes the models and produces Kotlin client code. The generated output includes typed operations, serializers, and deserializers, maintaining alignment between the model and client implementation.</p> 
<p>For more information about writing Smithy models, see <a href="https://smithy.io/2.0/quickstart.html" rel="noopener noreferrer" target="_blank">Smithy’s quick start documentation</a>.</p> 
<h3>Clients</h3> 
<p>The generated clients support a range of features typical for service communication, including request/response handling, serialization, protocols, and error mapping. You only need to define them in the model and Smithy Kotlin writes the code for you. Because Smithy Kotlin targets Kotlin and generated clients run on the Java Virtual Machine (JVM), they integrate naturally with existing language tools. You can incorporate them into modern build systems, use concurrency features, and combine them with established libraries and frameworks already used in Kotlin. An example of a generated Kotlin client:</p> 
<div class="hide-language"> 
 <pre><code class="lang-php">CoffeeShopClient {
 &nbsp; &nbsp;endpointProvider = CoffeeShopEndpointProvider {
 &nbsp; &nbsp; &nbsp; &nbsp;endpointUrl = Url.parse("http://localhost:8888")
 &nbsp; &nbsp;}
}.use { client -&gt;
&nbsp; &nbsp; val menu =&nbsp;client.getMenu()
}</code></pre> 
</div> 
<p>For more information about how to start generating Kotlin clients from Smithy models, see the&nbsp;<a href="https://smithy.io/2.0/languages/kotlin/client/generating-clients.html" rel="noopener noreferrer" target="_blank">client generation guide</a>.</p> 
<h2>What does general availability mean?</h2> 
<p>Smithy Kotlin has been in development and available in developer preview for a few years. This milestone reflects production readiness, stability, and broader confidence in adopting the generated clients as part of standard development workflows.</p> 
<h2>Conclusion</h2> 
<p>In this blog post, we covered what Smithy Kotlin client generation is, how it works, and how you can use it. To get started with Smithy Kotlin client code generation see the&nbsp;<a href="https://github.com/smithy-lang/smithy-examples/tree/main/smithy-kotlin-examples/quickstart-kotlin" rel="noopener noreferrer" target="_blank">quick start example</a> and <a href="https://smithy.io/2.0/languages/kotlin/index.html" rel="noopener noreferrer" target="_blank">documentation page</a>. If you’d like to share feedback, ask a question, or discuss, you can reach us<a href="https://github.com/smithy-lang/smithy-kotlin/issues" rel="noopener noreferrer" target="_blank">&nbsp;through GitHub issues</a>&nbsp;and <a href="https://github.com/smithy-lang/smithy-kotlin/discussions" rel="noopener noreferrer" target="_blank">GitHub discussions</a>.</p> 
<hr /> 
<h2>About the author</h2>
