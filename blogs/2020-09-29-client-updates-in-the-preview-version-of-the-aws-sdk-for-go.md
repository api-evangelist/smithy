---
title: "Client Updates in the Preview Version of the AWS SDK for Go V2"
url: "https://aws.amazon.com/blogs/developer/client-updates-in-the-preview-version-of-the-aws-sdk-for-go-v2/"
date: "Tue, 29 Sep 2020 01:55:21 +0000"
author: "Sean McGrail"
feed_url: "https://aws.amazon.com/blogs/developer/tag/smithy/feed/"
---
<blockquote>
 <p>As of January 19th, 2021, the AWS SDK for Go, version 2 (v2) is generally available.</p>
</blockquote> 
<p>The AWS SDK for Go V2 developer preview was introduced in December 2017 based on customer feedback provided by you, and we focused on improving its ease of use, performance, consistency, and discoverability. We’re happy to share the updated clients for the <a href="https://github.com/aws/aws-sdk-go-v2/releases/tag/v0.25.0">v0.25.0</a> preview version of the AWS SDK for Go V2.</p> 
<p>The updated clients leverage new developments and advancements within AWS and the Go software ecosystem at large since our original preview announcement. Using the new clients will be a bit different than before. The key differences are: simplified API operation invocation, performance improvements, support for error wrapping, and a new middleware architecture. &nbsp;So below we have a guided walkthrough to help try it out and share your feedback in order to better influence the features you’d like to see in the GA version.</p> 
<h2>Performance improvements</h2> 
<p>Our clients were rewritten to take advantage of the <a href="https://awslabs.github.io/smithy">Smithy</a> modeling language. The switch to Smithy provided the opportunity to introduce code generated serialization and deserialization of the AWS protocols. Using the service model, we are able to generate the exact specification for handling an operation’s request and response data structures, bypassing the need for runtime-based reflection. This elimination allows for a marked improvement in CPU and memory utilization when using the API clients.</p> 
<p>Looking at the Amazon DynamoDB client as an example, we can see that there is an overall improvement in CPU and memory utilization of our benchmark program. Here we show an example of a small and large API response for the Scan API operation.</p> 
<table style="border: 2px solid black; border-collapse: collapse; margin-left: auto; margin-right: auto; height: 288px;" width="750"> 
 <tbody> 
  <tr style="border-bottom: 1px solid black; background-color: #e0e0e0;"> 
   <td style="border-right: 1px solid black; padding: 4px; text-align: center;"><strong>Test Case</strong></td> 
   <td style="border-right: 1px solid black; padding: 4px; text-align: center;"><strong>CPU Time</strong></td> 
   <td style="border-right: 1px solid black; padding: 4px; text-align: center;"><strong>&nbsp;Δ%</strong></td> 
   <td style="border-right: 1px solid black; padding: 4px; text-align: center;"><strong>Bytes Allocated</strong></td> 
   <td style="border-right: 1px solid black; padding: 4px; text-align: center;"><strong>&nbsp;Δ%</strong></td> 
   <td style="border-right: 1px solid black; padding: 4px; text-align: center;"><strong>Total Allocations</strong></td> 
   <td style="border-right: 1px solid black; padding: 4px; text-align: center;"><strong>&nbsp;Δ%</strong></td> 
  </tr> 
  <tr style="border-bottom: 1px solid black;"> 
   <td style="border-right: 1px solid black; padding: 4px; text-align: center;"><strong>V1_SMALL</strong></td> 
   <td style="border-right: 1px solid black; padding: 4px; text-align: center;">63391 ns/op</td> 
   <td style="border-right: 1px solid black; padding: 4px; text-align: center;">–</td> 
   <td style="border-right: 1px solid black; padding: 4px; text-align: center;">60172 B/op</td> 
   <td style="border-right: 1px solid black; padding: 4px; text-align: center;">–</td> 
   <td style="border-right: 1px solid black; padding: 4px; text-align: center;">1285 allocs/op</td> 
   <td style="border-right: 1px solid black; padding: 4px; text-align: center;">–</td> 
  </tr> 
  <tr style="border-bottom: 1px solid black;"> 
   <td style="border-right: 1px solid black; padding: 4px; text-align: center;"><strong>V2_SMALL</strong></td> 
   <td style="border-right: 1px solid black; padding: 4px; text-align: center;">49326 ns/op</td> 
   <td style="border-right: 1px solid black; padding: 4px; text-align: center;">24.96%</td> 
   <td style="border-right: 1px solid black; padding: 4px; text-align: center;">58181 B/op</td> 
   <td style="border-right: 1px solid black; padding: 4px; text-align: center;">3.36%</td> 
   <td style="border-right: 1px solid black; padding: 4px; text-align: center;">779 allocs/op</td> 
   <td style="border-right: 1px solid black; padding: 4px; text-align: center;">49.03%</td> 
  </tr> 
  <tr style="border-bottom: 1px solid black;"> 
   <td style="border-right: 1px solid black; padding: 4px; text-align: center;"><strong>V1_LARGE</strong></td> 
   <td style="border-right: 1px solid black; padding: 4px; text-align: center;">208793 ns/op</td> 
   <td style="border-right: 1px solid black; padding: 4px; text-align: center;">–</td> 
   <td style="border-right: 1px solid black; padding: 4px; text-align: center;">283147 B/op</td> 
   <td style="border-right: 1px solid black; padding: 4px; text-align: center;">–</td> 
   <td style="border-right: 1px solid black; padding: 4px; text-align: center;">5515 allocs/op</td> 
   <td style="border-right: 1px solid black; padding: 4px; text-align: center;">–</td> 
  </tr> 
  <tr style="border-bottom: 1px solid black;"> 
   <td style="border-right: 1px solid black; padding: 4px; text-align: center;"><strong>V2_LARGE</strong></td> 
   <td style="border-right: 1px solid black; padding: 4px; text-align: center;">138348 ns/op</td> 
   <td style="border-right: 1px solid black; padding: 4px; text-align: center;">40.59%</td> 
   <td style="border-right: 1px solid black; padding: 4px; text-align: center;">224662 B/op</td> 
   <td style="border-right: 1px solid black; padding: 4px; text-align: center;">23.03%</td> 
   <td style="border-right: 1px solid black; padding: 4px; text-align: center;">2563 allocs/op</td> 
   <td style="border-right: 1px solid black; padding: 4px; text-align: center;">73.09%</td> 
  </tr> 
 </tbody> 
</table> 
<p>We will be continuing to benchmark and analyze our clients and find opportunities to further optimize and reduce latencies in the serialization layers.</p> 
<h2>Improved discoverability and usage</h2> 
<p>The new clients have been restructured to ease discoverability of the operations and types. API types have been moved into a separate <code>types</code> package. This reduces the amount of documentation present in the client package, and makes it easier to see the set of operations supported by a service client and the respective inputs and outputs.</p> 
<p>The number of steps required for initiating a service operation has also been reduced in the clients. Service operations are now invoked on the client directly, with each operation method taking the API input, and returning the response or error. This simplification will allow easier mocking of the client when unit-testing your application.</p> 
<h2>Simplified error handling</h2> 
<p>Errors have been revamped to take advantage of the Go 1.13 error <code>Unwrap</code> functionality. Error unwrapping provides the ability to inspect an error or the sequence of errors that occurred by using <code>errors.Is</code> and <code>erorrs.As</code>. This feature allows us to provide you more diagnostic information about the underlying failure. For example, errors returned from the client will be wrapped in a <code>OperationError</code>, which provides meaningful information about which service and operation had a failure, but still retains the ability to retrieve the original error. This can be useful when producing error messages for a centralized logging solution. Let’s look at an example showing how we can use the errors package to determine if an error was due to a specific API response.</p> 
<pre><code class="lang-go">if err != nil {
	if errors.Is(err, &amp;types.ConflictException{}) {
		panic("user already has a session in progress")
	} else {
		panic("failed to put session: " + err.Error())
	}
}</code></pre> 
<p>Here we use errors.Is to check if the error returned is due to a <code>ConflictException</code>, this method will return true if the returned error was a <code>ConflictException</code> or was caused by ConflictException. Similarly <code>errors.As</code> can be used to perform a type assert on the sequence of errors and inspect the underlying error type.</p> 
<pre><code class="lang-go">if err != nil {
	var conflictException *types.ConflictException
	if errors.As(err, &amp;conflictException) {
		panic("conflict exception message: " + *conflictException.Message)
	} else {
		panic("failed to put session: " + err.Error())
	}
}
</code></pre> 
<h2>Extensible and customizable middleware</h2> 
<p>The client request handler system from the original V1 SDK has been fully replaced to provide a more extensible middleware system that provides finer grained control around how requests are constructed and sent over the wire. Custom middleware can be written to satisfy your specific use cases, or you can use one of the provided helper utilities. The getting started example will show you how to use middleware to add a custom header to your Amazon Lex request.</p> 
<h2>Getting started</h2> 
<p>Using the AWS SDK for Go V2 will look different from the original developer preview. Each client is now contained in a Go module that is independently versioned. This allows you to better control service clients updates and decreases the compiled binary size of your overall application. For this walkthrough, the minimum Go version required is 1.15.</p> 
<ol> 
 <li>Initialize a Go module for your example application. Here we will make a new directory, and initialize our `blog` module. This initialization will create a go.mod file in the directory. <pre><code class="lang-bash">$ mkdir ~/go-blog-example
$ cd ~/go-blog-example
$ go mod init blog</code></pre> </li> 
 <li>In your editor of choice, create a main.go file, and add the following code. This code will load default SDK configuration, and will construct an Amazon Lex client. We will also include import statements that will be used later on. <pre><code class="lang-go">package main

import (
	"context"
	"errors"
	"fmt"

	"github.com/aws/aws-sdk-go-v2/aws"
	"github.com/aws/aws-sdk-go-v2/config"
	"github.com/aws/aws-sdk-go-v2/service/lexruntimeservice"
	"github.com/aws/aws-sdk-go-v2/service/lexruntimeservice/types"
	smithyhttp "github.com/awslabs/smithy-go/transport/http"
)

func main() {
	// Load the SDK's default configuration and credentials values from the environment variables,
	// shared credentials, and shared configuration files
	cfg, err := config.LoadDefaultConfig()
	if err != nil {
		panic("configuration error, " + err.Error())
	}

	// Create a new Amazon Lex Service Client
	client := lexruntimeservice.NewFromConfig(cfg)
}
</code></pre> </li> 
 <li>Now run the module tidy command, which will handle pulling down the referenced import dependencies automatically. <pre><code class="lang-bash">go mod tidy</code></pre> </li> 
 <li>Let’s look at the go.mod file in the editor and review the our projects dependencies. <pre><code class="lang-go">module blog

go 1.15

require (
	github.com/aws/aws-sdk-go-v2 v0.25.0
	github.com/aws/aws-sdk-go-v2/config v0.1.0
	github.com/aws/aws-sdk-go-v2/service/lexruntimeservice v0.1.0
	github.com/awslabs/smithy-go v0.1.0
)
</code></pre> <p>We can see that our application dependencies have now been updated reflect our dependency on the Amazon Lex service client, and the supporting SDK libraries.</p></li> 
</ol> 
<ol start="5"> 
 <li>Now let’s modify the code to make an API call to the service <pre><code class="lang-go">
	// Construct API input parameters
	params := &amp;lexruntimeservice.PutSessionInput{
		Accept:   aws.String("audio/mpeg"),
		BotAlias: aws.String("TEST"),
		BotName:  aws.String("OrderFlowers"),
		UserId:   aws.String("myUserId"),
		DialogAction: &amp;types.DialogAction{
			IntentName:   aws.String("OrderFlowers"),
			SlotToElicit: aws.String("FlowerType"),
			Type:         types.DialogActionTypeElicit_slot,
		},
	}

	response, err := client.PutSession(context.Background(), params)
	if err != nil {
		var conflictException *types.ConflictException
		if errors.As(err, &amp;conflictException) {
			panic("conflict exception message: " + *conflictException.Message)
		} else {
			panic("failed to put session: " + err.Error())
		}
	}

	fmt.Printf("put session response: %+v\n", response)
</code></pre> </li> 
 <li>Let’s try adding a custom header to our request using a function that utilizes the new middleware design. Here we will modify our API invocation method signature and mutate the client options for the call. <pre><code class="lang-go">response, err := client.PutSession(context.Background(), params, func(options *lexruntimeservice.Options) {
	options.APIOptions = append(options.APIOptions, smithyhttp.AddHeaderValue("X-Custom-Header", "YourCustomValue"))
})</code></pre> </li> 
</ol> 
<h2>What’s next for the developer preview?</h2> 
<p>The AWS SDKs are built in the open-source, and so is this&nbsp;developer preview. &nbsp;We are going to be focusing on delivering new improvements to the clients introduced today, including support for additional service specific features, Signature Version 4 pre-signing of operations, and paginators. Your engagement is critical for us to help make the AWS SDK for Go the best end-to-end experience for developing on the AWS platform. Tell us about your experience on&nbsp;<a href="https://github.com/aws/aws-sdk-go-v2/issues/new/choose">GitHub </a>with +1s or feature requests.</p>
