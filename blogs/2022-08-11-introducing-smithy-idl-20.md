---
title: "Introducing Smithy IDL 2.0"
url: "https://aws.amazon.com/blogs/developer/introducing-smithy-idl-2-0/"
date: "Thu, 11 Aug 2022 16:33:12 +0000"
author: "Kevin Stich"
feed_url: "https://aws.amazon.com/blogs/developer/tag/smithy/feed/"
---
<p>The AWS Smithy team is happy to announce the 2.0 release of the <a href="https://awslabs.github.io/smithy/" rel="noopener noreferrer" target="_blank">Smithy Interface Definition Language (IDL)</a>. This release focuses on improving the developer experience of authoring Smithy models and using code generated from Smithy models. It contains numerous new features, such as reduced nulls and optional types in generated code, custom default values, mixins to reduce model duplication, resource properties to improve consistency across operations, dedicated enumeration shapes, and syntax improvements.</p> 
<p>Smithy is a protocol-agnostic IDL and set of tools for generating clients, servers, documentation, and other artifacts. It’s purpose-built for code generation, can be extended with custom traits, and enables automatic API standards enforcement. Smithy is Amazon’s next-generation API modeling language, based on our experience building tens of thousands of services and generating SDKs.</p> 
<h2>Fewer nulls in generated code</h2> 
<p>One goal of Smithy IDL 2.0 is to reduce the amount of nullable properties in generated code. With the addition of Rust, Kotlin, and Swift as supported programming languages in AWS SDKs, it became apparent that we needed a better approach for defining nullability in Smithy models to make generated code in these languages more idiomatic. However, we also had to balance this desire against the business requirement of allowing services to safely evolve their models over long periods of time (sometimes decades).</p> 
<p>Prior to Smithy 2.0, Smithy code generators essentially treated every structure member as nullable. This was because: 1) nullability was somewhat confusing and people modeling structures didn’t realize that default values were provided by only some of the shapes targeted by members, and 2) the <code>@required</code> trait could be backward compatibly removed at any time because it was considered the removal of a constraint.</p> 
<p>Consider the following Smithy IDL 1.0 example:</p> 
<div class="hide-language"> 
 <pre class="unlimited-height-code"><code class="lang-kotlin">$version: "1.0"
namespace smithy.example

structure Message {
    // Non-null because it has a default value of false.
    delivered: MyBoolean,

    // Nullable because the required trait can be removed.
    @required
    message: String,
}

// Not marked with the box trait, so it provides a default
// value to members that target it.
boolean MyBoolean</code></pre> 
</div> 
<p>In the above example:</p> 
<ol> 
 <li>The <code>delivered</code> member of <code>Message</code> has a default value of <code>false</code> because <code>MyBoolean</code> isn’t marked with the <code>@box</code> trait. These semantics were inherited from Smithy’s predecessor within Amazon, and many teams get confused by how these semantics work because it’s a confusing kind of “action at a distance” from the member definition.</li> 
 <li>The <code>message</code> member is nullable because the <code>@required</code> trait can be removed at any time. This allowed teams using Smithy 1.0 to remove the <code>@required</code> trait from a member if they ever needed to due to changing business requirements. As such, using the <code>@required</code> trait to influence code-generated types was expressly prohibited by the specification.</li> 
</ol> 
<p>Smithy IDL 2.0 introduces custom default values for structure members. The previous example is written in Smithy IDL 2.0 as:</p> 
<div class="hide-language"> 
 <pre class="unlimited-height-code"><code class="lang-kotlin">$version: "2"
namespace smithy.example

structure Message {
    delivered: MyBoolean = false

    @required
    message: String
}

boolean MyBoolean</code></pre> 
</div> 
<p>With the introduction of default values in Smithy IDL 2.0, we realized that we can still provide almost the same level of model evolution flexibility while still reducing nullability in generated code. In Smithy IDL 2.0, if the <code>@required</code> trait is ever removed from a member, the member has to be provided a default value. This new affordance means that code generators can safely use the <code>@required</code> trait and default values to generate types that contain non-nullable properties. For example, if a structure member is <code>@required</code>, then accessing the member will always return a value. The same is true for members that have default values: they will always return a value even if one wasn’t explicitly provided by the end-user.</p> 
<p>Without these updates, languages like Rust that explicitly model nullability in their type systems would generate optional accessors. This would cause users to have to unsafely dereference most structure members.</p> 
<p>The nullability improvements in Smithy IDL 2.0 simplify how nullability is modeled, still provides developers the ability to evolve models over time if they need to remove the <code>@required</code> trait, and explicitly model default values of structure members rather than only relying on documentation.</p> 
<p><em>Please note that implementing these Smithy IDL 2.0 nullability semantics in Smithy code generators and AWS SDKs is still a work in progress, but they will be incorporated before they made are generally available.</em></p> 
<p>You can read more about member nullability in the <a href="https://awslabs.github.io/smithy/2.0/spec/aggregate-types.html#structure-member-optionality" rel="noopener noreferrer" target="_blank">Smithy documentation about structure member optionality.</a></p> 
<h2>Mixins to reduce model duplication</h2> 
<p>When authoring a Smithy model, the same members often need to be re-defined in related structures. Manually copy-pasting parts of shapes in a model is not only tedious; it’s also error prone and can lead to unintentional drift between shapes over time as different parts of a service are updated. Mixins are a kind of model-time copy-paste that eliminates this kind of model duplication. You can learn more about mixins in the <a href="https://awslabs.github.io/smithy/2.0/spec/mixins.html" rel="noopener noreferrer" target="_blank">Smithy documentation for mixins.</a></p> 
<p>Defining resources in Smithy 1.0 often requires a significant amount of duplication across inputs and outputs. The following example describes a single operation in Smithy 1.0:</p> 
<div class="hide-language"> 
 <pre class="unlimited-height-code"><code class="lang-kotlin">$version: "1.0"
namespace smithy.example

resource City {
    identifiers: { cityId: String },
    create: CreateCity
}

operation CreateCity {
    input: CreateCityInput,
    output: CreateCityOutput
}

@input
structure CreateCityInput {
    name: String,
    population: Integer,
    foundedOn: Timestamp,
}

@output
structure CreateCityOutput {
    cityId: String,
    name: String,
    population: Integer,
    foundedOn: Timestamp,
}</code></pre> 
</div> 
<p>Utilizing mixins for this API reduces the repetition and removes surface area for inconsistencies in the model (especially as more operations are added). Here is an updated version of the above model, using resource properties (explained later in the blog post) and mixins:</p> 
<div class="hide-language"> 
 <pre class="unlimited-height-code"><code class="lang-kotlin">$version: "2.0"
namespace smithy.example

resource City {
    identifiers: { cityId: String }
    properties: {
        name: String
        population: Integer
        foundedOn: Timestamp
    }
    create: CreateCity
}

operation CreateCity {
    input := with [CityData] {}
    output := with [CityRecord] {}
}

@mixin
structure CityData for City {
    $name
    $population
    $foundedOn
}

@mixin
structure CityRecord with [CityIdentifiers, CityData] {}

@mixin
structure CityIdentifiers for City {
    @required
    $cityId
}</code></pre> 
</div> 
<p>Mixins can be used with other shape types as well, like enumerations of various platform options, operations with a consistent set of errors, and services that share common operations like this example:</p> 
<div class="hide-language"> 
 <pre class="unlimited-height-code"><code class="lang-kotlin">service WeatherService with [TaggableService] {
    // This service inherits the operations of TaggableService.
}

@mixin
service TaggableService {
    operations: [
        TagResource
        UntagResource
        ListTagsForResource
    ]
}</code></pre> 
</div> 
<h2>Resource properties</h2> 
<p>Smithy models can define resources to communicate the “things” in an API. Resources in Smithy 1.0 only included operations and identifiers. This allowed services to define the resources in their APIs and ensure that operations are provided the right identifiers, but the lack of validation to ensure that resources expose properties consistently across operations can lead to usability issues. For example, it was easy to define a service with a <code>PutFoo</code> operation that accepts an <code>idList</code> member in its input while a <code>GetFoo</code> operation returns a corresponding member as <code>ids</code>. This kind of inconsistency is undesirable because it puts a usability burden on end users to determine that these members mean the same thing.</p> 
<p>Smithy 2.0 adds the ability to define the <code>properties</code> of a resource. With this new declaration, Smithy will automatically detect when a team accidentally uses the wrong property name in the input or output of a resource based operation.</p> 
<div class="hide-language"> 
 <pre class="unlimited-height-code"><code class="lang-kotlin">$version: "2"
namespace smithy.example

resource City {
    identifiers: { cityId: CityId }
    properties: {
        name: String
        population: Integer
        foundedOn: Timestamp
    }
    create: CreateCity
}</code></pre> 
</div> 
<p>Each property defines a name and the shape it targets. The input and output of resource operations are limited to members that match these property names and targets (with a few caveats explained in the specification). The new <code>@notProperty</code> trait can be used to indicate that a member of an instance operation isn’t a resource property, like a <code>dryRun</code> boolean or an idempotency token.</p> 
<p>Smithy 2.0 also introduces a new feature called <em>member elision</em> to refer to resource identifiers, resource properties, or mixin members. To use member elision, add a <code>$</code> character before a member name and omit the target of the member so it is inherited. This syntax eliminates the need to redefine the shape targeted by the referenced component, and makes it clear that the target is inherited.</p> 
<div class="hide-language"> 
 <pre class="unlimited-height-code"><code class="lang-kotlin">operation CreateCity {
    input := for City {
        @required
        $name

        @required
        $population

        @required
        $location
    }
}</code></pre> 
</div> 
<p><em>Note: In order to refer to resource properties or identifiers with member elision syntax, bind the resource to a structure using the following <code>for</code> syntax.</em></p> 
<p>You can learn more about these new features in the Smithy documentation for <a href="https://awslabs.github.io/smithy/2.0/spec/service-types.html#resource-properties" rel="noopener noreferrer" target="_blank">resource properties</a> and <a href="https://awslabs.github.io/smithy/2.0/spec/idl.html#idl-target-elision" rel="noopener noreferrer" target="_blank">target elision</a>.</p> 
<h2>Dedicated enumeration shapes</h2> 
<p>Smithy 1.0 previously defined enumerations using an <code>@enum</code> trait on a string shape. This resulted in several issues: the syntax used to define enums was verbose, enum values weren’t members so removing enum values from different projections (or views) or a Smithy model required special-casing, and the metadata associated with an enum value duplicated traits like <code>@documentation</code> and <code>@tags</code>.</p> 
<p>Smithy 2.0 introduces the <code>enum</code> and <code>intEnum</code> shapes and deprecates the <code>@enum</code> trait. <code>enum</code> and <code>intEnum</code> are used to represent a fixed set of string and integer values. You can learn more in the Smithy documentation for <a href="https://awslabs.github.io/smithy/2.0/spec/simple-types.html#enum" rel="noopener noreferrer" target="_blank">enum shapes</a> and <a href="https://awslabs.github.io/smithy/2.0/spec/simple-types.html#intenum" rel="noopener noreferrer" target="_blank">intEnum shapes</a>.</p> 
<p>Making enums dedicated shape types improves the modeling experience and makes their use with other parts of Smithy more consistent. Enum values are now members of enum shapes, meaning documentation, tags, and other properties now use the standard Smithy traits instead of additional properties of enum value definitions. Since enums are now shapes with their values as members, they’re affected by model transformations without needing to write specialized code.</p> 
<p>This example shows how enums were defined in Smithy 1.0:</p> 
<div class="hide-language"> 
 <pre class="unlimited-height-code"><code class="lang-kotlin">$version: "1.0"
namespace smithy.example

@enum([
    {
        value: "t2.nano",
        name: "T2_NANO",
        documentation: """
            T2 instances are Burstable Performance
            Instances that provide a baseline level of CPU
            performance with the ability to burst above the
            baseline.""",
        tags: ["ebsOnly"]
    },
    {
        value: "t2.micro",
        name: "T2_MICRO",
        documentation: """
            T2 instances are Burstable Performance
            Instances that provide a baseline level of CPU
            performance with the ability to burst above the
            baseline.""",
        tags: ["ebsOnly"]
    },
    {
        value: "m256.mega",
        name: "M256_MEGA",
        deprecated: true
    }
])
string MyString</code></pre> 
</div> 
<p>This changes significantly when using an <code>enum</code> shape in Smithy 2.0, utilizing traits and documentation comments instead.</p> 
<div class="hide-language"> 
 <pre class="unlimited-height-code"><code class="lang-kotlin">$version: "2"
namespace smithy.example

enum MyString {
    /// T2 instances are Burstable Performance Instances that
    /// provide a baseline level of CPU performance with the
    /// ability to burst above the baseline.
    @tags["ebsOnly"]
    T2_NANO = "t2.nano"
    
    /// T2 instances are Burstable Performance Instances that
    /// provide a baseline level of CPU performance with the
    /// ability to burst above the baseline.
    @tags["ebsOnly"]
    T2_MICRO = "t2.micro"
    
    @deprecated
    M256_MEGA = "m256.mega"
}</code></pre> 
</div> 
<p>The <code>intEnum</code> shape type provides the ability to define enumerated integers, rather than just strings. This enables compatibility with various serialization formats that use integers for enums rather than strings.</p> 
<div class="hide-language"> 
 <pre class="unlimited-height-code"><code class="lang-kotlin">$version: "2"
namespace smithy.example

intEnum FaceCard {
    JACK = 1
    QUEEN = 2
    KING = 3
    ACE = 4
    JOKER = 5
}</code></pre> 
 <h2>Smithy IDL syntax improvements</h2> 
 <p>The Smithy IDL was updated to improve the experience of authoring models:</p> 
 <ol> 
  <li>Commas are no longer required and are treated as whitespace. Having to worry about things like trailing commas to reduce diff noise was a distraction from defining a model. We realized that we don’t actually need commas to make models unambiguous, so they’re no longer required.</li> 
  <li>Multiple traits can be applied to a shape in a single block <code>apply</code> statement (e.g., <code>apply Foo { @required, @sensitive }</code>)</li> 
  <li>The input/output of operations can be defined inside of an operation.</li> 
 </ol> 
 <h3>Inline operation input and output</h3> 
 <p>Operation input and output shapes were a source of unnecessary verbosity in Smithy models — they’re always structures, almost never re-used, and usually have boilerplate names. In Smithy IDL 2.0, it is now possible to define input and output structures inline, centralizing the definition of an operation and reducing boilerplate. You can learn more in the <a href="https://awslabs.github.io/smithy/2.0/spec/idl.html#inline-input-output-shapes" rel="noopener noreferrer" target="_blank">Smithy documentation about inline operation inputs and outputs</a>.</p> 
 <p>An operation that has single purpose input and output shapes previously had to define them explicitly and separately.</p> 
 <div class="hide-language"> 
  <pre class="unlimited-height-code"><code class="lang-kotlin">$version: "1.0"
namespace smithy.example

operation CreateCity {
    input: CreateCityInput,
    output: CreateCityOutput,
}

@input
structure CreateCityInput {
    @required
    name: String,

    population: Integer,
    foundedOn: Timestamp,
}

@output
structure CreateCityOutput {
    @required
    cityId: String,

    @required
    name: String,

    population: Integer,
    foundedOn: Timestamp,
}</code></pre> 
 </div> 
 <p>With inline operation input and output, the operation definition is all in one place and doesn’t need to be explicitly named or annotated with an <code>@input</code> or <code>@output</code> trait, as that is done automatically. The following Smithy 2.0 model is equivalent to the previous 1.0 model:</p> 
 <div class="hide-language"> 
  <pre class="unlimited-height-code"><code class="lang-kotlin">$version: "2"
namespace smithy.example

operation CreateCity {
    input := {
        @required
        name: String

        population: Integer
        foundedOn: Timestamp
    }
    output := {
        @required
        cityId: String

        @required 
        name: String

        population: Integer
        foundedOn: Timestamp
    }
}</code></pre> 
 </div> 
 <p>Assuming <code>City</code> is a resource in the model with identifiers and properties, <code>CreateCity</code> can be further simplified using member elision:</p> 
 <div class="hide-language"> 
  <pre class="unlimited-height-code"><code class="lang-kotlin">$version: "2"
namespace smithy.example

operation CreateCity {
    input := for City {
        $name
        $population
        $foundedOn
    }
    output := for City {
        @required
        $cityId

        $name
        $population
        $foundedOn
    }
}</code></pre> 
 </div> 
 <h2>Recap</h2> 
 <p>We have taken a look at the significant new features introduced to Smithy in IDL 2.0 and how they will provide a more productive developer experience: reduced nulls and optional types in generated code, improved resource modeling for consistent properties, first-class enumeration modeling with <code>enum</code> and <code>intEnum</code> shapes, and simplified models using mixins and inline operation input/output.</p> 
</div> 
<p>To see the complete list of changes for this release, check out the <a href="https://github.com/awslabs/smithy/blob/main/CHANGELOG.md#1230-2022-08-10" rel="noopener noreferrer" target="_blank">Smithy repository changelog</a>.</p> 
<h2>Next Steps</h2> 
<ul> 
 <li>Already using Smithy 1.0 and ready to migrate to Smithy 2.0? Get started with the <a href="https://awslabs.github.io/smithy/2.0/guides/migrating-idl-1-to-2.html" rel="noopener noreferrer" target="_blank">IDL 1.0 to 2.0 migration guide</a>.</li> 
 <li>If you are new to Smithy and IDL 2.0 and want to explore more, take a look at the <a href="https://awslabs.github.io/smithy/" rel="noopener noreferrer" target="_blank">Smithy documentation</a> and watch this <a href="https://www.youtube.com/watch?v=3GpZzu4guTE" rel="noopener noreferrer" target="_blank">introductory video</a> from Michael Dowling, one of Smithy’s creators.</li> 
 <li>For feature requests, bug reports, contributions, or feedback of any kind, head over to the <a href="https://github.com/awslabs/smithy" rel="noopener noreferrer" target="_blank">Smithy GitHub repository</a>.</li> 
</ul>
