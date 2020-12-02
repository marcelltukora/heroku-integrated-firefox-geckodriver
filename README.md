# heroku-integrated-firefox-geckodriver



Buildpack `heroku-integrated-firefox-geckodriver` enables your application or client code - running in any high-level language such as *Python, Ruby or Node.js* - to access **Firefox** along with **Geckodriver** (the Selenium driver for Firefox) in a Heroku slug and enables the driver objects to perform automated operations defined in the source code.

Installation:
-----

To install and integrate the buildpack with your application running on Heroku's dyno:

```shell
$ heroku create --buildpack https://github.com/digitalcircuits/heroku-integrated-firefox-geckodriver

# or if your app is already created:
$ heroku buildpacks:add https://github.com/digitalcircuits/heroku-integrated-firefox-geckodriver

$ git push heroku master
```

Configurations:
-----

Update Heroku's environment variables to store the following path strings. 
                                
  
**FIREFOX_BIN**: */app/vendor/firefox/firefox*

**GECKODRIVER_PATH**: */app/vendor/geckodriver/geckodriver*

**LD_LIBRARY_PATH**: */usr/local/lib:/usr/lib:/lib:/app/vendor*

**PATH**: */usr/local/bin:/usr/bin:/bin:/app/vendor/*

                

These configuration vars can be updated via Heroku CLI as follows:

Executable command: `heroku config:set <ENV_VARIABLE>=<ABSOLUTE_PATH>`

```shell

$ heroku config:set FIREFOX_BIN=/app/vendor/firefox/firefox

Setting FIREFOX_BIN and restarting python-app... done, v6
FIREFOX_BIN: '/app/vendor/firefox/firefox'

```


How To Use - Example Script (Python)
---
For this example, I will be using Python. Here is an example script you can deploy:

```

from flask import Flask, jsonify
from selenium import webdriver
from selenium.webdriver.firefox.firefox_binary import FirefoxBinary #We import this so we can specify the Firefox browser binary location
import os

import os
app = Flask(__name__)
PORT = os.environ.get('PORT') or 3092
DEBUG = False if 'DYNO' in os.environ else True

FF_options = webdriver.FirefoxOptions()
FF_profile = webdriver.FirefoxProfile()
FF_options.add_argument("-headless")
FF_profile.update_preferences()


@app.route("/get")
def getcurrIp():
    try:
        #Notice in the driver, we specify the executable path to the geckodriver
        driver = webdriver.Firefox(options=FF_options, firefox_profile=FF_profile, executable_path=os.environ.get("GECKODRIVER_PATH"), firefox_binary=FirefoxBinary(os.environ.get("FIREFOX_BIN")))

        driver.get("https://icanhazip.com")

        ipaddr = driver.page_source

        driver.close()
        
        return jsonify({'status': 1, 'ip': ipaddr})
    except Exception as e:
        return jsonify({'status': 0, 'error': str(e)}), 500

@app.route('/')
def hello_world():
    return 'Hello, World!'

if __name__ == "__main__":
    app.run(port=PORT, debug=DEBUG)

```
