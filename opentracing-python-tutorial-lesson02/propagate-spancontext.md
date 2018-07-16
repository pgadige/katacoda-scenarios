You might have noticed an unpleasant side effect of our recent changes,  
that is, passing a span object as the first parameter to the functions,  
`format_string` and `print_hello`.

Unlike Go, programming languages like Python support thread-local storage,  
which is a convenient way to store such request-scoped data like the current  
span. However, since the OpenTracing API for Python does not support a standard  
mechanism for accessing such thread-local storage, we will use an alternative  
API called `opentracing_instrumentation` that provides an in-memory context  
propagation facility suitable for multiple thread apps and Tornado apps via  
its `request_context` submodule.

We modify `say_hello` to store top level `root_span` in the context,

<pre class="file">
from opentracing_instrumentation.request_context import get_current_span, span_in_context

def say_hello(hello_to):
  with tracer.start_span('say-hello') as span:
    span.set_tag('hello-to', hello_to)
    with span_in_context(span):
      hello_str = format_string(hello_to)
      print_hello(hello_str)
</pre>

Accordingly, we modify `format_string` and `print_hello` function to retrieve the main `root_span` from the context.

<pre class="file">
def format_string(hello_to):
  root_span = get_current_span()
  with tracer.start_span('format', child_of=root_span) as span:
      hello_str = 'Hello, %s!' % hello_to
      span.log_kv({'event': 'string-format', 'value': hello_str})
      return hello_str

def print_hello(root_span, hello_str):
  root_span = get_current_span()
  with tracer.start_span('println', child_of=root_span) as span:
      print(hello_str)
      span.log_kv({'event': 'println'})
</pre>

Observe that we are no longer passing an instance of Span as an argument  
to the functions, `format_string` and `print_hello`, because they can now  
retrieve the `root_span` with the `get_current_span` method.

Let's copy the below code into the file `hello.py` (or use the `Copy to Editor` button):

<pre class="file" data-filename="exercise/hello.py" data-target="replace">
## hello.py
## simple hello world program

import sys
# modify path variable to search for lib.tracing module
sys.path.append("/root/opentracing-tutorial/python")
import time
from lib.tracing import init_tracer
from opentracing_instrumentation.request_context import get_current_span, span_in_context

def say_hello(hello_to):
  with tracer.start_span('say-hello') as span:
    span.set_tag('hello-to', hello_to)
    with span_in_context(span):
      hello_str = format_string(hello_to)
      print_hello(hello_str)

def format_string(hello_to):
  root_span = get_current_span()
  with tracer.start_span('format', child_of=root_span) as span:
      hello_str = 'Hello, %s!' % hello_to
      span.log_kv({'event': 'string-format', 'value': hello_str})
      return hello_str

def print_hello(hello_str):
  root_span = get_current_span()
  with tracer.start_span('println', child_of=root_span) as span:
      print(hello_str)
      span.log_kv({'event': 'println'})

assert len(sys.argv) == 2

tracer = init_tracer('hello-world')

hello_to = sys.argv[1]
say_hello(hello_to)

# yield to IOLoop to flush the spans
time.sleep(2)
tracer.close()
</pre>

Let's run the program and find out its trace in [Jaeger UI](https://[[HOST_SUBDOMAIN]]-16686-[[KATACODA_HOST]].environments.katacoda.com/search):    
`python2.7 -m hello Alice`{{execute}}

This particular change is helpful in larger programs with many functions  
calling each other. By using `request_context` mechanism, we can access  
the current span from any place in the program without passing the `Span`  
object as an argument to all the function calls.

*Note* :To see a trace in *Jaeger UI* dashboard, follow these steps:  
1. Click **service name** drop-down list, and select `hello-world`  
2. Click **operation name** drop-down list, and select `say-hello`,  
  `format` or `println`
