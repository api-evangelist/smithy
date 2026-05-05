---
title: "Developer Preview: Ruby SDK code generation using Smithy"
url: "https://aws.amazon.com/blogs/developer/developer-preview-smithy-code-generated-ruby-sdk/"
date: "Wed, 12 Jan 2022 20:52:14 +0000"
author: "Matt Muller"
feed_url: "https://aws.amazon.com/blogs/developer/tag/smithy/feed/"
---
<h2>What is this?</h2> 
<p>The AWS SDK For Ruby team is happy to announce the developer preview of <a href="https://github.com/awslabs/smithy-ruby">smithy-ruby</a>, a toolchain that can be used to code generate a “<a href="https://en.wikipedia.org/wiki/White-label_product">white label</a>” Ruby SDK for your service API using Smithy modeling. An upcoming future version of the AWS SDK For Ruby will use Smithy code generation.</p> 
<h2>What is Smithy?</h2> 
<p>Smithy is an interface definition language and set of tools that allows developers to build clients and servers in multiple languages. Smithy models define a service as a collection of resources, operations, and shapes. A Smithy model enables API providers to generate clients and servers in various programming languages, API documentation, test automation, and example code. For more information about Smithy, see the <a href="https://awslabs.github.io/smithy/index.html">Smithy documentation</a>.</p> 
<h2>What’s included in the Ruby SDK</h2> 
<p style="text-align: center;"><a href="https://d2908q01vomqb2.cloudfront.net/0716d9708d321ffb6a00818614779e779925365c/2022/01/07/Screen-Shot-2021-11-30-at-12.34.34-PM.png"><img alt="" class="alignnone size-medium wp-image-9045" height="300" src="https://d2908q01vomqb2.cloudfront.net/0716d9708d321ffb6a00818614779e779925365c/2022/01/07/Screen-Shot-2021-11-30-at-12.34.34-PM-211x300.png" width="211" /></a></p> 
<p style="text-align: center;"><strong>Components of a code generated Ruby SDK</strong></p> 
<p>A code generated Ruby SDK using Smithy will have generic components and protocol specific components. These components are (in no particular order):</p> 
<ul> 
 <li><strong>Validators (private)</strong> – A set of classes that validate Ruby input types against the Smithy model.</li> 
 <li><strong>Builders (private, protocol)</strong> – A set of classes that build a protocol specific request using input (i.e. JSON over HTTP).</li> 
 <li><strong>Stubs (private, protocol)</strong> – A set of classes that build a protocol specific response using stub data, used for testing.</li> 
 <li><strong>Parsers (private, protocol)</strong> – A set of classes that parse a protocol specific response into data structures (i.e. XML over HTTP).</li> 
 <li><strong>Types (public)</strong> – A set of classes that represent structure shapes (Plain Old Ruby Objects).</li> 
 <li><strong>Errors (public, protocol)</strong> – A set of classes that represent error shapes and protocol specific error classes.</li> 
 <li><strong>Params (private)</strong> – A set of modules that convert hash-y input to rigid input types used by the Client operations.</li> 
 <li><strong>Paginators (public)</strong> – A set of classes used for traversing paginated operations automatically.</li> 
 <li><strong>Waiters (public)</strong> – A set of classes used to wait until an operation reaches a desired state before resuming control back to the client.</li> 
 <li><strong>Client (public)</strong> – A class that ties everything together; it is the public interface to the service API. The client is responsible for constructing requests and returning responses using middleware.</li> 
</ul> 
<p>For more information about the components, please see the <a href="https://github.com/awslabs/smithy-ruby/wiki">smithy-ruby wiki</a>.</p> 
<h2>Middleware</h2> 
<p>Middleware are classes that sit between the client and the server, providing a way to modify the request-response cycle. At minimum, middleware is used to build a request, send a request, and parse a response. Middleware is organized in a stack and are responsible for calling the next middleware.</p> 
<p style="text-align: center;"><a href="https://d2908q01vomqb2.cloudfront.net/0716d9708d321ffb6a00818614779e779925365c/2022/01/07/Screen-Shot-2021-05-17-at-1.26.28-PM.png"><img alt="" class="alignnone size-medium wp-image-9046" height="300" src="https://d2908q01vomqb2.cloudfront.net/0716d9708d321ffb6a00818614779e779925365c/2022/01/07/Screen-Shot-2021-05-17-at-1.26.28-PM-167x300.png" width="167" /></a></p> 
<p style="text-align: center;"><strong>Middleware stack</strong></p> 
<p>In the client, each API operation will have a method that is responsible for creating its own middleware stack and handling the request and response cycle. Seahorse will ship with 6 default middleware. Each middleware will have access to the request, response, and context.</p> 
<p>In detail, the middleware components are:</p> 
<ul> 
 <li><strong>Validate</strong> – Validates input using the Validator classes if configured to do so. (Optional – client configuration)</li> 
 <li><strong>Build</strong> – Builds a protocol specific request (i.e. JSON over HTTP) using the Builder classes and input.</li> 
 <li><strong>HostPrefix</strong> – Modifies the endpoint with a host prefix if configured to do so. (Optional – Smithy trait)</li> 
 <li><strong>Send</strong> – Sends the request using a protocol specific client (i.e. HTTP client). The middleware may return responses using the Stubs classes if configured to do so.</li> 
 <li><strong>Parse</strong> – Parses a protocol specific response (i.e. XML over HTTP) using the Parser classes and the raw service response.</li> 
 <li><strong>Retry</strong> – Retries a request for networking errors and any responses with retry-able or throttling errors.</li> 
</ul> 
<p>Protocol implementations may also insert their own code generated middleware. Middleware may also be added at runtime to a Client class or Client instance, and to individual operation calls.</p> 
<h2>Rails JSON Protocol</h2> 
<p>A Smithy built Ruby SDK needs a protocol implementation to fully function, much like a car needs an engine. As part of this developer preview, we will be including a protocol implementation that we are calling “<a href="https://github.com/awslabs/smithy-ruby/wiki/Rails-JSON-Protocol">Rails JSON</a>”. With the Rails JSON protocol definition, a Smithy model can be used to code generate a Ruby SDK that communicates directly with a Rails API over JSON. Neat!</p> 
<p>As a demo, the following sections will detail how to setup a Rails service and generate an SDK that can communicate with it.</p> 
<h3>Setup Rails API Service</h3> 
<p>Before we can create an SDK, we need a service for it to communicate to. Let’s first create a new Rails API service with: <code>rails new --api sample-service</code>.</p> 
<p>Next, echoing <a href="https://guides.rubyonrails.org/command_line.html">rails documentation</a>, let’s create a High Score model with <code>rails generate scaffold HighScore game:string score:integer</code> and run <code>rake db:migrate</code>.</p> 
<p>In <code>models/high_score.rb</code>, add a length validation to the game’s name by adding: <code>validates :game, length: { minimum: 2 }</code>. This validation will be used later.</p> 
<p>Now it’s time to start our rails app with <code>rails s</code> and verify it’s running on an endpoint such as <code>http://127.0.0.1:3000</code>; we will need this endpoint for later.</p> 
<p>If you aren’t able to generate a Rails app, don’t worry, a <a href="https://github.com/awslabs/smithy-ruby/tree/4cba34e43867a84272be12812d66cc032f92a966/sample-service">copy of this sample Rails service</a> lives in smithy-ruby for now.</p> 
<h3>Add the Smithy model</h3> 
<p>To generate the SDK, we need the Smithy model that describes the Rails service we just defined. I’ve conveniently defined this in smithy-ruby in <code><a href="https://github.com/awslabs/smithy-ruby/blob/4cba34e43867a84272be12812d66cc032f92a966/codegen/smithy-ruby-rails-codegen-test/model/high-score-service.smithy">high-score-service.smithy</a></code>. The model tells smithy-ruby to code generate shapes and a client API for the High Score service and to use Rails’ JSON protocol.</p> 
<p>Let’s break down some of the important parts.</p> 
<p>The first section tells Smithy to create the HighScoreService using the Rails JSON protocol and define its resources and operations. The resource has an identifier (Rails defaults to id), which is used to look up the High Score. The resource has all of the basic Rails CRUD operations: get, create, update, delete, and list</p> 
<pre><code class="lang-ruby">@railsJson
@title("High Score Sample Rails Service")
service HighScoreService {
    version: "2021-02-15",
    resources: [HighScore],
}

/// Rails default scaffold operations
resource HighScore {
    identifiers: { id: String },
    read: GetHighScore,
    create: CreateHighScore,
    update: UpdateHighScore,
    delete: DeleteHighScore,
    list: ListHighScores
}</code></pre> 
<p>The next sections define the service shapes. HighScoreAttributes is a shape that returns all of the properties of a High Score. HighScoreParams includes all of the properties that a High Score will need. The @length validation of &gt;2 characters is applied to game.</p> 
<pre><code class="lang-ruby">/// Modeled attributes for a High Score
structure HighScoreAttributes {
    /// The high score id
    id: String,
    /// The game for the high score
    game: String,
    /// The high score for the game
    score: Integer,
    // The time the high score was created at
    createdAt: Timestamp,
    // The time the high score was updated at
    updatedAt: Timestamp
}

/// Permitted params for a High Score
structure HighScoreParams {
    /// The game for the high score
    @length(min: 2)
    game: String,
    /// The high score for the game
    score: Integer
}</code></pre> 
<p>Next are the operation shapes. The @http trait is applied to each operation with the expected Rails path.</p> 
<pre><code class="lang-ruby">/// Get a high score
@http(method: "GET", uri: "/high_scores/{id}")
@readonly
operation GetHighScore {
    input: GetHighScoreInput,
    output: GetHighScoreOutput
}

/// Input structure for GetHighScore
structure GetHighScoreInput {
    /// The high score id
    @required
    @httpLabel
    id: String
}

/// Output structure for GetHighScore
structure GetHighScoreOutput {
    /// The high score attributes
    @httpPayload
    highScore: HighScoreAttributes
}

/// Create a new high score
@http(method: "POST", uri: "/high_scores", code: 201)
operation CreateHighScore {
    input: CreateHighScoreInput,
    output: CreateHighScoreOutput,
    errors: [UnprocessableEntityError]
}

/// Input structure for CreateHighScore
structure CreateHighScoreInput {
    /// The high score params
    @required
    highScore: HighScoreParams
}

/// Output structure for CreateHighScore
structure CreateHighScoreOutput {
    /// The high score attributes
    @httpPayload
    highScore: HighScoreAttributes,

    /// The location of the high score
    @httpHeader("Location")
    location: String
}

/// Update a high score
@http(method: "PUT", uri: "/high_scores/{id}")
@idempotent
operation UpdateHighScore {
    input: UpdateHighScoreInput,
    output: UpdateHighScoreOutput,
    errors: [UnprocessableEntityError]
}

/// Input structure for UpdateHighScore
structure UpdateHighScoreInput {
    /// The high score id
    @required
    @httpLabel
    id: String,

    /// The high score params
    highScore: HighScoreParams
}

/// Output structure for UpdateHighScore
structure UpdateHighScoreOutput {
    /// The high score attributes
    @httpPayload
    highScore: HighScoreAttributes
}

/// Delete a high score
@http(method: "DELETE", uri: "/high_scores/{id}")
@idempotent
operation DeleteHighScore {
    input: DeleteHighScoreInput,
    output: DeleteHighScoreOutput
}

/// Input structure for DeleteHighScore
structure DeleteHighScoreInput {
    /// The high score id
    @required
    @httpLabel
    id: String
}

/// Output structure for DeleteHighScore
structure DeleteHighScoreOutput {}

/// List all high scores
@http(method: "GET", uri: "/high_scores")
@readonly
operation ListHighScores {
    output: ListHighScoresOutput
}

/// Output structure for ListHighScores
structure ListHighScoresOutput {
    /// A list of high scores
    @httpPayload
    highScores: HighScores
}

list HighScores {
    member: HighScoreAttributes
}</code></pre> 
<h3>Generate the SDK</h3> 
<p>With the model and a Rails service, it’s now time to generate the SDK. Smithy code generation and integration is only available in Java environments. Fortunately, for this demo, the High Score Service SDK has already been generated and committed to the smithy-ruby repo. Download it <a href="https://github.com/awslabs/smithy-ruby/tree/4cba34e43867a84272be12812d66cc032f92a966/codegen/projections/high_score_service">from here</a> if you are following along!</p> 
<p>If you’d like to generate it yourself, or generate your own Smithy model, you can follow the <a href="https://github.com/awslabs/smithy-ruby/blob/main/README.md#generating-an-sdk-for-a-rails-json-api">README instructions</a> that detail how to use smithy-ruby in your Gradle project.</p> 
<h3>Use the SDK</h3> 
<p>Now we have a Rails service and an SDK. Start up <code>irb</code> with <code>irb -I high_score_service/lib</code> and try it out!</p> 
<pre><code class="lang-ruby">require 'high_score_service'

# Create an instance of HighScoreService's Client.
# This is similar to the AWS SDK Clients.
# Here we use the endpoint of the Rails service.
client = HighScoreService::Client.new(endpoint: 'http://127.0.0.1:3000')

# List all high scores
client.list_high_scores
# =&gt; #&lt;struct HighScoreService::Types::ListHighScoresOutput high_scores=[]&gt;

# Try to create a high score
# Should raise an UnprocessableEntityError, let's find out why
 begin
  client.create_high_score(high_score: { score: 9001, game: 'X' })
rescue =&gt; e
  puts e.data
  # =&gt; #&lt;struct HighScoreService::Types::UnprocessableEntityError errors={"game"=&gt;["is too short (minimum is 2 characters)"]}&gt;
end

# Actually create a high score
client.create_high_score(high_score: { score: 9001, game: 'Frogger' })
# =&gt; #&lt;struct HighScoreService::Types::CreateHighScoreOutput

# List high scores again
resp = client.get_high_score(id: '1')
resp.high_score
# =&gt; #&lt;struct HighScoreService::Types::HighScoreAttributes id=1, game="Frogger", score=9001 ... &gt;</code></pre> 
<p>As an exercise, try out the <code>delete_high_score</code> and <code>update_high_score</code> operations.</p> 
<h2>Future Plans</h2> 
<p>Looking forward, smithy-ruby will be used to generate the new service client versions (gem version 2, core version 4) of AWS SDK For Ruby.</p> 
<p>We’d like to explore more Smithy Ruby and Rails API use cases. Perhaps a Smithy model can be parsed and translated into a set of <code>rails new</code> and <code>rails generate</code> commands; going further, perhaps a “server-side SDK” can be a pluggable Rails engine that handles building and parsing of concrete types and protocols.</p> 
<h2>Feedback</h2> 
<p>If you have any questions, comments, concerns, ideas, or other feedback, please create an issue or discussion in the <a href="https://github.com/awslabs/smithy-ruby">smithy-ruby repository</a>. We welcome any SDK design feedback and improvements, and we especially welcome any community contributions.</p> 
<p>Thanks for reading!</p> 
<p>-Matt</p>
