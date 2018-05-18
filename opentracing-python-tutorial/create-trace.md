We use the Jaeger tracer instance initialized in the previous step    
in our simple hello world program as follows:

<pre class="file" data-target="clipboard">
## File:hello-world.py

def say_hello(hello_to):
	## Create trace
	span = tracer.start_span('say-hello')
	hello_str = 'Hello, %s!' % hello_to
	print hello_str
	span.finish()
</pre>

We use the following basic features of the OpenTracing API:  
* a `tracer` instance is used to start new spans via `start_span` method
* each `span` is given an operation name, `say-hello` in this example
* each `span` must be finished by explicitly calling its `finish()` method
* start and finish timestamps of a span are automatically captured by   
  the tracer implementation, in this case, the Jaeger Tracer captures  
  the start and finish timestamps of each `span`

It's tedious to call `finish()` manually on a given `span`. Instead, we  
can use `span` as context manager as shown below:

<pre class="file" data-target="clipboard">
## File:hello-world.py

def say_hello(hello_to):
	## span as context manager
	with tracer.start_span('say-hello') as span:
		hello_str = 'Hello, %s!' % hello_to
		print hello_str
</pre>

Since the program is short-lived and it exits immediately, we can flush  
the span to Jaeger backend by adding the following to `hello-world.py`:

<pre class="file" data-target="clipboard">
## File:hello-world.py
## Initialize Jaeger Tracer

import time

def say_hello(hello_to):
	## span as context manager
	## Create trace
	with tracer.start_span('say-hello') as span:
		hello_str = 'Hello, %s!' % hello_to
		print hello_str

## Flush span to Jaeger backend
time.sleep(2)
tracer.close()
</pre>

We instrument the simple hello world program in `hello.py`. Copy the content  
below into the file (or click `Copy to Editor` button):

<pre class="file" data-filename="exercise/hello.py" data-target="replace">
## a simple hello world program
## to demonstrate instrumenting a
## simple Python program using OpenTracing API
## and JaegerTracing system

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
        hello_str = "Hello, %s!" % hello_to
        print hello_str

assert len(sys.argv) == 2

hello_to = sys.argv[1]
say_hello(hello_to)

time.sleep(20)
tracer.close()
</pre>

We run the above program with command line argument, `Alice`, and should  
be able to see a span logged:  
`python2.7 hello-world.py Alice`{{execute}}

A visual representation of the trace in the form of a timing diagram is  
available through [Jaeger UI](https://[[HOST_SUBDOMAIN]]-16686-[[KATACODA_HOST]].environments.katacoda.com/search?service=hello-world).  
We select the service name as `hello-world` and the operation name as  
`say-hello` for accessing the corresponding timing diagram.
