Since we want to establish a causal relationship between the two new spans  
`format`; `println` and the root span `say-hello`, we pass an additional  
argument to the `start_span` method.

<pre class="file">
def format_string(root_span, hello_to):
  with tracer.start_span('format', child_of=root_span) as span:
      hello_str = 'Hello, %s!' % hello_to
      span.log_kv({'event': 'string-format', 'value': hello_str})
      return hello_str

def print_hello(root_span, hello_str):
  with tracer.start_span('println', child_of=root_span) as span:
      print(hello_str)
      span.log_kv({'event': 'println'})
</pre>

A trace is a directed acyclic graph where the nodes represent spans  
and the edges represent causal relationships or references between spans.  
In the OpenTracing Specification, the edges are abstracted as `SpanReference`  
type that consists of `SpanContext` and `ReferenceType` label.

The `SpanContext` is an immutable thread safe portion of the span that can  
be used to establish references between spans or to propagate spans over the  
wire.

The `ReferenceType` describes the nature of the relationship between spans.  
There are two kinds of references:  
* `ChildOf` reference: A span maybe a `ChildOf` parent span. In this example,  
`root_span` is the parent span of `format` and `println` spans. In this  
reference, the parent span logically depends on its child span(s) before it  
can complete its operation.
* `FollowsFrom` reference: A child span `FollowsFrom` its parent span. For  
example, `root_span` maybe the ancestor of the DAG and it does not depend  
on completion of its child span(s).

Copy the below code into `hello.py` file (or use the `Copy to Editor`  
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

def format_string(root_span, hello_to):
  with tracer.start_span('format', child_of=root_span) as span:
      hello_str = 'Hello, %s!' % hello_to
      span.log_kv({'event': 'string-format', 'value': hello_str})
      return hello_str

def print_hello(root_span, hello_str):
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

Let's run the modified python program:  
`python2.7 -m hello Alice`{{execute}}

The reported spans in the output now belong to the same trace. We observe  
that the third hexadecimal segment in both the new spans is no longer `0`  
but is the value of the `root_span` Jaeger trace ID.

Let's find out how the trace would look in the [Jaeger UI](https://[[HOST_SUBDOMAIN]]-16686-[[KATACODA_HOST]].environments.katacoda.com/search).  
We can observe that the spans `format` and `println` are causally related to  
the root span `say-hello`.   

*Note* :To see a trace in *Jaeger UI* dashboard, follow these steps:  
1. Click **service name** drop-down list, and select `hello-world`  
2. Click **operation name** drop-down list, and select `say-hello`,  
   `format` or `println`
