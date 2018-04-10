---
title:  Getting Started (2)
layout: default
root: true
permalink: docs/getting-started-new
collection: "Articles2"
--- 

## Getting Started

In this tutorial, we will walk you through the process of adding Extend to an existing, if simple, application. Our application is called ZeroCRM and is a basic CRM application.

![Alt text](/docs/docs/assets/img/zerocrm1.png )

### Testing the Initial Version

To get started, first clone the repository here: https://github.com/goextend/zerocrmv2. The initial application may be found in the `before` subdirectory. This sample application is written in Node.js, so if you haven't installed Node, you'll want to do that first. As this guide will also walk you through the entire process, you are welcome to just read and skip working with the code. 

Once cloned, be sure to run `npm i` to get all the dependencies. (Again, remember to start in the `before` directory.) You can then start the application by using `npm run start`. You will be dropped immediately in the "Add New Lead" interface. Fill in anything you want here and click the "Create Lead" button. This will fire off an Ajax request back to the server and then render the response:

![Alt text](/docs/docs/assets/img/zerocrm2.png)

In the response, you'll notice that a "Created" value has been added. This was done by the server simply to demonstrate that, typically, the server would do something with the data, like persist it. 

There's one more feature of this application and you can find it under the Settings page. The application wanted to allow users basic customization and it did this through a pretty common method - webhooks.

![Alt text](/docs/docs/assets/img/zerocrm3.png)

Webhooks work by asking the end user to set up their own server, or serverless function, and respond to a call from the application. In this case, the application will be sending data about the lead and the webhook is responsible for doing something with the data and returning a result. To help illustrate this, you'll see a default webhook URL provided. (And it's actually on the same server, which normally wouldn't make sense!) In this case, it will run a basic route that checks the value of the lead:

```js
app.post('/webhook', (req, res) => {
	let lead = req.body;

	if(lead.value > 1000) lead.vip = true;
	res.json(lead);
});
```

The customization basically adds a VIP flag for high value leads. You can see this yourself if you go back to to the Lead form and add a new one with a high value:

![Alt text](/docs/docs/assets/img/zerocrm4.png)

Here's the route that handles accepting new leads:

```js
app.post('/api/leads',  (req, res) => {
	let data = req.body;
	// this is where - normally - a process of somesort would persist the value
	// data.created is simply demonstrating a server-side change
	data.created = new Date();

	if(webhookUrl != '') {

		let options = {
			method:'POST',
			url:webhookUrl,
			json:data
		};

		request(options, (error, response, body) => {
			if(error) throw new Error(error);
			res.json(body);
		});

	} else {
		res.json(data);
	}
})
```		

`webhookUrl` is a global variable just persisted in RAM. If you wish, you can actually build a real webhook, replace the URL, and try it out yourself. 

### Building the Extend Version

Now we're going to build the Extend version of the application. If you want to skip ahead to the end, you can find the final version of the application in the `after` subdirectory. Don't forget to run `npm i` inside the folder to add the necessary dependencies.

To begin working with Extend, you'll first need to create a developer account. Point your browser at https://auth0.com/extend/try and you'll be prompted to login:

![Alt text](/docs/docs/assets/img/zerocrm5.png)

After you've logged in, you'll be presented a page with your keys for testing Extend:

![Alt text](/docs/docs/assets/img/zerocrm6.png)

For now, you only need to make note of two values - the container and token. Keep them handy as we'll be using them in a moment. 

Alright, so how do we add Extend to the application? 

The first decision you make is figuring out your "extension points", or where you will allow for customizations. Luckily we've already done that. We've got a settings that with a webhook that customizes the "New Lead" process.

Once we've figured that out, we need to make two changes to enable Extend for this customization. First, we have to add the editor. This is a hosted editor your users will work with to write their customization. The second change is to make use of their end point when a new lead is created. Our back end code is already making a network request to a webhook so this won't be a big change. Let's get started.

<strong>Setting up the Keys</strong>

Let's begin by getting our keys recognized in the application. We're making use of the [dotenv](https://www.npmjs.com/package/dotenv) package which makes storing and using "secret" values like our keys easy. Start by adding a new file to the root of the application called `.env`. 

	AUTH0_CONTAINER=the container key
	AUTH0_TOKEN=the token

Obviously you will want to replace the values above with the keys you got after logging in.

We're going to use these keys when displaying the editor and when invoking the extension, so we'll load it in the `routes/index.js` file. Open it up and add this to the top:

```js
require('dotenv').config();
```

This will read in the `.env` file you created and set the values in the `process.env` scope. Let's use them now:

```js
const auth0Container = process.env.AUTH0_CONTAINER;
const auth0ExtendURL = `https://${auth0Container}.run.webtask.io/`;
const auth0Token = process.env.AUTH0_TOKEN;
```

This code sets up two local variables for each key and defines the extension URL based on the container value. 

<strong>Adding the Editor</strong>

Now let's add the Extend editor. While still in the `routes/index.js` file, scroll down to the `/settings` route and modify it like so:

```js
app.get('/settings', (req, res) => {
	res.render('settings', { 
		nav:'settings',
		container:auth0Container,
		token:auth0Token
	});
});
```

All we're doing here is passing the values from our keys to the view. Let's see how these are used. Open up `views/settings.handlebars`. Modify it so it looks like the code below:

```html
<h1>Settings</h1>
<h2>Add New Lead</h2>

<div id="extend-editor" style="height: 600px; width: 100%"></div>
<script>
ExtendEditor.create(document.getElementById('extend-editor'), {
	hostUrl: 'https://webtask.it.auth0.com',
	webtaskContainer: '{{container}}',
	token: '{{token}}',
	webtaskName: 'saveLead',
	createIfNotExists: true
});
</script>
```

Adding the Extend editor is incredible simple. You add a div that will be replaced by the editor and then drop in the code. There is an [incredible amount](https://auth0.com/extend/docs/libraries/extend-editor#integration-options) of customizations you can do to the editor but the defaults are fine for us. There are two things to take note of here. First, the `webtaskName` value gives a unique name to the extension. While you can name this whatever you want, it makes sense to give it a name that represents the customization or extension it is working with. Secondly, note that the keys passed to the route are used in the code so that the editor can act on your behalf. 

Before we use the editor, we need to load the JavaScript file that let's the `ExtendEditor` API work. In `views/layouts/main.handlebars`, add this script tag in the head block:

```html
<script src="https://cdn.auth0.com/auth0-extend/1/extend-editor.js"></script>
```

At this point, you should be able to test editor. Run `npm run start` and go to the settings page.

![Alt text](/docs/docs/assets/img/zerocrm7.png)

Look at that beautiful editor. You've got basically a miniature IDE right within the browser. Of course, you don't have to use this. You can provide your users a CLI path to building their extensions as well and they can use desktop editors, but this is certainly incredibly convenient. They don't have to create a file, setup a server, or anything else. They can immediately begin writing code to add their customization.

The default code in the editor (which can be customized as well!) is a basic "hello world", so let's modify it to mimic the functionality from the previous webhook. 

```js
module.exports = function(context, cb) {
	var lead = context.body;
	if(lead.value > 1000) lead.vip = true;
	
	cb(null, { lead:lead });
};
```

The value of the lead will exist in the `context.body` property so we make a quick copy just to make the code a bit more readable. We then do our logic (and feel free to modify this to your liking) and then return the lead with the callback provided in the `cb` argument. 

Once done, be sure to click the floppy save icon to persist your change. (Or use CTRL/CMD+S like any proper editor!) 

<strong>Calling the Extension</strong>

The next change is even simpler. The application was already calling out to a webhook to run the user's customization. Now we will change the code to hit the Extend endpoint for their code instead. Open up `routes/index.js` again and replace the `/api/leads` with this updated version:

```js
app.post('/api/leads',  (req, res) => {
	let data = req.body;
	// this is where - normally - a process of somesort would persist the value
	// data.created is simply demonstrating a server-side change
	data.created = new Date();

	let options = {
		method:'POST',
		url:auth0ExtendURL +'saveLead',
		headers:{'Authorization':`Bearer ${auth0Token}`},
		json:data
	};

	request(options, (error, response, body) => {
		if(error) throw new Error(error);
		res.json(body);
	});

});
```

Basically all we've done here is change the URL and add authorization information to the header. The URL is based on the value created earlier (which itself was based on your keys) and adds the name of the extension. This needs to match the `webtaskName` value used in the editor. Note that the lead value in `data` is posted to the endpoint as JSON data. 

Save your work, restart the server, and test out your change by creating a new lead.

![Alt text](/docs/docs/assets/img/zerocrm8.png)
