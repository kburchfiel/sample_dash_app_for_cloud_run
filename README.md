# How to Deploy a Simple Dash App to Google Cloud Run

Note: [Click here](https://youtu.be/64rih-vUgd4) to watch a video tutorial that accompanies these steps.

Deploying a Dash app to Cloud Run can potentially be cheaper than deploying it to Heroku. Heroku used to offer a free tier, but as of September 2023, their [pricing data](https://www.heroku.com/pricing) indicates that it will cost at least $5 a month to run an app there. The Cloud Run setup shown in this project, on the other hand, can cost only pennies per month (if that) depending on how frequently the app is used. (Heroku does have price caps for many of their hosting packages, so it could very well be the better option in case your Dash app is accessed frequently). 

(Many of the following steps are based on the Readme in my [Dash School Dashboard](https://github.com/kburchfiel/dash_school_dashboard) project.)

# Steps for Creating a Simple Dash App, Then Deploying It to Cloud Run:

1. You may wish to create a new virtual environment for this project, but it's not strictly necessary. For steps on doing so, visit the Readme within my [Dash School Dashboard](https://github.com/kburchfiel/dash_school_dashboard) project.

1. Copy the demo app code in the Dash tutorial's [Minimal App](https://dash.plotly.com/minimal-app) section into a new Python file called app.py, and place this file into a folder named 'sample_app' (or any name of your choice). 

1. Run this app locally to make sure it's working as expected. Steps for doing so can be found within Dash's [Minimal Dash App](https://dash.plotly.com/minimal-app) documentation page. 

    *Note: I received a 'Service Unavailable' message in my Cloud Run version of this app when trying to use a name other than app.py. Therefore, I recommend sticking to app.py as the program's name.*

1. Create a new Google Cloud project within your [Google Cloud Console](https://console.cloud.google.com/). I named my project 'sampleapp', but you can of course choose whichever name you'd like.

1. If you haven't already, set up the Google Cloud Command Line Interface (CLI) on your computer; this tool will let you run your Python project online via Google Cloud Run. (See ['Install the gcloud CLI'](https://cloud.google.com/sdk/docs/install) for more information.) 

    (Instead of using the bundled version of Python, I updated my Windows system PATH file with the Python environment I wanted to use. [This guide](https://leifengblog.net/blog/Installing-Google-Cloud-SDK-to-Use-Python-from-Anaconda/) explains how to accomplish this step.)

1. You'll next want to follow the steps shown in Google's [Deploy a Python service to Cloud Run](https://cloud.google.com/run/docs/quickstarts/build-and-deploy/deploy-python-service) article while making a couple changes along the way. These changes are based on the [Dash Heroku deployment guide](https://dash.plotly.com/deployment#heroku-for-sharing-public-dash-apps-for-free) and on Arturo Tagle Correa's [Deploying Dash to Google Cloud Run in 5 minutes](https://medium.com/kunder/deploying-dash-to-cloud-run-5-minutes-c026eeea46d4) guide. (Following the Google guide exactly won't work because it uses a Flask app as its example rather than a Dash app).

    One note before I go through these changes: The guide explains that you need to enter 'gcloud config set project PROJECT_ID' within the CLI in order to select your project, and that you should "Replace PROJECT_ID with the name of the project you created for this quickstart." However, make sure to replace Project ID with the project's ID, not its name, as the two won't always be the same! 
    
    (To find your project's ID, go to https://console.cloud.google.com/welcome and select your project within the top dropdown menu. You'll then be taken to a 'Welcome' page that shows your project's name (e.g. sampleapp) and its ID (e.g. sampleapp-398021). The ID will also be shown below the project name when you first creating your project.)


    Here are the changes you'll need to make to the steps shown in the Cloud Run guide:

    1. Instead of using the main.py file shown in the Google article, use the app.py example discussed earlier (e.g. the one [within the Dash Tutorial's Minimal App section](https://dash.plotly.com/minimal-app)). (Make sure to call it 'app.py' instead of 'main.py'.) Next, add  "server = app.server" under the line "app = Dash(\_\_name\_\_)", as this is seen in both Arturo's guide and Dash's Heroku deployment tutorial. 

    1. To create your requirements.txt file, simply type in:

        ```
        dash

        pandas

        gunicorn
        ```

    1. Change the final part of the Dockerfile example in the Google guide from "main:app" to "app:server". I made this change because (1) I was receiving error messages relating to gunicorn's inability to find 'main' and (2) [Arturo's guide](https://medium.com/kunder/deploying-dash-to-cloud-run-5-minutes-c026eeea46d4) showed app:server here as well. (Also note that the Heroku Procfile shown in the Heroku deployment guide also ends in app:server.) By the way, don't forget to capitalize 'Dockerfile' within your project directory!


1. Once you make these changes, you should be able to successfully deploy your app to Cloud Run. You can do so by opening your command prompt, navigating to the folder containing your app.py file, and entering:

    `gcloud run deploy --source .`

    (Note that the space and period after 'source' are part of the command and must be included. You may also need to enter gcloud auth login if you haven't logged in already.)

    For guidance on what to enter within the Cloud Console after this command, visit [this section](https://cloud.google.com/run/docs/quickstarts/build-and-deploy/deploy-python-service#deploy) of Google's Deploy a Python Service to Cloud Run guide. You'll likely be prompted to authorize a number of permissions at this point unless your project already has them enabled.

    When asked to provide a service name, you can simply hit Enter to choose the default option. ([Source](https://cloud.google.com/run/docs/quickstarts/build-and-deploy/deploy-python-service))

1. Ideally, after the CLI finishes processing your request, you'll see a message similar to the following:

    *"Service [dsd] revision [dsd-00003-qah] has been deployed and is serving 100 percent of traffic.
    Service URL: https://sample-dash-app-vtwzngx2pa-uc.a.run.app"*

1. Click on the service URL shown in the CLI message (which will be different from the above URL in your case). If everything went well, you should now see a copy of the current version of your app, now hosted online for anyone to view and interact with. However, you might instead see a black page with a 'Service Unavailable' message. This means that an error in my app's code or another file prevented Cloud Run from deploying your app.

1. When these errors arose, you can debug them using your project's log at https://console.cloud.google.com/logs/ . Sometimes, the most useful error messages were listed under the 'Default' severity category rather than the 'Error category,' so it can be helpful to look at all of the messages rather than just those labeled 'error'. Trying to run the app locally can also help, as an error that you encounter during local debugging can also be the explanation for an issue with the cloud-based version of the app.

1. In order to limit the cost of your project, I recommend deleting older versions of the app from Google's storage after each new deployment. You can do so by going to console.cloud.google.com, navigating to the Cloud Storage section, and deleting obsolete copies of the app. Similarly, you can visit https://console.cloud.google.com/artifacts/docker/ to delete obsolete Docker containers. (To identify any other services, such as Cloud Run, that are incurring charges, go to https://console.cloud.google.com/billing)