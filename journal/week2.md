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
