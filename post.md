My Co-founder and I built [LinkTexting.com](https://www.linktexting.com) over a weekend and we've since grown it to $2,000/mo. in recurring revenue for us, we'll tell you how we got there :)

## The Initial Problem

My Co-Founder Kumar was helping a friend on marketing his app, [Let's Walk](http://www.letswalkapp.com). It was a simple app that lets you track just your walking steps and compete against friends to see who would win the week. 

While building it Kumar was optimizing everything to make sure downloads would flow in. One of these was putting a text to download solution on the landing page of the website.

When a person visits the website of a mobile app on their desktop or laptop, there isn't really a straightforward built in way to get the app to your phone. You could email yourself the link or search for it on the app store, but there's a pretty sizeable drop off for converting the download. This is where text to download comes in.

This is the ***highest converting*** way to get a desktop or laptop visitor to download your app. This enables visitors to input their phone number on the website and receive a text with a link to the app that could detect their device type and get them to the right app store.

Kumar asked his friend to put this widget on their website, how hard could it be right?

Turns out there wasn't an easy drop in solution. It took spinning up a backend server and no longer just hosting a static page, connecting to the Twilio API and learning it for the first time as well as validation and design.

This process took almost ***two entire days*** away from working on the core product.

Kumar figured there has to be a better way to do this, and thus the idea for LinkTexting was born.

## Turning down working on LinkTexting

This is when Kumar came to me about this idea, explained his pain point and asked if I'd build it with him. 

At the time I was working on my own started called VUE Analytics, trying to improve upon the Mixpanel's and Flurry's of the world. I came from enterprise and wanted the deep, flexible analytics we had there for my mobile apps so I set out building that.

Kumar repeatedly asked me for 10 weeks to build it and I repeatedly turned him down.

Finally I came around. Him asking me for this long I had essentially built out the product in my head. It was a very simple solution and in the same space as my mobile analytics venture. I agreed to go 50/50 with him and the agreement was I could brand everything "Powered by VUE Analytics" to drive conversions back to my tool.

Then I got to building.

## Building the Product

LinkTexting's initial version took a total of 48 hours to launch, started on a Friday night and continuing through Sunday afternoon. I'll step through the tools I used to build it and how you can build a simple product as well.

### 1) Using Mean.io

Over the last two years I've become very proficient in the MEAN (Mongo, Express, Angular, Nod) stack and I wanted to keep it simple so I stuck with what I knew to get the product in front of customers faster.

The people over at [Linnovate](http://linnovate.net/) built a super simple MEAN boilerplate app that you can download at [http://mean.io](http://mean.io) and have a running MEAN app in under 5 minutes.

I downloaded their boilerplate and immediately had a system where a basic CRUD interface and a user login system existed and was ready to be deployed.

The mean.io project also has a [blog post](http://mean.io/blog/2014/01/installing-mean-io-on-heroku-from-a-to-z/) and buildback for deploying to Heroku, which made it super simple. I simply setup the mean.io app locally, spun up a Mongo instance on [Compose](http://compose.io) (a database hosting company), and deployed the base app to Heroku in under an hour.

### 2) Setting up Twilio

After getting the base app up and running I started to explore Twilio.

I'd never used their API before but it didn't seem terribly hard. I downloaded a sample Twilio app and played around with it to get a feel for it. Then I set to install their node.js library via npm to my existing mean.io app.

This is where the existing CRUD interface helped a lot. I simple took the existing framework of MEAN.io that was setup to create, edit, and view "Articles" and turned this into creating, editing, and viewing "Links"

Now a user can sign up, create a "Link" with the content of their SMS.

I then worked my way through setting up a system where when we receive an API hit with a Link ID and phone number, it sets up the corresponding text message to go out.

With Twilio this part took about a day.

I had to learn about purchasing phone numbers, cycling through them to make sure we don't hit daily rate limits per long code (5 digit short codes are $3k/3 months), and making sure I had all the right callbacks setup so I could determine the status of a message and react appropriately.

One lesson we learned here (and cost us a few hundred $$) was that Twilio on an SMS callback isn't guaranteed to send you the cost of that message. At first we assumed they did and passed on the returned value to the customer to pay for their SMS. Turns out in small volumes yes you always see the price, but at larger volumes Twilio may say the SMS is sent, but you have to send in another API call later to get the price. We found this after a couple weeks and losing some money.

After all of this we had the Twilio API setup and running and returning the saved SMS back to the phone number.

### 4) Building the drop in widget

This was (and still is) the trickiest part of the entire process.

To build the LinkTexting widget that people put on their website I first put together the basic HTML form that would accept a perfectly formatted phone number. 

Next I added [jackocnr's](http://http://jackocnr.com/) intl-tel-input widget hosted on github [check it out](https://github.com/Bluefieldscom/intl-tel-input). This powerful widget can detect a user's country, show the right flag and prepopulate the correct country code as well as validation and a great design. 

I then put together the javascript include that would read the widget and make the API calls back to our system. We wrote most of the SDK in pure javascript (no jQuery) except for the parts that called out to the intl-tel-input widget since that is written in jQuery. 

You can take a look at the javascript [here](http://d3q6uu7asevdsg.cloudfront.net/1.3/js/link_texting.js).

At this point we tested it on various domains. This is when I had to go back to the core platform and make some changes. Where user's could before just create a link with the content, they now had to add a list of the domains they were whitelisting and I added the logic to properly check the origin of the requests from the SDK to make sure we weren't having bad requests come in.

At this point it was ready to go!

While writing the widget was super easy, we ran into a ton of issues once customers had it. jQuery conflicts, css overwrites, the list went on. I wish I had a good guide to making a better SDK but honestly you have to let it run in the wild, see what breaks, and fix it fast. And for common errors throw up an FAQ as soon as possible to save you time on support emails.

With the basic product working it was time to start making sure customers could pay us :)

### 3) Installing Stripe

This was super easy, dropped in the stripe SDK, followed the documentation on their website, and had a payments modal up and running in a couple hours.

I used this pretty credit card form [here](https://github.com/jessepollak/card) and getting that setup took a bit of extra time. If you use Angular the below directive will be super useful with his plugin

    .directive('paymentForm', function() {
      return {
        link: function(scope, elm) {
            elm.card({
                // a selector or jQuery object for the container
                // where you want the card to appear
                container: '.card-wrapper', // *required*
                numberInput: 'input#number', // optional — default input[name="number"]
                expiryInput: 'input#expiry', // optional — default input[name="expiry"]
                cvcInput: 'input#cvc', // optional — default input[name="cvc"]
                nameInput: 'input#name', // optional - defaults input[name="name"]
    
                width: 325, // optional — default 350px
                formatting: true, // optional - default true
    
                // Default values for rendered fields - options
                values: {
                    number: '•••• •••• •••• ••••',
                    name: 'Full Name',
                    expiry: '••/••',
                    cvc: '•••'
                }
            });
        }
      };
    })

And we were ready to launch!

### 4) Launching!

Day one we launched and had 3 of our friends apps as paying customers. We were a $10/mo. unlimited plan at the time. 

We then were posted to Product Hunt by a beta tester of ours and it was off to the races!

### 5) Improvements since

Since we built it over that weekend we've done minor upgrades to the platform. 

We integrated with Branch.io so you can build a smart link that detects the device type right within LinkTexting and a bunch of stability fixes to make sure our customers are happy and they continue to get more downloads.

## Going Forward

We plan on running LinkTexting for as long as apps need to convert desktop visitors to app downloads.

Happy to answer any questions you may have about building LinkTexting :)

