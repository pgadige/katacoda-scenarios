Let's begin from where we left in lesson01. Let's rewrite our simple hello world  
program from lesson01. Click the link below to open  
`hello.py` in the editor:  
`exercise/hello.py`{{open}}

Once the file is opened in the editor, you can then copy the content  
below into the file (or use the `Copy to Editor` button):

<pre class="file" data-filename="exercise/hello.py" data-target="replace">
## hello.py
## simple hello world program

import sys
# modify path variable to search for lib.tracing module
sys.path.append("/root/opentracing-tutorial/python")
import time
from lib.tracing import init_tracer


def say_hello(hello_to):
    with tracer.start_span('say-hello') as span:
        span.set_tag('hello-to', hello_to)

        hello_str = 'Hello, %s!' % hello_to
        span.log_kv({'event': 'string-format', 'value': hello_str})

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

*Note* :Observe that `init_tracer` function is moved to a separate library directory.

Let's move the two operations - formatting and printing a string - to  
standalone functions and wrap each function into its own span. We observe  
that the functions `format_string` and `print_hello` are introduced in  
the program, respectively, to format and print strings.

We copy the code below into `hello.py` file (or use the `Copy to Editor`  
button):

<pre class="file" data-filename="exercise/hello.py" data-target="replace">
## hello.py
## simple hello world program

import sys
# modify path variable to search for lib.tracing module
sys.path.append("/root/opentracing-tutorial/python")
import time
from lib.tracing import init_tracer

def say_hello(hello_to):
  with tracer.start_span('say-hello') as span:
    span.set_tag('hello-to', hello_to)
    hello_str = format_string(span, hello_to)
    print_hello(span, hello_str)

# format given string
def format_string(root_span, hello_to):
  with tracer.start_span('format') as span:
      hello_str = 'Hello, %s!' % hello_to
      span.log_kv({'event': 'string-format', 'value': hello_str})
      return hello_str

# display given string
def print_hello(root_span, hello_str):
  with tracer.start_span('println') as span:
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

Let's run the above python program with a command line argument,  
say, `Alice`:

`python2.7 -m hello Alice`{{execute}}

We find three spans if we observe the output carefully, each with a unique  
Jaeger trace ID (the first hexadecimal segment). If we search the IDs  
in the [Jaeger UI's](https://[[HOST_SUBDOMAIN]]-16686-[[KATACODA_HOST]].environments.katacoda.com/search) dashboard, each ID will represent a separate trace  
with a single span. Instead, we would want to set up a relationship  
between the two new spans - `format` and `println` - and the main span  
`say_hello`.

*Note* :To see a trace in Jaeger UI dashboard, select service name as `hello-world`  
and operation name as either `say-hello`, `format` or `println` accordingly.
