---
layout: post
title: LightSwitch to Android via Azure, OData, Basic Authentication, SSL, and odata4j
author: Jami Moubry
date: 2013-05-06 21:13
comments: true
categories:
---

Prior to this experiment, I was developing an ASP.NET MVC app hosted in Azure to serve as a REST API for feeding data to my Android app.  I was really not into ASP.NET MVC, so when I learned that LightSwitch 2012 apps produce OData services, I jumped at the chance to try to replace my "homemade" REST API with OData.

Assumptions:

- You have an active Azure subscription and SQL database hosted in Azure with Firewall rules set to where you can access the database from your local machine
- You have Visual Studio 2012
- You know how to create a LightSwitch project
- You have a basic Android app (a Hello World one will do)

Here are the steps I took to complete this proof-of-concept.

1. Create a new LightSwitch project in Visual Studio 2012.  Attach to an external data source -- your Azure database.  Create some screens.

2. On the properties of your LightSwitch project, set the [Access Control](/images/LSAccessControl.png) to use Forms Authentication.

3. Publish the app with the following [settings](/images/LSPublishSettings.png):

		Application Type: Web
		Application Server Configuration: Windows Azure
		Subscription: Download and then import your credentials file.
		Service Configuration - Common: choose or create a new cloud service
		Service Configuration - Advanced:  set the Deployment Name to the name you want for the OData service, such as "MyAppData".
		Security Settings - Application Adminstrator: Create an application administrator and provide the credentials.
		Security Settings - HTTPS: Create a new self-signed certificate.
		Data Connections: Specify the user and admin connection strings.

4. Test the LightSwitch application in IE (the LightSwitch app won't run in Chrome -- hopefully this will be fixed in the final version of VS 2012). You can find the URL to the application in your Azure Management Portal under Hosted Services.  Select the item with the service name you provided and use the URL listed in the DNS name property. For staging, the URL will be something like `http://123abc.cloudapp.net`.

5. Test the OData service by appending the name you set for the service: `http://123abc.cloudapp.net/MyAppData.svc`.  You will get a certificate error since you self-signed the certificate, but that's okay (you can trust yourself).  This should list the names of your entities in the form of an atom feed.  You can also try accessing `http://123abc.cloudapp.net/MyAppData.svc/Movies` where 'Movies' is the name of one of your entity sets.

6. Open AndroidManifest.xml in your Android app and make sure you declare the internet permission.

	`<uses-permission android:name="android.permission.INTERNET" />`

7. Locate a recent build of the [odata4j project](http://code.google.com/p/odata4j/) (the current stable version `odata4j-archive-0.6.zip` does not support LightSwitch OData). I used the [build](https://odata4j.ci.cloudbees.com/job/odata4j-ci/1062/org.odata4j$odata4j-dist/) from 8/6/2012 9:29:04 PM. Download `odata4j-dist-0.7.0-SNAPSHOT-clientbundle.jar` and add it to your Android project.

8. Use the following code to connect to the OData service in one of your Android activities and get a list of entities:

		ODataConsumer c = ODataConsumers.newBuilder("https://123abc.cloudapp.net/MyAppData.svc/")
							.setClientBehaviors
							(
								OClientBehaviors.basicAuth("MyUsername", "MyPassword"),
								AllowSelfSignedCertsBehavior.allowSelfSignedCerts()
							).build();

		List<OEntity> lstMovieEntities = c.getEntities("Movies").execute().toList();

9. Then, you can grab the entity properties using the following code:

		List<MyMovie> movies = new ArrayList<MyMovie>();

		for (OEntity movie : lstMovieEntities) {
			MyMovie m = new MyMovie();

			m.Title = movie.getProperty("Title").getValue().toString();
			m.Year = movie.getProperty("Year").getValue().toString();

			movies.add(m);
		}


*Note: Azure SQL Tables can produce an OData service on their own, but I imagine setting up basic authentication for various users would be more difficult.*