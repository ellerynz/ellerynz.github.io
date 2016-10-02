---
layout: post
title: Hooking into Github
category: Rails
tags: Rails github webhooks ngrok
year: 2016
month: 10
day: 02
summary: Webhooks are a pretty simple way to get event data from a service.
---

It's pretty simple to get information from your Github PRs using webhooks.
Let's say you want a form of changelog and the titles of your PR are good enough.
Github will send you information about the PR which you can then pick apart to do what you need with it.

### App
Imagine a basic rails app that stores changelog entries.
When a PR is made, we want Github to send the PR info to the app. We'll then store it as an entry.

### Github Webhooks
When an event occurs, POST some relevant data to a specified URL.
You can set one up for a repo under settings. 'Webhooks & services' is on the left.
We're interested in the [pull request](https://developer.github.com/v3/activity/events/types/#pullrequestevent) event.
But first...

### Ngrok
We need a URL for the webhook to send data to. [Ngrok](https://ngrok.com/) is a neat little tool that can allow something you're running on local to be a public URL.
This way we can avoid having to deploy anything while testing out the webhook. Running `/.ngrok http 3000` should give something like so `http://abcd1234.ngrok.io`.

Another advantage is the ability to replay requests by the interface at `http://localhost:4040`.
That way, we don't need to keep creating/changing PRs to trigger a request.

### Plugging it in
Now we can plug that into the webhooks setup with a minor alteration.
We want the rails app to create an entry, the webhook is a POST, so we'll send it to the CREATE path `http://abcd1234.ngrok.io/entries`.

It still won't _quite_ work. Rails [`protect_from_forgery`](http://api.rubyonrails.org/classes/ActionController/RequestForgeryProtection/ClassMethods.html) is checking for csrf tokens for the create action - the webhook POST request didn't come from a page in our rails app so it won't have the token.
Easiest for now is to skip the before action only for `entries#create`.

### Making it useful
Everything is set up now. Creating a PR or adding a label to one, sends a bunch of PR info to the route we specified.
At the moment we're creating an entry on every change.
If we only wanted PRs merged to master, we can pull out the information like so:

```
# Merged && closed == merged PR
params[:pull_request][:merged] && params[:pull_request][:state] == 'closed'

params[:pull_request][:base][:ref] == branch
```

### Keeping it secure
We don't want to create data from any old request that has the right format.
We should check that it comes from [Github](https://developer.github.com/webhooks/securing/).
We need to:

- Add a secret token in the webhook settings e.g. `SecureRandom.hex(20)`
- Add an environment variable with the same value `ENV['SECRET_TOKEN']`. For development, I like the approach outlined [here](http://stackoverflow.com/a/11765775/847664).
- Add a similar like the securing doc suggests:

```
def verify_signature
  payload_body = request.body.read
  signature = 'sha1=' + OpenSSL::HMAC.hexdigest(OpenSSL::Digest.new('sha1'), ENV['SECRET_TOKEN'], payload_body)
  return halt 500, "Signatures didn't match!" unless Rack::Utils.secure_compare(signature, request.env['HTTP_X_HUB_SIGNATURE'])
end
```

The final file would look something like [this](https://gist.github.com/ellerynz/dac6a835500f2db0a9dca53a7feda220).
