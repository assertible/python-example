# API integration testing a Python Flask app on Travis CI

This repository contains an example app which
uses [Assertible](https://assertible.com) to run API integration tests
for a Python Flask app in a continuous integration build
using [Travis CI](https://travis-ci.org).

Assertible is web service testing tool for developers that focuses on
creating simple and deterministic tests combined with flexible
automation.

The setup is 4 steps:
- 1. [Create a Flask app](1.-create-a-flask-app)
- 2. [Create a Travis CI config](2.-create-a-travis-ci-config)
- 3. [Create an Assertible web service](3.-create-an-assertible-web-service)
- 4. [Copy the trigger URL to a Travis variable](3.-create-an-assertible-web-service)


# 1. Create a Flask app

Create a new directory for the code repository:

```
mkdir myapp && cd myapp
```

Save the following code to a file named `app.py`:

```python
from flask import Flask
app = Flask(__name__)

@app.route("/")
def hello():
    return "Hello Assertible!"

if __name__ == "__main__":
    app.run()
```

Save the code and push it to a GitHub repository:

```
git add app.py
git commit -a -m "Checkin app.py"
git push
```

_The code above assumes you have a GitHub repository setup for this
tutorial. If not, check here: https://docs.travis-ci.com/user/for-beginners_


# 2. Create a Travis CI config

This steps assumes you have created
a [Travis account](https://travis-ci.org/) and enabled the repo.

Save this Travis configuration to `.travis.yml` file:

```
language: python
python:
  - "2.7"

addons:
  apt:
    packages:
      - ca-certificates

install: "pip install Flask"

script:
  - echo "Unit tests"

after_script:
  - |
    python app.py &
    APP_PID=$!

  - |
    set -e

    curl -s https://bin.equinox.io/c/4VmDzA7iaHb/ngrok-stable-linux-amd64.zip > ngrok.zip
    unzip ngrok.zip

    # wget ngrok
    ./ngrok http 5000 > /dev/null &

    NGROK_URL=$(curl -s localhost:4040/api/tunnels/command_line | jq --raw-output .public_url)

    curl -s $TRIGGER_URL -d'{
      "environment": "my-dynamic-env",
      "url": "'"$NGROK_URL"'",
      "wait":true
    }'

  - kill $APP_PID
```

```
git add .travis.yml
git commit -a -m "Checkin app.py"
git push
```


# 3. Create the Assertible web service

Next, you need to create an Assertible account and create a new web
service. Simply [sign-in to Assertible](https://assertible.com/signup).

The first time you log-in, you will be prompted with a form to create
a web service:

<img
  src="https://s3-us-west-2.amazonaws.com/assertible/blog/assertible-new-service-go-heroku-example.png"
  alt="Configuring a web service in Assertible" />


# 4. Copy the trigger URL to a Travis variable

After creating a web service in the Assertible dashboard, navigate to
your web service's **Settings** tab and click **Trigger URL** in the
left side navigation. Copy the trigger URL to your clipboard.

In the Travis CI interface, navigate to your repository. On the left
hand side, click the **More options** then click **Settings**.

Add a new variables named `TRIGGER_URL` and paste the trigger URL into
the value input.

<img
  src="https://s3-us-west-2.amazonaws.com/assertible/blog/travis-ci-environment-variable-form.png"
  alt="Travis CI environment variable form" />

Finally, click **Restart build** or push some changes to your app to
initiate a build. Navigate back to the Assertible web service
dashboard and click **Results**. You should now see a result for your
API integration tests:

<img
  src="https://s3-us-west-2.amazonaws.com/assertible/blog/test-result-via-dynamic-trigger-url.png"
  alt="Assertible test results via dynamic trigger URL" />
