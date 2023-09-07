# Our first web app with Sinatra

Now that we've seen how to _send_ HTTP requests with Ruby, let's learn how to _receive_ HTTP requests and send back HTTP responses. In other words, it's time to finally do what we set out to do at the beginning of this course: **build a dynamic web application**.

In order to listen for HTTP requests, we need a piece of software called a "web server". It would be an interesting challenge to write a web server from scratch, but it's not the best use of our time right now. So, we'll use the Puma gem, one of the many free, open-source, and powerful Ruby web servers that are available.

In keeping with our philosophy of not reinventing wheels, in order to parse HTTP requests, assemble valid HTTP responses, and send them out, we'll use another gem: Sinatra. Sinatra is a lightweight and flexible framework for building web applications.

Let's get started!

---

[Here is a video for this lesson](https://share.descript.com/view/yWLc5Hb7vXy). You should not rely entirely on the video. PLEASE READ the below lesson as you are going through the steps, since there is much more detail in the text than in the video.

## Create a codespace

For our first dynamic web app, we'll write a simple tool that displays the outcome of dice rolls — but this time, in the browser, rather than in the terminal!

We'll use the `appdev-projects/sinatra-dice-roll` repository to explore. Since there are no automated tests included in this project, [you can click here to fork it directly to your personal account](https://github.com/appdev-projects/sinatra-dice-roll/fork). Then, set up a codespace based on your fork.

The sinatra-dice-roll repo is simply a copy of the `appdev-projects/ruby-template` repo — in other words, it starts out completely blank. We will be building up our first web application starting from scratch.

Our target for this lesson and the next is to build an app that matches this: 

[dice-roll.matchthetarget.com](https://dice-roll.matchthetarget.com/)

## Create a Gemfile

First, we need to pull in the `puma` and `sinatra` gems. Let's create a `Gemfile` and add them:

```ruby
# /Gemfile
source "https://rubygems.org"

gem "puma"
gem "sinatra"
```

And then don't forget to `bundle install` to download and install everything in the `Gemfile`.

## Create our program

Now that we have our dependencies squared away, let's create our program. Create a file called `dice.rb`, and within it, `require "sinatra"`:

```ruby
# /dice.rb

require "sinatra"
```

If we run the program now, we'll already see interesting behavior. At a bash prompt, run the command `ruby dice.rb` and observe:

```
sinatra-dice-roll main % ruby dice.rb 
== Sinatra (v3.0.6) has taken the stage on 4567 for development with backup from Puma
Puma starting in single mode...
* Puma version: 6.3.0 (ruby 3.2.1-p31) ("Mugi No Toki Itaru")
*  Min threads: 0
*  Max threads: 5
*  Environment: development
*          PID: 2733
* Listening on http://127.0.0.1:4567
* Listening on http://[::1]:4567
Use Ctrl-C to stop
█
```

Even though all we've done so far is `require "sinatra"`, our program already has a lot of functionality; when executed, it starts up the Puma web server and begins listening for HTTP requests!

Notice that the program did not execute, finish, and return us to the bash prompt as usual; instead, it says "Use Ctrl-C to stop".

When we start a web server, the terminal tab in which we started it goes into an infinite loop of waiting for HTTP requests. The server also prints information about all incoming requests, to help us understand our traffic and debug. This output is known as **the server log**.

If we want to run any other bash commands in this terminal tab, we must shut down the server with <kbd>Ctrl</kbd>+<kbd>C</kbd>; or, my preference is to open a new terminal tab to get a fresh bash prompt to execute other commands.

To open the live app preview in a new browser tab, visit the "Ports" tab in the bottom pane, look for the port with a green dot 🟢 on the left side (indicating that it is active), and hover over the "Local Address" column, then click the globe 🌐 button to "Open in Browser":

![](https://res.cloudinary.com/dmxgp9oq2/image/upload/v1688656108/ports-tab_cfaa1k.png)

If you click on the "Ports" tab and see "No forwarded ports", like this:

![](https://res.cloudinary.com/dmxgp9oq2/image/upload/v1688680107/no-forwarded-ports_rapxoa.png)

Then click on "Forward a port" and then enter 4567.

---

We haven't told the application how to respond to any requests yet, so it's not very exciting yet; but it is up and running!

![](https://res.cloudinary.com/dmxgp9oq2/image/upload/v1688656184/sinatra-404_zqdahu.png)

## Our first route

On the homepage of our live app preview, Sinatra has some good advice for us. It says to try this:

```ruby
get("/") do
  "Hello World"
end
```

Let's add that to our program:

```ruby
# /dice.rb

require "sinatra"

get("/") do
  "Hello World"
end
```

Now try refreshing your live app preview — and you'll see that nothing happens yet 😔 

It turns out that we need to shut down and restart the server in order for it to pick up changes to our code. Annoying! We'll fix this soon, but for now press <kbd>Ctrl</kbd>+<kbd>C</kbd> in the terminal tab running the web server to shut it down, and then `ruby dice.rb` to start it up again, and then try visiting the live app preview again:

<aside markdown="1">
Sometimes at this point, some text gets inserted at the top of `dice.rb`:

```ruby
Found no changes, using resolution from the lockfile
require "sinatra"

get("/") do
  "Hello World"
end
```

If that happened in your case, delete the "Found no changes...." text on the first line before re-running the program. This will not be a problem for long.
</aside>

![](https://res.cloudinary.com/dmxgp9oq2/image/upload/v1688656750/sinatra-hello-world_rif0wn.png)

Yay! What just happened?

Recall that an HTTP request looks something like this:

```http
GET /wiki/Chicago HTTP/1.1
Host: en.wikipedia.org
```

This request, when sent, will wind up at the server located at `en.wikipedia.org`; and that server will try to **match** the resource path (in this example, `/wiki/Chicago`). If it has a match for that path, then it will send back a response with a code of `200` (all went well); otherwise it will send back a `404` (resource not found).

A resource path of only the bare slash by itself and nothing after it (`/`) is special — we refer to this as the "root URL" or "homepage". So when we visited the root URL of our live app preview with our browser, the browser sent a request that looks like this:

```http
GET / HTTP/1.1
Host: raghubetina-curly-computing-machine-www7pwj47h59gw-4567.preview.app.github.dev
```

The `Host` is different for each of us based on our GitHub username and codespace name; and eventually when we deploy our apps from our development environment to real infrastructure, the host will be `ourdomain.com`.

When we first ran `dice.rb`, we didn't have any instructions in it to handle a request for `/`. So, when a request came in for that resource, Sinatra sent back the standard 404 (resource not found) page, "Sinatra doesn't know that ditty".

But then, we added our first **route**:

```ruby
get("/") do
  "Hello World"
end
```

The `get()` method was defined by Sinatra when we `require`d it. The argument to `get()` must be a `String` containing a resource path that we want to support — in this case, the root path (`/`).

We must also provide a block (between the `do` and `end`) containing Ruby code that we want to be executed when someone places a request for that path. This block, the code we want to be executed when an HTTP request comes in, is referred to as an "**action**".

An action can contain however many lines of code that we need, but the _last evaluated expression_ in the action is what Sinatra sends back as the body of our HTTP response. In this case, `Hello World`.

So, ultimately (after adding the route and restarting the server), when we used our browser to place a request for `/`, Sinatra assembled and sent back the following HTTP response:

```http
HTTP/1.1 200 OK
Content-Type: text/html

Hello World
```

And then the browser displayed the body of the response in its viewport.

Let's try visiting another path in the live app preview. Add `/zebra` on to the end of the URL of your live app preview. For me, this becomes:

```
https://raghubetina-curly-computing-machine-www7pwj47h59gw-4567.preview.app.github.dev/zebra
```

Notice the `/zebra` on the very end, after the host. Your host will be different since your codespace has a different live app preview URL than mine.

When you visit `/zebra`, you should see something like this:

![](https://res.cloudinary.com/dmxgp9oq2/image/upload/v1688664664/sinatra-another-404_yueb8b.png)

When a request came in to `GET /zebra`, Sinatra didn't find a matching route, so it sent back the 404 page.

Let's add a route for the request `GET /zebra`:

```ruby
get("/zebra") do
  "We must add a route for each path we want to support"
end
```

Restart your server and then try visiting `/zebra` in your live app preview again. Now you should see the content we prepared:

![](https://res.cloudinary.com/dmxgp9oq2/image/upload/v1688662681/sinatra-second-route_qdtrti.png)

Congratulations on your first two routes! Now get ready to write 1,000,000 more, because this is going to be our job for the rest of our careers as web developers:

1. Decide on a path we want users to be able to visit.
2. Add a route for that path.
3. Write some code within the action to handle the request.
4. Send back an HTTP response.

Onwards!

---

**Checkpoint:**

- [Here you can see the changes that I've made since the last commit.](https://github.com/raghubetina/sinatra-dice-roll/commit/2e3a66f17929489c8b0584f15535897b84c5cafd)
- [Here you can browse my entire codebase at this point in time.](https://github.com/raghubetina/sinatra-dice-roll/tree/2e3a66f17929489c8b0584f15535897b84c5cafd)

## Automatically reloading code

Before we go any further, let's take care of the huge annoyance of having to shut down and restart our web server any time we make a change to our code.

Perhaps unsurprisingly at this point, we're going to use a gem for that, called `sinatra-contrib`. Let's add it to our `Gemfile`:

```ruby
gem "sinatra-contrib"
```

and then `bundle install`.

Next, add the following `require` statement to `dice.rb`:

```ruby
require "sinatra/reloader"
```

And finally, make sure to shut down and restart your server.

With that done, add a third route — maybe something like this:

```ruby
get("/giraffe") do
  "Hopefully this shows up without having to restart the server 🤞🏾"
end
```

And then try visiting `/giraffe` in your live app preview. It should appear without needing a server restart — phew! For all of our apps from now on, we'll add this right from the start.

**Checkpoint:**

- [Here you can see the changes that I've made since the last commit.](https://github.com/raghubetina/sinatra-dice-roll/commit/dd524d738d0644d0553d65053bc68c18245037aa)
- [Here you can browse my entire codebase at this point in time.](https://github.com/raghubetina/sinatra-dice-roll/tree/1f3384d10ba963df707bfdd2c96a3566a957f0b1)


## Dynamic responses

You might be wondering, "So what? We were able to make something show up in the browser ages ago, back when we first started studying HTML, simply by creating `.html` files in the root folder. Why go to all this extra trouble?"

The reason, my friends, is because this is an interactive web application; not a static web site. In other words, the responses we send back can be _dynamic_; rather than unchanging HTML files that we write by hand.

For example: let's add a route for the request `GET /dice/2/6`. When someone sends that request, we should simulate rolling two 6-sided dice on their behalf. I.e., we should:

- Generate two random numbers between 1 and 6.
- Add them up.
- Send back both numbers and the total in the body of the response.

This time, let's also add some HTML tags to the content:

- An `<h1>` that says `2d6` (short for "two 6-sided dice")
- A `<p>` tag wrapping the rolls and the total.

```ruby
get("/dice/2/6") do
  first_die = rand(1..6)
  second_die = rand(1..6)
  sum = first_die + second_die
	
  outcome = "You rolled a #{first_die} and a #{second_die} for a total of #{sum}."
	
  "<h1>2d6</h1>
   <p>#{outcome}</p>"
end
```

Note that I'm using the [interpolation technique for adding strings together, rather than `+` and `to_s`](https://learn.firstdraft.com/lessons/113-ruby-intro-printing-and-string-interpolation#string-interpolation-alternative-to-to_s). This is a very handy technique, and I will start using it a lot from now on — adding strings together with `+` and having to worry about `.to_s` is very tedious for me, especially now that we're crafting these very long strings that will become HTTP response bodies.

Now try visiting `/dice/2/6` in your live app preview. You should see a response that was dynamically generated on the fly:

![](https://res.cloudinary.com/dmxgp9oq2/image/upload/v1688676775/dynamic-1_gqsueq.png)

If you place the request again, i.e. refresh the page, then you'll see _different_ content:

![](https://res.cloudinary.com/dmxgp9oq2/image/upload/v1688676791/dynamic-2_kbpiq2.png)

And this is what all of our hard work has been leading up to! We learned HTML, and Ruby, so that we could put them both together to generate pages _on the fly, dynamically_.

## Some practice

To develop some muscle memory, try adding support for the following requests. As you do so, _don't copy-paste_; type each on out. You won't learn much by copy-pasting.

- `GET /dice/2/10` (simulate two 10-sided dice)
- `GET /dice/1/20` (simulate one 20-sided die)
- `GET /dice/5/4` (simulate five 4-sided dice)
- As many more as you feel up to, until you can type out the whole route and action without referring to previous ones.
- As you build out support for each path, add a link to your homepage leading to that path, to make it easy for users (and yourself) to get there. Typing into the address bar over and over will be a drag! Your homepage should end up looking something like this:

<aside markdown="1">
The standard 7-dice set for the popular tabletop roleplaying game Dungeons & Dragons includes a 4-sided die, a 6-sided die, an 8-sided die, two 10-sided dice, a 12-sided die, and a 20-sided die. Gamers often prefer to have multiples of some die in order to roll pools of dice, such as 6d6 in one roll instead of repeating an individual d6 roll six times.
</aside>

![](https://res.cloudinary.com/dmxgp9oq2/image/upload/v1688676994/homepage-with-nav_snzqzu.png)

### Better error pages

As you are practicing, you will likely run into an error at some point; perhaps you will make a typo in a variable name, or forget a `"`, or something.

For example, while I was typing out `GET /dice/2/10`, I made a typo in one of my variable names:

```ruby
outcome = "You rolled a #{first_die} and a #{second_due}."
```

When I tried visiting `/dice/2/10`, I saw this:

![](https://res.cloudinary.com/dmxgp9oq2/image/upload/v1688675193/default-errors_ty8jft.png)

While the default error page did give me enough of a hint with "NameError at /dice/2/10 undefined local variable or method "second_due'" to find my mistake, it's not the friendliest of messages. Fortunately, some members of the Ruby community have written gems that provide a much better debugging experience.

Add the following gems to your `Gemfile`:

```ruby
gem "better_errors"
gem "binding_of_caller"
```

Add the following require statements to the top of `dice.rb`:

```ruby
require "better_errors"
require "binding_of_caller"
```

Below those require statements, you also need to add the following:

```ruby
# Need this configuration for better_errors
use(BetterErrors::Middleware)
BetterErrors.application_root = __dir__
BetterErrors::Middleware.allow_ip!('0.0.0.0/0.0.0.0')
```

This looks a bit mysterious, but just trust me for now.

Make sure to shut down your server, `bundle install`, and then start the server again.

Now, try planting a bug on purpose, and then visit the buggy path. You should see a much different looking error page:

![](https://res.cloudinary.com/dmxgp9oq2/image/upload/v1688677613/better_errors_vxhcyo.png)
{: .bleed-full }

The fact that `better_errors` automatically throws a breakpoint in for us to give us an interactive debugger right after the error is incredibly helpful. From now on, we'll include `better_errors` and its required configuration in all of our apps from the very beginning.

**Checkpoint:**

- [Here you can see the changes that I've made since the last commit.](https://github.com/raghubetina/sinatra-dice-roll/commit/cfb7733f66f3e9ab6b91b2183ce1bfa9b9c28698)
- [Here you can browse my entire codebase at this point in time.](https://github.com/raghubetina/sinatra-dice-roll/tree/cfb7733f66f3e9ab6b91b2183ce1bfa9b9c28698)

---

How does your app look at this point? Hopefully you are getting closer to the target: 

[dice-roll.matchthetarget.com](https://dice-roll.matchthetarget.com/)

We will add the navigation links and make some changes to the dice pages in the next lesson.

But for now: Congratulations on your first dynamic web app! 🎉

Next, we'll cleanup our code with embedded Ruby view templates, and then we'll practice everything by making a simple game: Rock, Paper, Scissors.

- Approximately how long (in minutes) did this lesson take you to complete?
{: .free_text_number #time_taken title="Time taken" points="1" answer="any" }
	
---
