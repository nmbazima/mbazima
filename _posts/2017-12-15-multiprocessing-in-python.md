---
layout: post
comments: true
title: Multiprocessing in Python
excerpt: Parallel computing is a power tool, and Python makes it easy. If you are dealing with a long-running program, Python's multiprocessing package might what you need. 
image: images/python-logo.png
date: 2017-12-12
author: pjryan126
category: techblog
tags: [python, multiprocessing, big data, parallel computing]
---

By the time the kids woke me up this morning, there were four inches of snow on the ground. My typical commute into work can take anywhere from ninety minutes to two and a half hours, so the need to shovel snow before catching a bus was frustrating, to say the least. 

Inside my shed are four shovels. Unfortunately, I can only use one shovel at a time, and it takes about an hour for that one shovel to push all the snow out of my driveway and into the street. I really don't have an hour to spare this morning, though, so I need to find some way to speed up this process. Luckily, I've got a couple of underworked miniature facsimiles of myself watching cartoons in the living room, and with a few minor threats (and counting all the way to two and a half), they were outside pushing snow into the street with me, while my wife stayed inside and worked on some of our other everyday tasks. 

And this, my friends, is multiprocessing. Nowadays, most of our laptops have multi-core processors, and each core can handle a series, or "thread", of tasks independently of the others. Many hands make light work, and a multiprocessing program can often reduce the time it would otherwise take for a processor to run it as a single thread of tasks by letting multiple cores or processors each handle some aspect of the overall plan. The trick is to make sure that your problem is "CPU-bound", that is, that the program's bottlenecks are related to processing data. If your bottlenecks are "IO-bound" (related to filesystem or network tasks), you might be better off using a multithreaded approach. 

One pattern that lends itself well to multiprocessing programs is the famous, and powerful, map-reduce algorithm. A workhorse when it comes to "embarrasingly parallel" problems, the map-reduce algorithm marshals input data into separate groups, applies the same function in each group, and then applies a second function to the output data from the original function to obtain a final result. (It's actually a lot more complicated than this, but this is basically what it does.)

Before we run, let's learn to walk. We'll use the `multiprocessing` package to compute the factorial for each number in a given list of numbers.

As always, we start with the imports.

```
from multiprocessing import Pool
from time import time # for timing execution of the code

import numpy as np # for generating list of integers
```

The Pool class of the multiprocessing package allows us to open a group of processes that can execute a given function multiple times at once. In other words, the pool of processes are prepared to run "in parallel." 

Next, we write a recursive function that multiplies a given integer by its predecessor. 

```
def factorial(n):
    if n == 0:
        return 1
    else:
        return n * factorial(n-1)
```

Next, we write a helper function that opens a pool of processes and then maps the inputs to a given function.

```
def run(n, func, inputs):
    with Pool(n) as p:
        p.map(func, inputs)
    return
```

Finally, we add the Python code to execute the factorial function on each of the items in a large list of integers that we create. We will execute the code with one, two, three, and four processes in the pool to see whether we see any improvements in speed.

```
# generate large list of integers with values between 1 and 100
inputs = np.random.randint(1, high=100, size=1000000).tolist()

# iterate over number of processes between 1 and 4
for i in range(4):
    start = time()
    run(i + 1, factorial, inputs)
    end = time() - start
    print('Processes: {}, Seconds: {:.2f}'.format(num_proc, end))
```

Putting it all together, our code looks like this.

```
from multiprocessing import Pool
from time import time # for timing execution of the code

import numpy as np # for generating list of integers


def factorial(n):
    if n == 0:
        return 1
    else:
        return n * factorial(n-1)

def run(n, func, inputs):
    with Pool(n) as p:
        p.map(func, inputs)
    return

for i in range(4):
    start = time()
    run(i + 1, factorial, inputs)
    end = time() - start
    print('Processes: {}, Seconds: {:.2f}'.format(num_proc, end))

```

When we run the code in a Jupyter notebook, we see something like the following output.

Processes: 1, Seconds: 8.40
Processes: 2, Seconds: 4.63
Processes: 3, Seconds: 5.27
Processes: 4, Seconds: 5.05

Splitting up the work among multiple processes improves the speed of even our toy example above. In cases where we are dealing with computationally expensive code, using multiprocessing might be just what you need to make it bearable. If not, you might need something even stronger. 

# Additional Reading

If you want to know how Python's `multithreading` package relates to all this, check out my last post (<a href="https://pjryan126.github.io/big-data-little-laptop-pt1-multithreading/">here</a>).

If you are interested in implementing a simple map-reduce program with the `multiprocessing` package, there is a great example at the Python Module of the Week site (<a href="https://pymotw.com/2/multiprocessing/mapreduce.html">here</a>).

For another great overview of both the `multithreading` and `multiprocessing` Python packages, check out Plogging Dev (<a href="https://www.ploggingdev.com/2017/01/multiprocessing-and-multithreading-in-python-3/">here</a>).


{% if page.comments %}

If you have any ideas on how to improve these methods, please feel free to leave a comment below. I'd love to hear about it.

<div id="disqus_thread"></div>
<script>

/**
*  RECOMMENDED CONFIGURATION VARIABLES: EDIT AND UNCOMMENT THE SECTION BELOW TO INSERT DYNAMIC VALUES FROM YOUR PLATFORM OR CMS.
*  LEARN WHY DEFINING THESE VARIABLES IS IMPORTANT: https://disqus.com/admin/universalcode/#configuration-variables*/
/*
var disqus_config = function () {
this.page.url = PAGE_URL;  // Replace PAGE_URL with your page's canonical URL variable
this.page.identifier = PAGE_IDENTIFIER; // Replace PAGE_IDENTIFIER with your page's unique identifier variable
};
*/
(function() { // DON'T EDIT BELOW THIS LINE
var d = document, s = d.createElement('script');
s.src = 'https://https-pjryan126-github-io.disqus.com/embed.js';
s.setAttribute('data-timestamp', +new Date());
(d.head || d.body).appendChild(s);
})();
</script>
<noscript>Please enable JavaScript to view the <a href="https://disqus.com/?ref_noscript">comments powered by Disqus.</a></noscript>
                            
{% endif %}
