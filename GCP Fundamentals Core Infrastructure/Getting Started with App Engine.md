# Google Cloud Fundamentals: Getting Started with App Engine

## Objectives

- In this lab, We learn how to perform the following tasks:
  - Initialize App Engine.
  - Preview an App Engine application running locally in Cloud Shell.
  - Deploy an App Engine application, so that others can reach it.
  - Disable an App Engine application, when you no longer want it to be visible.

## Initialize App Engine

Initialize your App Engine app with your project and choose its region:

```
gcloud app create --project=$DEVSHELL_PROJECT_ID
```

Clone the source code repository for a sample application in the hello_world directory:

```

git clone https://github.com/GoogleCloudPlatform/python-docs-samples

```

Navigate to the source directory:

```
cd python-docs-samples/appengine/standard_python3/hello_world
```

## Run Hello World application locally

In this task, you run the Hello World application in a local, virtual environment in Cloud Shell.

Ensure that you are at the Cloud Shell command prompt.

Execute the following command to download and update the packages list.

```
sudo apt-get update
```

Set up a virtual environment in which you will run your application. Python virtual environments are used to isolate package installations from the system.

```
sudo apt-get install virtualenv
```

```
virtualenv -p python3 venv
```

Activate the virtual environment.

```
source venv/bin/activate
```

Navigate to your project directory and install dependencies.

```
pip install  -r requirements.txt
```

Run the application:

```
python main.py
```

In Cloud Shell, click Web preview (Web Preview) > Preview on port 8080 to preview the application.
To access the Web preview icon, you may need to collapse the Navigation menu.

Result:

![Hello World application in a local](./images/hello-world-locally.png)

To end the test, return to Cloud Shell and press Ctrl+C to abort the deployed service.

## Deploy and run Hello World on App Engine

To deploy your application to the App Engine Standard environment:

Navigate to the source directory:

```
cd ~/python-docs-samples/appengine/standard_python3/hello_world
```

Deploy your Hello World application.

```
gcloud app deploy
```

This app deploy command uses the app.yaml file to identify project configuration.

Launch your browser to view the app at http://YOUR_PROJECT_ID.appspot.com

```
gcloud app browse
```

Copy and paste the URL into a new browser window.

Result:

![Hello World application in a local](./images/hello-world-app-engine.png)

Congratulations! You created your first application using App Engine.

## Disable the application

App Engine offers no option to Undeploy an application. After an application is deployed, it remains deployed, although you could instead replace the application with a simple page that says something like "not in service."

However, you can disable the application, which causes it to no longer be accessible to users.

In the Cloud Console, on the Navigation menu (Navigation menu), click App Engine > Settings.

Click Disable application.

Read the dialog message. Enter the App ID and click DISABLE.

![Hello World application in a local](./images/hello-world-app-engine-disable.png)
