A trace is a directed acyclic graph of spans. A span is a logical  
representation of some work done in your application. Each span has  
these minimum attributes: an operation name, a start time, and a finish time.

Let's create an instance of a real tracer, such as [Jaeger](http://github.com/uber/jaeger-client-python).

A Configuration class is used to initialize the Jaeger Tracer so that  
we can easily parameterize the Tracer.

We define a function that returns the Tracer and call that function  
explicitly after all imports are done.

<pre class="file" data-target="clipboard">
## File:hello-world.py
## Initialize Jaeger Tracer
## using Jaeger Python client

import logging
from jaeger_client import Config

def init_jaeger_tracer(service):
  logging.getLogger('').handlers = []
  logging.basicConfig(format='%(message)s', level=logging.DEBUG)

  # Config class constructor
  config = Config(
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

## Initialize Jaeger Tracer
tracer = init_jaeger_tracer('hello-world')
</pre>

We create a `tracer` instance with the sevice name `hello-world`, which  
is passed as an argument to `init_jaeger_tracer` function.
Thus, all spans emitted from the tracer are marked as originating from   
the service `hello-world`.

We have now successfully initialized an instance of Jaeger tracer.
