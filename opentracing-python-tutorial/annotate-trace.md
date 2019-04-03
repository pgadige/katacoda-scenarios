What if we call our program with a different command line argument,  
say `Bob` instead of `Alice`? Both the resulting traces will be identical.  
We can capture command line arguments in the traces to distinguish them  
by annotating traces with tags and logs.

However, we cannot use a span's operation name to distinguish traces,  
because the operation name represents a class of spans, rather than a  
unique span. Moreover, a general operation name allows a tracing system  
to do aggregations.

A *tag* is a key:value pair that provides certain metadata about the span.  

A *log* is similar to a regular log statement, it contains a timestamp and  
some data, but it is associated with the span from which it is logged.

If a span represents an HTTP request, the URL of the request should be  
made a tag because the URL is relevant for the entire duration of the  
span. On the other hand, if the server responds with an indirect URL,  
logging it would make more sense since there is a clear timestamp  
associated with such event.

### Using Tags
Consider `Hello, Bob`. The string `Bob` is a good candidate for a span  
tag, since it applies to the whole span and not to a particular moment  
in time.

<pre class="file">
with tracer.start_span('say-hello') as span:
  span.set_tag('hello_to', hello_to)
</pre>

### Using Logs
The two operations in our hello world program - formatting `hello_str`,  
and then printing it - take certain time, so we can log their completion.

<pre class="file">
# log string formatting
hello_str = 'Hello, %s!' % hello_to
span.log_kv({'event':'string-format', 'value':hello_str})

# log string printing
print(hello_str)
span.log_kv({'event':'println'})
</pre>

The OpenTracing API for Python exposes structured logging API method `log_kv`  
that takes a dictionary of key:value pairs.

The OpenTracing Specification also recommends all log statements to always  
contain an `event` field that describes the overall event being logged, with  
other attributes of the event provided as additional fields.

The complete instrumented code is shown below (copy it into `hello.py` by  
clicking on `Copy to Editor` button) :

<pre class="file" data-filename="exercise/hello.py" data-target="replace">
# a simple hello world program
# to demonstrate annotating a
# trace with tags and logs
# using JaegerTracing system

import sys
import time
import logging
from jaeger_client import Config

def init_jaeger_tracer(service):
    logging.getLogger('').handlers=[]
    logging.basicConfig(format='%(message)s', level=logging.DEBUG)

    config=Config(
        config={
            'sampler':{
                'type':'const',
                'param':1
            },
            'logging':True
        },
        service_name=service
    )

    return config.initialize_tracer()

tracer = init_jaeger_tracer('hello-world')

def say_hello(hello_to):
    with tracer.start_span('say-hello') as span:
        span.set_tag('hello_to', hello_to)
        hello_str = "Hello, %s!" % hello_to
        span.log_kv({'event':'string-format', 'value':hello_to})

        print hello_str
        span.log_kv({'event':'printlin'})

assert len(sys.argv) == 2

hello_to = sys.argv[1]
say_hello(hello_to)

time.sleep(20)
tracer.close()
</pre>

Let's run the above program with `Bryan` as the command line argument:  
`python2.7 hello.py Bryan`{{execute}}

A visual representation of the trace in the form of a timing diagram is  
available through [Jaeger UI](https://[[HOST_SUBDOMAIN]]-16686-[[KATACODA_HOST]].environments.katacoda.com/search?service=hello-world&operation=say-hello).    
To view the corresponding timing diagram, follow these steps:  
1. Click the **service name** drop-down list, select `hello-world`
2. Click the **operation name** drop-down list, select `say-hello`
