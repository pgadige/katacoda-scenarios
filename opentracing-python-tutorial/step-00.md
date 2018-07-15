Let's write a simple python program to print "Hello, `arg`!", where   
`arg` is a command line argument to the program.

Click the below link to open `hello.py` in the editor.    
`exercise/hello.py`{{open}}
The file is located at  
`/root/opentracing-tutorial/python/lesson01/exercise`

Once the file is opened in the editor, you may copy the below content  
into the file (or use the `Copy to Editor` button):

<pre class="file" data-filename="exercise/hello.py" data-target="replace">
## hello.py
## simple hello world program

import sys

def say_hello(hello_to):
  hello_str = 'Hello, %s!' % hello_to
  print hello_str

assert len(sys.argv) == 2

hello_to = sys.argv[1]
say_hello(hello_to)
</pre>

We run the above program with an argument, say `Alice`,  
`python2.7 hello.py Alice`{{execute}}

We should see the output as:  
`Hello, Alice!`

Let's instrument the above code to illustrate the features of     
OpenTracing (an opensource standard for distributed tracing) using   
Jaeger (an opensource distributed tracing system). You may also use other   
tracing systems, such as, Zipkin, LightStep etc.
