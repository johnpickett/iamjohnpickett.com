---
ID: 73
post_title: Modern Cloud for Beginners (like me)
author: john
post_excerpt: ""
layout: post
permalink: >
  https://iamjohnpickett.com/2018/03/modern-cloud-for-beginners-like-me/
published: true
post_date: 2018-03-03 15:35:33
---
Having a bit of spare time, I decided to take a fresh look at my personal infrastructure over the Christmas 2017 holiday. I wanted to modernize my environment (basically unchanged since 2004; slightly updated in 2011) to enable new capabilities and transition to a cloud-integrated mindset, while also optimizing my costs as much as possible. I've learned a vast amount and wanted to summarize my perspective, hopefully getting some feedback where I'm off or where I'm on par ;-)
<h2>What is "Cloud"?</h2>
Consuming new ideas and concepts runs something like this for me:
<blockquote>Awareness. Understanding. Consideration.</blockquote>
It took me longer than I'd like to admit to really dig into understanding the potential and exponential innovation associated with cloud design concepts and strategies. A few years ago in my awareness stage, I was trying to convey what cloud means to one of my employees. I told him the most concise way I could sum it up:
<blockquote>"Discrete composable resources."</blockquote>
Then expanded into what that meant to me at the time. I think the greatest clarity I've found in past few months is about <em>how</em> resources are composed. I have really latched onto the <a href="https://www.terraform.io/" target="_blank" rel="noopener nofollow">Terraform</a> methodology and, finding success, am faithfully (albeit slowly) working backwards and forwards through the entire <a href="https://www.hashicorp.com/" target="_blank" rel="noopener nofollow">HashiCorp Suite</a>. I plan to write a technical article outlining my architecture, but if you're interested in Terraform, let me save you a few hours and point you to <a href="https://github.com/gruntwork-io/terragrunt" target="_blank" rel="noopener nofollow">Terragrunt</a> (Terraform OOS wrapper to enable multi-service deployments).

<img class="aligncenter size-large wp-image-74" src="https://iamjohnpickett.com/wp-content/uploads/2018/03/AAMAAQDGAAgAAQAAAAAAAA9iAAAAJGIyMzA0NzhjLTM3MDAtNDMyYS1hYmM1LThhYTkwNDg1MmUzZQ-1024x337.png" alt="" width="525" height="173" />

I now have a slightly updated understanding on what cloud means to me:
<blockquote>"Intelligently composed services utilizing resources from any provider."</blockquote>
I had always struggled to manage both resource orchestration and service validation. It's easy enough to spin something up, but then you have to track so much STUFF if you want to make sure it's doing what you want it to. Enter <a href="https://en.wikipedia.org/wiki/Infrastructure_as_Code#Types_of_approaches" target="_blank" rel="noopener nofollow">IaC</a>, specifically a declarative/intelligent approach with a touch of procedural where appropriate.

Imagine if the process for creating a service was identical to the process for validating it, oh yeah and fixing it. That sounds incredibly complex. Now imagine if it were this easy:
<pre class="ql-syntax" spellcheck="false">aws_region = <span class="hljs-string">"us-west-2"</span>
gcp_project = <span class="hljs-string">"omniapp-xxxx"</span>
gcp_region = <span class="hljs-string">"us-central1"</span>
name     = <span class="hljs-string">"core04"</span>
instance_<span class="hljs-built_in">type</span> = <span class="hljs-string">"t2.nano"</span>
key_name   = <span class="hljs-string">"secret"</span></pre>
That's all the config I need to build an AWS EC2 instance with DNS registered in GCP Cloud DNS. Even better, I know that those resources are constantly being validated for changes, and will automatically be repaired if at all possible. Instead of going down this rabbit hole though, the remainder is focused on my personal view about the strengths and considerations of the major cloud providers I consume.
<h2>Cloud Providers</h2>
I currently have tenancy and resources deployed across several cloud providers including <a href="https://cloud.google.com/" target="_blank" rel="noopener nofollow">GCP</a>, <a href="https://aws.amazon.com/" target="_blank" rel="noopener nofollow">AWS</a>, <a href="https://www.dreamhost.com/cloud/computing/" target="_blank" rel="noopener nofollow">DreamCompute</a>, and <a href="https://www.digitalocean.com/" target="_blank" rel="noopener nofollow">DigitalOcean</a>. I have tenancy in <a href="https://azure.microsoft.com/en-us/" target="_blank" rel="noopener nofollow">Azure</a> as well, but recently migrated my only remaining long-term project to Google (NLP, Conversation UI [who I call Buddy Boy]). I may look at some of their other services (Event Grid?), but I typically find other APIs easier to consume.

As I alluded, a key to my success has been ease of integration. This is where researching each providers offerings pays off. While the customer-visible interface may be fairly standard across providers (i.e. you access an Ubuntu compute via key-based SSH or console-based web UI; they each have a similar web UI method to create a VM), providers will make different assumptions unless specified, and sometimes other supportive components are required to be specified with one provider, while another does it automatically. This leads to a particular service from one provider more favorable (to me) than others. The trick is identifying all of the "best for you" services from a variety of providers, and then use them in a cohesive way (did I mention I love Terraform?). Of course, you also need to consider that various tools and other modules make integration on a given platform easier still, complicating things further.

Another consideration is cost. Sometimes replacing a more consumable or feature-rich service with something utilitarian that still meets your requirements (or maybe determine other mitigations for any deficiency) will do just fine. If you have good tooling, choosing one over another becomes trivial.

A few real examples that hopefully demonstrate the potential complexity when assessing cloud options:
<h3>Compute</h3>
For general compute, I use DreamCompute and have been exploring DigitalOcean recently. The cost is low, SLAs are great for my needs, and they both have APIs with plenty of examples. These are my 24/7 boxes that don't have any specific dependency on low-latency integration with resources from other providers. Planning resource placement is a whole other topic; I'm not going to schedule some Spark Machine Learning experiment using compute from one provider and terabytes of datasets from another.

I've considered figuring out if I can leverage <a href="https://aws.amazon.com/ec2/pricing/reserved-instances/" target="_blank" rel="noopener nofollow">Reserved Instances</a> on AWS to reduce my costs for these consistent loads, but what I'm more interested in are <a href="https://cloud.google.com/compute/docs/instances/preemptible" target="_blank" rel="noopener nofollow">Preemptible</a> ones on GCP. I'm trying to design my services in a resilient enough way that I gain benefit of the incredible resource geo-availability and distribution of GCP or AWS while also being able to sustain immediate (30-second notice) and continual (always after 24 hours if not before) destruction. Yikes! But exhilarating! The immensely discounted rate of compute, if you can function within those constraints, allows greater resource scaling, across either axis or both, at less cost. My current compute strategy is something like <a href="https://cloud-init.io/" target="_blank" rel="noopener nofollow">cloud-init</a>, <a href="https://www.ubuntu.com/containers/lxd" target="_blank" rel="noopener nofollow">LXD</a>, <a href="https://saltstack.com/" target="_blank" rel="noopener nofollow">salt</a>, <a href="https://www.docker.com/" target="_blank" rel="noopener nofollow">Docker</a>, and Preemptible Instances on GCP. Having said that, <a href="https://aws.amazon.com/ec2/" target="_blank" rel="noopener nofollow">EC2</a> is where I do most of my prototyping when I just want to get it done, mainly because it's the most familiar.
<h3>DNS</h3>
This is a perfect example of where I don't necessarily need feature-rich <a href="https://aws.amazon.com/route53/" target="_blank" rel="noopener nofollow">Route 53</a> from AWS and feel pretty comfortable using GCP <a href="https://cloud.google.com/dns/" target="_blank" rel="noopener nofollow">Cloud DNS</a> at a lower cost for my use case. For many practical use cases however, they're virtually identical in cost. I still do most of my DNS through DreamHost since it's included as part of my annual subscription and so I'm only targeting automation-focused domains for GCP.

So why DNS in the cloud? Features and API. I needed the CAA record type for automatic SSL certificate fulfillment, which to my knowledge DreamHost doesn't support so I had to look elsewhere. Knowing the zones in GCP are also served by a global anycast DNS system is a great warm and fuzzy as well. Also, most cloudy tools have simple integration into Cloud DNS, making developing services... easier.

Personal Assistant

Early on this path, I was lamenting with a friend how I was at the point where I wanted some kind of UI and was dreading hacking together some kind of web UI, especially when I knew my real goal was to have something API-driven and I didn't have an API yet. I don't know how many beers in we were at the time, but we were also exploring (more like abusing) one of the Google Home Mini's I have. He said, "Hey Google, give me a f@#$ing server!". It wasn't a proud moment, but it was profound, especially to a few inebriated geeks.

21 days later...

[embed]https://youtu.be/IT6exxhTxQg[/embed]

Buddy - <a href="https://dialogflow.com/" target="_blank" rel="noopener nofollow">Dialogflow</a>, <a href="https://firebase.google.com/" target="_blank" rel="noopener nofollow">Firebase</a> Functions (<a href="https://nodejs.org/en/" target="_blank" rel="noopener nofollow">Node.js</a>), <a href="https://developers.google.com/actions/" target="_blank" rel="noopener nofollow">Actions on Google</a>:
<ul>
 	<li>Requests new compute services (ready in less than 10 minutes) - Terraform, Terragrunt, Firebase Database &amp; Storage, <a href="https://github.com/" target="_blank" rel="noopener nofollow">GitHub</a></li>
 	<li>Helps me schedule Bill Pay (and send templated snail mail, but who does that anymore) - <a href="https://lob.com/" target="_blank" rel="noopener nofollow">Lob</a></li>
 	<li>Keep me quickly updated with the current value of various blockchain - <a href="https://www.binance.com/" target="_blank" rel="noopener nofollow">Binance</a></li>
 	<li>All while serving up a ubiquitous interface through <a href="https://assistant.google.com/intl/en_us/" target="_blank" rel="noopener nofollow">Google Assistant</a>, SMS (via <a href="https://www.twilio.com/" target="_blank" rel="noopener nofollow">Twilio</a>), or <a href="https://slack.com/" target="_blank" rel="noopener nofollow">Slack</a>.</li>
</ul>
I have a few more ideas for this guy but am waiting until I refine everything else a bit before I continue.
<h3>Biggest Surprise</h3>
I've been a cloud consumer for a while, but not really a developer. Not <em>really.</em> I didn't realize how crucial the API is to consuming cloud. I think I get it now, and here's what I see the typical breakdown for a DevOps mindset:
<ul>
 	<li>Web UI [Aware]: This is how you get going. Need to test something, try out a new service, or otherwise easily see how 'it' works? Quick and easy. If you find yourself in here most of the time, you're probably aware of and interested in cloud but unsure how to really flex it.</li>
 	<li>API / Clients [Understand/Consider]: This is where you start taking what you've learned about a cloud resource and begin to leverage it for some kind of automation. If you're using this interface to manage your resources, you're well down the path.</li>
 	<li>CLI [Ops]: I have switched to CLI for most of what I was using the Web UIs for. It's a good way to quickly interact with a cloud service, in a way that most are familiar with. I hope to build my operational tasks into my service designs as much as possible, but for anything I have to touch, it's handy.</li>
</ul>
<h3>Cloud Provider Summary</h3>
My general direction is settling towards Google Cloud Platform. They blend ease-of-use, capabilities/features, and price in a way that works well for most of my use cases. I also consume several Google services personally and integration with those tends to be easy peesy from GCP.

Amazon Web Services is rock solid of course and it's basically sixes in most instances. The best part though is that it doesn't really matter. With how I view building services now, my goal is to abstract even provider configuration differences to allow redeployment through a simple key:value type config change.

I've got many other aspects of my overall design to consider including my own API (<a href="https://feathersjs.com/" target="_blank" rel="noopener nofollow">Feathers</a>, <a href="https://loopback.io/" target="_blank" rel="noopener nofollow">Loopback</a>?) and really taking a deeper look at <a href="https://www.consul.io/" target="_blank" rel="noopener nofollow">Consul</a> / <a href="https://www.nomadproject.io/" target="_blank" rel="noopener nofollow">Nomad</a> to make sure I integrate their capabilities instead of stress about duplicating them. Right now, I'm working to build my images using Vagrant/Packer. Then I'll have a consistent baseline wherever I deploy and will then start building the rest of HashiCorp Suite out, taking it slow to figure out how Terraform, Consul, and Nomad work best together.