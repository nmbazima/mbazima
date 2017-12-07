---
layout: post
comments: true
title: Connecting to a Database in Jupyter
excerpt: SQL is everywhere, and if you are doing any sort of analysis in an enterprise setting, it is more likely than not that you will need to access a SQL database for at least some of your data. With the pandas library, extracting data from a SQL database in a Jupyter notebook is almost trivial, but before we can extract the data, we need to establish a connection to the database. 
---

# Introduction

SQL is everywhere, and if you are doing any sort of analysis in an enterprise setting, it is more likely than not that you will need to access a SQL database for at least some of your data. With the pandas library, extracting data from a SQL database in a Jupyter notebook is almost trivial, but before we can extract the data, we need to establish a connection to the database. 

Here we explore some methods for establishing a connection to a SQL database in a Jupyter notebook. For purposes of this tutorial, we will assume the database is stored on a Microsoft SQL Server, but the connection process should be about the same no matter what type of database management system you are using. We will also assume that there is an existing SQL user with all of the permissions required to access the database.

We will start with the least secure method -- hardcoding credentials into the connection code -- because it is easy to understand, but we will build toward our ultimate goal, a connection that is both secure and easy to use.

# Getting Started

Before we do anything, we will need to install some third-party Python packages to help us establish and use our connections. The sqlalchemy engine works well with pandas dataframes, so we will use those libraries to perform our SQL queries below. If you are curious, sqlalchemy's 'create_engine' function can leverage the pyodbc library to connect to a SQL Server, so we import that library, as well.

If you do not already have these packages installed, you can install them using pip. The pyodbc library can be tricky to install, so a bit of light googling might also be required.

```
! pip install pandas
! pip install pyodbc
! pip install sqlalchemy 
```

# Method 1: Hard-Coding 

Now that we have the initial imports out of the way, we can establish our first database connection. To connect to a SQL Server via ODBC, the *sqlalchemy* library requires a connection string that provides all of the parameter values necessary to (1) identify the database and (2) authenticate and authorize the user. For Python beginners, the simplest way to provide that string is to define it directly in the notebook. Below we define the connection string and parse it into the necessary format for *pyodbc* using the *urllib* library. Make sure you substitute your own values for the 'DATABASE', 'UID', and 'PWD' parameters, or you won't be able to connect.

```
import urllib

params = 'DRIVER={ODBC Driver 13 for SQL Server};' \
         'SERVER=localhost;' \
         'PORT=1433;' \
         'DATABASE=nb-database;' \
         'UID=nb-user;' \
         'PWD=nb-password;'
            
params = urllib.parse.quote_plus(params)
```

Here is what the parsed connection string looks like:

DRIVER%3D%7BODBC+Driver+13+for+SQL+Server%7D%3BSERVER%3Dlocalhost%3BPORT%3D1433%3BDATABASE%3Dnb-database%3BUID%3Dnb-user%3BPWD%3Dnb-password%3B

Using this connection string, we then create a *sqlalchemy* database engine with a persistent connection to the database.

```
from sqlalchemy import create_engine

db = create_engine('mssql+pyodbc:///?odbc_connect=%s' % params)
```

The database engine then allows us to query the database and return the results as a *pandas* dataframe using the pandas 'read_sql_query' method.

```
import pandas as pd

sql = '''
SELECT *
FROM dbo.spt_monitor
ORDER BY lastrun DESC;
'''

pd.read_sql_query(sql, db)
```

# Method 2: Decoupling Credentials

Hard-coding a connection string can be the simplest and fastest way to establish a database connection, and in an enterprise setting, speed can be highly valued. But the hardcoding method presents some extremely serious issues. 

First, placing login credentials directly into the code presents a serious security issue. Anyone with access to the code can discover the credentials quite easily, and if the code is maintained on publicly accessible version control site like Github, you could be giving your data away to anyone who wants it. 

Second, if the login credentials change, you will need to find and update every instance of the connection string in your files or Jupyter notebooks. If the connection is a one-time need, this is no big deal, but if you rely on the same connection string for connections in numerous notebooks, an update to the credentials might cause some serious tension headaches.  

To mitigate the side effects of hardcoding a connection string, we can try storing our credentials in a separate file and importing the credentials when we need to establish a connection. For example, we could create a file called **creds.py** in the same directory as our Jupyter notebook and then place the credential values in a Python dictionary.

```
%writefile ./creds.py

creds = dict(driver='{ODBC Driver 13 for SQL Server}',
             server='localhost',
             port=1433,
             database='nb-database',
             user='nb-user',
             passwd='nb-password')
```

We can then import the creds dictionary whenever we need to establish a database connection. In fact, to minimize code reuse, we can also create our database engine object in its own **database.py** module and import it into our notebooks whenever we need it.

```
%writefile ./database.py
import urllib

import pyodbc
from sqlalchemy import create_engine

params = 'DRIVER=' + creds['driver'] + ';' \
         'SERVER=' + creds['server'] + ';' \
         'DATABASE=' + creds['database'] + ';' \
         'UID=' + creds['user'] + ';' \
         'PWD=' + creds['passwd'] + ';' \
         'PORT=' + str(creds['port']) + ';'
            
params = urllib.parse.quote_plus(params)
db = create_engine('mssql+pyodbc:///?odbc_connect=%s' % params)
```

By separating the credentials from the connection string, we can reduce the security risk posed by the hardcoding method by limiting distribution of the creds.py file. To hide it from viewers on Github, for example, we can simply add the filename to our .gitignore file.

Furthermore, if the credentials ever change -- perhaps the login credentials must be updated every 3 months -- we can simply update the creds.py file and know that the updates will be reflected in all of our Jupyter notebooks.

# Method 3: Login Prompts

The decoupling method is a vast improvement over hardcoding our credentials, but it's still far from ideal. The problem is that it reduces our security risk only if we remember to hide the creds.py file from other users, and even then, the file remains in plaintext on our own machine.

The obvious answer is to force users to enter credentials whenever they want to connect to the database. Of course, following this approach to the letter would maximize security but minimize ease of use. To balance these concerns, we can take a hybrid approach that uses a params.py file to store database parameters but prompts the user for authentication credentials.

The params.py file is similar to the creds.py file above, only the 'user' and 'passwd' parameter have been omitted. 

```
%%writefile ./params.py
params = dict(driver='{ODBC Driver 13 for SQL Server}',
              server='localhost',
              port=1433,
              database='nb-database')
```

We also update the database.py file by replacing the db object with a 'connect' function, which takes a username and password as parameters.

```
%%writefile ./database.py
import urllib

import pyodbc
from sqlalchemy import create_engine

from params import params

def connect(username, password):
    str_params = 'DRIVER=' + params['driver'] + ';' \
             'SERVER=' + params['server'] + ';' \
             'DATABASE=' + params['database'] + ';' \
             'UID=' + username + ';' \
             'PWD=' + password + ';' \
             'PORT=' + str(params['port']) + ';'
            
    str_params = urllib.parse.quote_plus(str_params)
    db = create_engine('mssql+pyodbc:///?odbc_connect=%s' % str_params)
    return db
```

Now we can prompt the user for a valid SQL username and password for the database directly in our Jupyter notebook. We use the *getpass* library to hide the password from view.

```
import getpass

from database import connect

username = input('username:')
password = getpass.getpass('password:')

db = connect(username, password)
```

The 'connect' function returns a sqlalchemy engine object, which we can again use to query the SQL database.

```
import pandas as pd

sql = '''
SELECT *
FROM dbo.spt_monitor
ORDER BY lastrun DESC;
'''

pd.read_sql_query(sql, db)
```

By retaining a Python module to hold the SQL parameters but prompting the user for login credentials, we ensure that the user's login credentials remain secret and maintain the overall ease of connecting to the database. When sharing notebooks among analysts and data scientists, this a very useful approach.

# Method 4: Environment Variables

The login prompt method is great when sharing notebooks with users who have database access, but what if we want to share the notebook with trusted but less technical users? Sometimes, we need to provide our analyses or visualizations to business users, and those business users need to interact with the data in some way. The simple solution would be to export the data to a flat file source (e.g., a .csv or .txt file), but size of the data might make that impossible. 

Enter environment variables. Assuming that we have a Jupyter server that our audience can access, we can serve our notebooks on the Jupyter server and save the necessary login credentials in the server's environment. In such case, our *database.py* file might look something like this.

```
%writefile ./database.py
import os
import urllib

import pyodbc
from sqlalchemy import create_engine

username = os.environ.get('username')
password = os.environ.get('password')

params = 'DRIVER=' + creds['driver'] + ';' \
         'SERVER=' + creds['server'] + ';' \
         'DATABASE=' + creds['database'] + ';' \
         'UID=' + username + ';' \
         'PWD=' + password + ';' \
         'PORT=' + str(creds['port']) + ';'
            
params = urllib.parse.quote_plus(params)
db = create_engine('mssql+pyodbc:///?odbc_connect=%s' % params)
```

The process of adding variables to a server environment will depend on the server's operating system, but if you have never done it before, there are plenty of online resources available to help you get started. It's easy when you get the hang of it.

Like the login prompts method, using environment variables maintains password secrecy, but it also allows us to provide multiple trusted users with authorization to query a database without having to provide those users with individual SQL accounts. 

# Conclusion

The correct method of connecting to a database in Jupyter will usually depend on the situation. If the notebook is a personal notebook, and the database is not so sensitive that you worry about intrusion, a hardcoded connection string might suffice. But if the database is sensitive, or if you are sharing the notebook with other users, I strongly recommend using one of the other options. 

In most cases, I wind up using Method 3 (Login Prompts) because it balances security with ease of use and does not require that I remember to exclude my credentials file from version control. Of course, in cases where I need to provide trusted business users with access to my analysis, I might lean toward Method 4 (Environment Variables). 

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
