---
layout: post
comments: true
title: Big Data, Little Laptop (Part 1)
excerpt: Distributed computing is a powerful thing, but the use of traditional computing systems to store, process, and analyze data is often unavoidable. When the size of your data is so great that your laptop slows to a crawl, or when you need to develop and test your code locally before deploying it in a distributed environment, these simple tools and methods might be exactly what you need.
header-img: images/big-data-little-laptop.jpg
date: 2017-12-12
author: pjryan126
category: techblog
tags: [python, multithreading, big data]
---

# Introduction

Distributed computing is a powerful thing. With systems like Hadoop, Apache Spark, and Elasticsearch, data and computing professionals are conquering problems that were previously insurmountable in more traditional computing systems. What's more, their continuing growth and development in cloud computing environments has made distributed systems accessible and affordable to global enterprises and aspiring data scientists and developers alike.  

When you are working as a consultant or in back office operations, though, every minute and penny counts, and in some cases, it makes sense to test the limits of your traditional systems or, at the very least, to plan and test your code locally before jumping feet first into a distributed computing environment. 

In other cases, it may not even be your choice. Perhaps you are working as an outside consultant, and your clients are extremely sensitive to where their data is stored and processed. With the neverending newstream of embarrassing data breaches, could you blame them if that were the case?

Whatever the constraints, the use of traditional computing systems to store, process, and analyze data is often unavoidable, and when the size of that data is so great that your laptop slows to a crawl, or when you need to develop and test your code locally before deploying it in a distributed environment, you might need a little help. 

In this series of posts, I would like to explore some tools and methods for Python that can help you work with larger data sets directly on your laptop. I won't hit on every such tool available in the Python ecosystem, but I touch on some of the more common tools that I have encountered over my years of Python coding. 

We will start with some simple yet powerful coding patterns that help to maximize available CPU and RAM -- multithreading and multiprocessing.

# Getting Started

Your Python distribution will have all of the packages required for writing multithreaded and multiprocess programs, but you can make your life a little easier on the multiprocessing side by installing `pathos`, a fork of the standard Python `multiprocessing` library that allows to pass just about anything to your processes without encountering a pickling error. For purposes of this article, all code snippets have been written with Python 3 in mind.

```
$ pip install pathos
```

# Multithreading vs. MultiProcessing

At this point, multi-core CPUs are practically standard equipment in any laptop you might buy. Each core processes a string -- or "thread" -- of program instructions independently of one another. The ability to run more than one thread of instructions at a time can lead to significant performance increases, especially when running process-intensive tasks. 

But Python was developed to keep things simple. To remove the possibility of conflicts arising among multiple threads of programming instructions, Python's interpreter includes something called a "Global Interpreter Lock" (the "GIL"), which prevents all other threads from running in a CPU (or in the particular core of a multi-core CPU) while execution of a Python statement is underway. By locking all threads during execution, Python forces a "single-threaded" processing environment, where each Python statement is serially executed. 

For this reason, creating multiple threads to handle execution of Python statements generally provides no benefit.  Python programs with complex processing statements are typically "CPU-bound", meaning the time that it takes to run the program is primarily dependent on how long it takes to serially process all of the program's Python statements.

We can still greatly improve the performance of certain Python programs, however, using a multithreaded approach. Such programs are generally "IO-bound", that is, the time that it takes to complete the program is primarily dependent on how long it takes to perform network or filesystem tasks, such as calling a web-based API or working with a large number of files on an external drive.  

# Multithreaded Programming

In many cases, the coding pattern for a multithreaded Python program is fairly straightforward. Items for processing are placed into a processing queue, and a number of threads are opened to process the items. Each item of the queue is fed to the thread, and as soon as a thread has finished processing an item, it plucks another item from the queue. 

An example of a coding pattern that I often use for multithreaded Python programs is below. In this example, I am reading the first five rows of data from each of hundreds of identically formatted Microsoft Excel Files and combining the data into a single `pandas` data frame. 

```
from multiprocessing import Manager
import os
from queue import Queue
import shutil
from threading import Thread

import pandas as pd

# set the number of threads to open
num_threads = 20

# initialize the queue
queue = Queue()

# initialize a multiprocessing list object to hold results of each thread
manager = Manager()
mlist = manager.list()

# define directory holding files
pwd = '../data'

# build queue
files = []
for root, dirnames, filenames in os.walk(pwd):
    # iterate over MS Excel files
    for f in filenames:
        # check if file is a MS Excel file
        fpath = os.path.join(root, f)
        if os.path.isfile(fpath) and os.path.splitext(f)[-1] in ('.xls', '.xlsx'):
            queue.put(fpath)

# define worker function -- this is the function that will process each item in the queue
def worker_func(i, q):
    while True:
        try:
            # get queue item
            fpath = q.get()
            
            # read MS Excel file and append first five rows 
            df = pd.read_excel(fpath)[:5]
            # add filepath to data frame
            df['fpath'] = fpath
            
            # append first 5 rows of data frame to multiprocessing list
            mlist.append(df)
        
        except Exception as e:
            print(e)
        
        q.task_done()

# open threads
for i in range(num_threads):
    worker = Thread(target=worker_func, args=(i, queue))
    worker.setDaemon(True)
    worker.start()

# start the task queue
queue.join()

# join data frames from each queue item into a single df
df = pd.concat(mlist)
```

First, we define some variables for the program, including the number of threads to open. You should experiment with the number of threads to see what works best. It will vary from program to program.

Second, we build the queue of items to be processed. The queue above is a simple list of file paths, but the items can be more complex, **e.g.**, a list of `pandas` series objects. 

Third, we define a worker function, which is the function that each thread will apply to items in the queue. If your code is complex, you can define additional helper functions and then call each helper function from inside the primary worker function. Keep in mind, however, that we are solving IO-bound problems with this code pattern, so if your code is too complex, you might wind up turning your IO-bound problem into a CPU-bound problem.

Fourth, we open the threads and start feeding items from the queue. Each thread opens the filepath item, reads the first five rows into a `pandas` data frame, performs a light transformation, and then adds the data frame to a special list type that makes itself available to all of the open threads. 

Finally, when all of the items have been processed, we concatenate the outputs stored in the multiprocessing list into a single data frame for review. 

# Conclusion

This is a fairly simple example of multithreading in Python, but it can be extremely powerful. When I'm dealing with processing thousands of files, I almost always try using this method first before considering a distributed solution like Hadoop.

In the next article, I will continue the discussion with an overview and example of multiprocessing in Python. The method can be invaluable when dealing complex computations or data transformations, and it's often no more complicated than the multithreaded example above. 