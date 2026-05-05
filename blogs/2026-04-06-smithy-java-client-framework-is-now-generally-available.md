---
title: "Smithy Java client framework is now generally available"
url: "https://aws.amazon.com/blogs/developer/smithy-java-client-framework-is-now-generally-available/"
date: "Mon, 06 Apr 2026 17:41:04 +0000"
author: "Manuel Sugawara"
feed_url: "https://aws.amazon.com/blogs/developer/tag/smithy/feed/"
---
<p><a href="https://github.com/smithy-lang/smithy-java" rel="noopener" target="_blank">Smithy Java</a> client code generation is now generally available. You can use it to build type-safe, protocol-agnostic Java clients directly from Smithy models. With Smithy Java, serialization, protocol handling, and request/response lifecycles are all generated automatically from your model. This removes the need to write or maintain any of this code by hand.</p> 
<p>In this post, you will learn what Smithy Java client generation is, how it works, what makes it different, and how you can use it. Modern service development is built on strong contracts and automation. <a href="https://smithy.io/" rel="noopener" target="_blank">Smithy</a> provides a model-driven approach to defining services and generating code from those definitions. It produces clients, services, and documentation from a single source of truth that stays aligned with your API as it evolves. Smithy Java client code generation enforces protocol correctness and removes serialization boilerplate, so you can focus on building features instead of hand-writing requests and responses.</p> 
<h2>How it works</h2> 
<p>At a high level, Smithy Java client code generation transforms Smithy models into strongly typed Java clients.</p> 
<h3>Model-driven development</h3> 
<p>At the core of the workflow is modeling services using Smithy. You define services, operations, and data shapes in a declarative format that captures API structure, constraints, and protocol bindings. These models act as the canonical definition of the API surface. For example:</p> 
<pre><code class="lang-smithy">namespace com.example

use aws.api#service
use smithy.protocols#rpcv2Cbor

@title("Coffee Shop Service")
@rpcv2Cbor
@service(sdkId: "CoffeeShop")
service CoffeeShop {
&nbsp;&nbsp;&nbsp; version: "2024-08-23"
&nbsp;&nbsp;&nbsp; operations: [
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; GetMenu
&nbsp;&nbsp;&nbsp; ]
}

@readonly
operation GetMenu {
&nbsp;&nbsp;&nbsp; output := {
        items: CoffeeItems
    }
} 
...
</code></pre> 
<p>Smithy Java consumes the models and produces Java client code. The generated output includes typed operations, serializers, deserializers, and protocol handling.</p> 
<p>For more information about writing Smithy models, see <a href="https://smithy.io/2.0/quickstart.html" rel="noopener" target="_blank">Smithy’s quick start documentation</a>.</p> 
<h3>Generated clients</h3> 
<p>The generated clients support a range of features that are typical for client-service communication, including request/response handling, serialization, protocol negotiation, retries, error mapping, and custom interceptors. You only need to define them in the model, and Smithy Java writes the code for you.</p> 
<p>The following is an example of a generated Java client:</p> 
<pre><code class="lang-java">var client = CoffeeShopClient.builder()
&nbsp;&nbsp;&nbsp; .endpointProvider(EndpointResolver.staticEndpoint("http://localhost:8888"))
&nbsp;&nbsp;&nbsp; .build();

var menu = client.getMenu();</code></pre> 
<p>You can regenerate clients after API changes to the model, keeping them up to date without writing any manual code.</p> 
<p>For more information about how to start generating Java clients from Smithy models, see our <a href="https://smithy.io/2.0/languages/java/quickstart.html" rel="noopener" target="_blank">quick start guide</a>.</p> 
<h2>Key capabilities</h2> 
<h3>Protocol flexibility</h3> 
<p>Smithy Java generated clients are protocol-agnostic. The framework includes built-in support for HTTP transport, AWS protocols (including <a href="https://smithy.io/2.0/aws/protocols/aws-json-1_0-protocol.html" rel="noopener" target="_blank">AWS JSON 1.0</a>/<a href="https://smithy.io/2.0/aws/protocols/aws-json-1_1-protocol.html" rel="noopener" target="_blank">1.1</a>, <a href="https://smithy.io/2.0/aws/protocols/aws-restjson1-protocol.html" rel="noopener" target="_blank">restJson1</a>, <a href="https://smithy.io/2.0/aws/protocols/aws-restxml-protocol.html" rel="noopener" target="_blank">restXml</a> and <a href="https://smithy.io/2.0/aws/protocols/aws-query-protocol.html" rel="noopener" target="_blank">Query</a>), and <a href="https://smithy.io/2.0/additional-specs/protocols/smithy-rpc-v2.html" rel="noopener" target="_blank">Smithy RPCv2 CBOR</a>. You can swap protocols at runtime without rebuilding the client, enabling gradual protocol migrations and multi-protocol support with no code changes.</p> 
<h3>Dynamic client</h3> 
<p>Not every use case requires code generation at build time. Smithy Java includes a dynamic client that loads Smithy models at runtime and can interact with any service API without a codegen step. This is particularly useful for building tools, service aggregators, or systems that must interact with unknown services at build time, all while keeping the deployment footprint small.</p> 
<p>The following is an example of calling the Coffee Shop service using the DynamicClient :</p> 
<pre><code class="lang-java">var model = Model.assembler().addImport("model.smithy").assemble().unwrap();
var serviceId = ShapeId.from("com.example#CoffeeShop");
var client = DynamicClient.builder().model(model).serviceId(serviceId).build();
var result = client.call("GetMenu");</code></pre> 
<h3>Shape code generation independent of services</h3> 
<p>Smithy Java can generate type-safe Java classes from Smithy shapes without any service context. This extends Smithy’s model-first approach beyond service calls into the data and logic layers of your system, enabling code reuse and consistency across projects that share common types.</p> 
<h3>Built on Java virtual threads</h3> 
<p>Smithy Java is built from the ground up around <a href="https://docs.oracle.com/en/java/javase/21/core/virtual-threads.html" rel="noopener" target="_blank">Java 21’s virtual threads</a>. Instead of exposing complex async APIs with callbacks or reactive streams, it provides a blocking-style interface that is straightforward to read, write, and debug, without sacrificing performance. Users can concentrate on their business logic while letting Smithy Java and the JVM handle task scheduling, synchronization, and structured error handling.</p> 
<p>The following example demonstrates using <a href="https://aws.amazon.com/transcribe/" rel="noopener" target="_blank">Amazon Transcribe</a> with Smithy’s Java event streams blocking API. To send an event, Smithy clients use a <code>EventStreamWriter&lt;T&gt;</code> with a <code>write(T event)</code> method, and to receive an event the client uses <code>EventStreamReader</code> with a <code>T read()</code> method. For example:</p> 
<pre><code class="lang-java">// Create an Amazon Transcribe client
var client = TranscribeClient.builder().build();
var audioStream = EventStream.&lt;AudioStream&gt;newWriter();

// Create a stream transcription request
var request = StartStreamTranscriptionInput.builder().audioStream(audioStream).build();

// Create a VT to send the audio that we want to transcribe
Thread.startVirtualThread(() -&gt; {
&nbsp;&nbsp;&nbsp; try (var audioStreamWriter = audioStream.asWriter()) {
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; for (var chunk : iterableAudioChunks()) {
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; var event = AudioEvent.builder().audioChunk(chunk).build()
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; audioStreamWriter.write(AudioStream.builder().audioEvent(event).build());
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; }
&nbsp;&nbsp;&nbsp; }
});

// Send the request to Amazon Transcribe
var response = client.startStreamTranscription(request);

// Create a VT to read the transcription from the audio.
Thread.startVirtualThread(() -&gt; {
&nbsp;&nbsp;&nbsp; // The reader
&nbsp;&nbsp;&nbsp; try (var results = response.getTranscriptResultStream().asReader()) {
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; // The reader implements Iterable
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; for (var event : results) {
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; switch (event) {
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; case TranscriptResultStream.TranscriptEventMember transcript -&gt; {
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; var transcriptText = getTranscript(transcript);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; if (transcriptText != null) {
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; appendAudioTranscript(transcriptText);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; }
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; }
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; default -&gt; throw new IllegalStateException("Unexpected event " + event);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; }
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; }
&nbsp;&nbsp;&nbsp; }
});</code></pre> 
<h2>Conclusion</h2> 
<p>In this post, I explained what Smithy Java client generation is and how it works. With this general availability release, Smithy Java’s public APIs are now stable; we commit to backwards compatibility, making it ready for use in production systems. To get started with Smithy Java client code generation, use our <a href="https://smithy.io/2.0/languages/java/quickstart.html" rel="noopener" target="_blank">quick start guide</a> and <a href="https://smithy.io/2.0/languages/java/index.html" rel="noopener" target="_blank">documentation</a>. If you want to send us feedback, ask a question, or discuss, you can reach us through <a href="https://github.com/smithy-lang/smithy-java/issues" rel="noopener" target="_blank">GitHub issues</a> and <a href="https://github.com/smithy-lang/smithy-java/discussions" rel="noopener" target="_blank">GitHub discussions</a>.</p> 
<hr /> 
<h2>About the author</h2>
