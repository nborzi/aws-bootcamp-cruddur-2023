# Week 2 — Distributed Tracing

Login to Honeycomb and create an environment adn set up API keys that set up the environments get the api keys and add to the env in gitpod

```sh
export HONEYCOMB_API_KEY=""
export HONEYCOMB_SERVICE_NAME="Cruddur"
gp env HONEYCOMB_API_KEY=""
gp env HONEYCOMB_SERVICE_NAME="Cruddur"
```

Add teh following Env Vars to `backend-flask` in docker compose to edit the OTEL OpenTelemetry 

```yml
OTEL_EXPORTER_OTLP_ENDPOINT: "https://api.honeycomb.io"
OTEL_EXPORTER_OTLP_HEADERS: "x-honeycomb-team=${HONEYCOMB_API_KEY}"
OTEL_SERVICE_NAME: 'backend-flask'
```

![image](https://user-images.githubusercontent.com/86881008/223443608-4493b5e0-41a0-43f9-bcf7-1932352f54fc.png)


in backend-flask add the following to the requirements.txt and app.py - then run the pip install requirements 


```
opentelemetry-api 
opentelemetry-sdk 
opentelemetry-exporter-otlp-proto-http 
opentelemetry-instrumentation-flask 
opentelemetry-instrumentation-requests
```

We'll install these dependencies:

```sh
pip install -r requirements.txt
```

Add to the `app.py`

```py
from opentelemetry import trace
from opentelemetry.instrumentation.flask import FlaskInstrumentor
from opentelemetry.instrumentation.requests import RequestsInstrumentor
from opentelemetry.exporter.otlp.proto.http.trace_exporter import OTLPSpanExporter
from opentelemetry.sdk.trace import TracerProvider
from opentelemetry.sdk.trace.export import BatchSpanProcessor
```
### Initialize tracing and an exporter that can send data to Honeycomb
provider = TracerProvider()
processor = BatchSpanProcessor(OTLPSpanExporter())
provider.add_span_processor(processor)
trace.set_tracer_provider(provider)
tracer = trace.get_tracer(__name__)

app = Flask(__name__)
### Initialize automatic instrumentation with Flask
FlaskInstrumentor().instrument_app(app)
RequestsInstrumentor().instrument()

```py
![image](https://user-images.githubusercontent.com/86881008/223447849-560e426c-eaa9-49a5-bd7e-aa63de35e9b3.png)

Remember to commit the changes 'instrument honeycomb

go to the frontend-react-js and install npm again (as we are using the same docker compose file within branches - in dev/prod you shouldnt do that)
```sh
npm i
```
go to docker compose and put it up 

# Note
### changes to gitpod to automate the ports opening when the docker/service is up and running 
ports:
  - name: frontend
    port: 3000
    onOpen: open-browser
    visibility: public
  - name: backend
    port: 4567
    visibility: public
  - name: xray-daemon
    port: 2000
    visibility: public
    
   ------------------------------------------------------------
Add the following to the app.py code so we can troubleshoot logs - it should appear in the api in honeycomb
![image](https://user-images.githubusercontent.com/86881008/223454642-3eb78b0d-8c3f-4a21-b8a8-8c7275881981.png)

IF YOU ARE NOT RECEIVING DATA IN HONEYCOMB WEBSITE - then go to the link below and use your HONEYCOMB_API_KEY

https://honeycomb-whoami.glitch.me/ 

# doc 
https://docs.honeycomb.io/getting-data-in/opentelemetry/python/

# Create a span home activities
Go to backend-flask > services > home_activities.py


```
from opentelemetry import trace

tracer = trace.get_tracer(__name__)
with tracer.start_as_current_span("http-handler"):
    with tracer.start_as_current_span("my-cool-function"):
```

![image](https://user-images.githubusercontent.com/86881008/223485734-c648e570-40bb-4dc3-a1ff-c49bc9fe465e.png)

```
from datetime import datetime, timedelta, timezone
from opentelemetry import trace

tracer = trace.get_tracer("home.activities")

class HomeActivities:
  def run():
    with tracer.start_as_current_span("home-activites-mock-data"):
      span = trace.get_current_span()
      now = datetime.now(timezone.utc).astimezone()
      span.set_attribute("app.now", now.isoformat())
      results = [{
```

at the end (just line above results add the app result length 

```
      ]
      span.set_attribute("app.result_length", len(results))
      return results
```


# AWS x-ray 

go to the requirements.txt file under backend and add the following line 

```
aws-xray-sdk
```

in ssh now cd to backend-flask and run the requirements file 

```
pip install -r requirements.txt 
```

go to aap.py and add the content 


```
from aws_xray_sdk.core import xray_recorder
from aws_xray_sdk.ext.flask.middleware import XRayMiddleware

xray_url = os.getenv("AWS_XRAY_URL")
xray_recorder.configure(service='backend-flask', dynamic_naming=xray_url)

#this needs to be added under app definition
XRayMiddleware(app, xray_recorder)

```
![image](https://user-images.githubusercontent.com/86881008/223678376-53da1975-9e6e-4978-8b0f-beaa0090179e.png)


### Setup AWS X-Ray Resources 

Create a aws/json folder to the project and add a xray.json file 
```
{
    "SamplingRule": {
        "RuleName": "Cruddur",
        "ResourceARN": "*",
        "Priority": 9000,
        "FixedRate": 0.1,
        "ReservoirSize": 5,
        "ServiceName": "backend-flask",
        "ServiceType": "*",
        "Host": "*",
        "HTTPMethod": "*",
        "URLPath": "*",
        "Version": 1
    }
  }
```
 in ssh create a new group in x-ray (note aws region must be set you can check with aws configure and set up for eu-west-2, you can check if it is created in the console by accessing https://eu-west-2.console.aws.amazon.com/cloudwatch/home?region=eu-west-2#xray:settings/groups  
 ```sh
FLASK_ADDRESS="https://4567-${GITPOD_WORKSPACE_ID}.${GITPOD_WORKSPACE_CLUSTER_HOST}"
aws xray create-group    --group-name "Cruddur"    --filter-expression "service(\"backend-flask"\)"
```
go to the link to access the sampling rules https://eu-west-2.console.aws.amazon.com/cloudwatch/home?region=eu-west-2#xray:settings/sampling-rules
```sh
aws xray create-sampling-rule --cli-input-json file://aws/json/xray.json
```

### Add Deamon Service to Docker Compose

go to the docker-compose file and add the below 

```yml
  xray-daemon:
    image: "amazon/aws-xray-daemon"
    environment:
      AWS_ACCESS_KEY_ID: "${AWS_ACCESS_KEY_ID}"
      AWS_SECRET_ACCESS_KEY: "${AWS_SECRET_ACCESS_KEY}"
      AWS_REGION: "eu-west-2"
    command:
      - "xray -o -b xray-daemon:2000"
    ports:
      - 2000:2000/udp
```

We need to add these two env vars to our backend-flask in our `docker-compose.yml` file

```yml
      AWS_XRAY_URL: "*4567-${GITPOD_WORKSPACE_ID}.${GITPOD_WORKSPACE_CLUSTER_HOST}*"
      AWS_XRAY_DAEMON_ADDRESS: "xray-daemon:2000"
```
Go to this link to see the data https://eu-west-2.console.aws.amazon.com/cloudwatch/home?region=eu-west-2#xray:traces/query 

![image](https://user-images.githubusercontent.com/86881008/223743953-b06e8ae0-2bf8-4e22-a7cb-d7f374493588.png)

#adding Subsegments to X-Ray 
This goes beyond my knowledge in it - so i literally followed along 

![image](https://user-images.githubusercontent.com/86881008/223755968-bbd57942-b6a4-4381-b814-880ce2928db0.png)

aim is to get data to appear in X-Ray for activities that happens from user '@andrewbrown' and home page

in app.py add these 2 rows under user and home activities in their appropriated segments 
```
@xray_recorder.capture('activities_users')
@xray_recorder.capture('activities_show')
```

![image](https://user-images.githubusercontent.com/86881008/223756718-68059fd1-4538-49ca-842e-e503e4054926.png)


in backend-flask> services> user_activities.py add the following to the main 
```py
from aws_xray_sdk.core import xray_recorder
```

after model data results then add the segment 

```py 
      subsegment = xray_recorder.begin_subsegment('mock-data')
      # xray ---
      dict = {
        "now": now.isoformat(),
        "results-size": len(model['data'])
      }
      subsegment.put_metadata('key', dict, 'namespace')
      xray_recorder.end_subsegment()
    finally:  
    #  # Close the segment
      xray_recorder.end_subsegment()
```

## CloudWatch Logs



Add to the `requirements.txt`

```
watchtower
```

```sh
pip install -r requirements.txt
```


In `app.py`

```
import watchtower
import logging
from time import strftime
```

```py
# Configuring Logger to Use CloudWatch
LOGGER = logging.getLogger(__name__)
LOGGER.setLevel(logging.DEBUG)
console_handler = logging.StreamHandler()
cw_handler = watchtower.CloudWatchLogHandler(log_group='cruddur')
LOGGER.addHandler(console_handler)
LOGGER.addHandler(cw_handler)
```

```py
@app.after_request
def after_request(response):
    timestamp = strftime('[%Y-%b-%d %H:%M]')
    LOGGER.error('%s %s %s %s %s %s', timestamp, request.remote_addr, request.method, request.scheme, request.full_path, response.status)
    return response
```

MAKE SURE YOU EDIT THE LOGGER IN HOME_ACTIVITIES AND APP.PY
![image](https://user-images.githubusercontent.com/86881008/223775610-436ac9c1-dc99-45ff-878b-77e3e4da23d8.png)



We'll log something in an API endpoint
```py
LOGGER.info('Hello Cloudwatch! from  /api/activities/home')
```



Set the env var in your backend-flask for `docker-compose.yml`

```yml
      AWS_DEFAULT_REGION: "${AWS_DEFAULT_REGION}"
      AWS_ACCESS_KEY_ID: "${AWS_ACCESS_KEY_ID}"
      AWS_SECRET_ACCESS_KEY: "${AWS_SECRET_ACCESS_KEY}"
```

https://eu-west-2.console.aws.amazon.com/cloudwatch/home?region=eu-west-2#logsV2:log-groups/log-group/cruddur
![image](https://user-images.githubusercontent.com/86881008/223775208-184f50b8-2d3b-487e-bfbc-0c7109222ac1.png)

