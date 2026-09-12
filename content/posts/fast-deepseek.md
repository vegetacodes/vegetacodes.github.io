---
title: "fast-deepseek: Building a Lightweight DeepSeek Client for Ruby"
date: 2025-03-26T18:12:39-04:00
draft: false
tags: ["Ruby", "AI", "DeepSeek", "Open Source", "Developer Tools"]
---

AI APIs have become remarkably easy to call.

The hard part isn't sending an HTTP request. It's deciding how much abstraction you actually need.

When I started experimenting with DeepSeek from Ruby, I didn't want an AI framework, an agent abstraction, or a large dependency tree. I wanted this:

```ruby
client = FastDeepseek::Client.new(
  api_key: ENV.fetch("DEEPSEEK_API_KEY")
)

response = client.chat(
  "Explain Ruby blocks in simple terms",
  model: "deepseek-chat"
)
```

That idea became [fast-deepseek on RubyGems](https://rubygems.org/gems/fast-deepseek) and [fast-deepseek on GitHub](https://github.com/your-handle/fast-deepseek) — a lightweight Ruby client for the DeepSeek API.

## Why another client?

Ruby has a tradition of small libraries that do one thing well. For an external API client, I want three things:

1.  A small public API
2.  Reasonable defaults
3.  No opinions about my application architecture

I don't need my API client to know about my models, controllers, jobs, or prompts. Those are application concerns. The client should handle communication with DeepSeek and get out of the way.

## The smallest useful API

Add it to your Gemfile:

```ruby
gem "fast-deepseek"
```

Configure and call it:

```ruby
client = FastDeepseek::Client.new(
  api_key: ENV.fetch("DEEPSEEK_API_KEY")
)

response = client.chat(
  "What makes Ruby's blocks so powerful?",
  model: "deepseek-chat"
)

puts response.dig("message", "content")
```

`fast-deepseek` unwraps the raw DeepSeek response so you don't have to dig through `choices[0].message`. The important part isn't that the API is complicated. It's that it isn't.

## Why not build a larger abstraction?

If you need multiple providers, RAG, embeddings, agents, or tool calling, you should use a framework. That's a different problem.

I wanted `fast-deepseek` to sit one level lower:

```
Rails Application (business logic)
        ↓
   fast-deepseek (communication)
        ↓
    DeepSeek API
```

The Rails app owns the logic. The gem owns the HTTP.

## Using it in a Rails app without coupling

It's tempting to call the client directly from a controller. That quickly couples the controller to which provider you use, which model you use, and how to parse the response.

A service object gives you a cleaner boundary and a place for dependency injection:

```ruby
# app/services/deepseek_service.rb
class DeepseekService
  def initialize(client: default_client)
    @client = client
  end

  def ask(question)
    response = @client.chat(question, model: "deepseek-chat")
    response.dig("message", "content")
  end

  private

  def default_client
    FastDeepseek::Client.new(
      api_key: ENV.fetch("DEEPSEEK_API_KEY")
    )
  end
end
```

Now the controller only handles HTTP, and the slow work goes to a background job. AI calls can take seconds, you don't want to block a web request for that:

```ruby
class QuestionsController < ApplicationController
  def create
    question = Question.create!(
      content: params[:question],
      status: :processing
    )

    GenerateAnswerJob.perform_later(question.id)

    render json: { id: question.id, status: question.status }
  end
end

class GenerateAnswerJob < ApplicationJob
  queue_as :default

  def perform(question_id)
    question = Question.find(question_id)
    answer = DeepseekService.new.ask(question.content)
    question.update!(answer: answer, status: "completed")
  end
end
```

Controller -> Question -> Job -> Service -> fast-deepseek -> DeepSeek. Each layer has one job.

## Testing without hitting the network

Because the service accepts a client, you can test it with a double:

```ruby
RSpec.describe DeepseekService do
  it "returns the generated response" do
    client = instance_double(FastDeepseek::Client)
    allow(client).to receive(:chat).and_return(
      { "message" => { "role" => "assistant", "content" => "Ruby is a programming language." } }
    )

    service = described_class.new(client: client)

    expect(service.ask("What is Ruby?")).to eq("Ruby is a programming language.")
  end
end
```

No network request, no API key, no flakiness.

## Failures are part of the design

Once an external API is in your request path, you have to expect failures. Networks fail, rate limits happen, servers go down.

`fast-deepseek` exposes typed errors so your app can decide what to do:

```ruby
begin
  client.chat("Explain Ruby fibers", model: "deepseek-chat")
rescue FastDeepseek::RateLimitError
  Rails.logger.warn("DeepSeek rate limit exceeded")
  # retry with backoff
rescue FastDeepseek::ServerError => e
  Rails.logger.error("DeepSeek server error: #{e.message}")
rescue FastDeepseek::Error => e
  Rails.logger.error("DeepSeek request failed: #{e.message}")
end
```

## The interesting part starts after the client

Once the call is behind a service, you can build useful behavior around it. Keep prompts out of controllers and jobs entirely:

```ruby
class TicketSummarizer
  def initialize(ai_client: DeepseekService.new)
    @ai_client = ai_client
  end

  def call(ticket)
    @ai_client.ask(<<~PROMPT)
      Summarize the following support ticket in three concise bullet points:

      #{ticket.body}
    PROMPT
  end
end
```

The same pattern works for classification, drafting replies, or extracting data. At this point `fast-deepseek` isn't the interesting part anymore. What you build on top of it is.

## What I learned and what's next

The hardest part wasn't the HTTP integration. It was deciding what to leave out. Every abstraction is a cost for both maintainer and user.

For what's next, I want to keep the core small while exploring better streaming support, retry handling, and token usage helpers.

The goal isn't to become another AI framework. The goal is to make this:

```ruby
FastDeepseek::Client.new(api_key: ENV.fetch("DEEPSEEK_API_KEY"))
```

feel as natural as any other small Ruby API client.
