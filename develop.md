
### SBT

https://www.scala-sbt.org/download.html   
https://docs.databricks.com/user-guide/dev-tools/sbt-plugin.html  

### Gradle

https://gradle.org/install/

### Visual Studio

https://marketplace.visualstudio.com/search?term=spark&target=VS&category=All%20categories&vsVersion=&sortBy=Relevance

### Visual Studio Code

https://marketplace.visualstudio.com/search?term=spark&target=VSCode&category=All%20categories&sortBy=Relevance

Scala Language Server  
https://marketplace.visualstudio.com/items?itemName=dragos.scala-lsp

Cython
https://marketplace.visualstudio.com/items?itemName=tcwalther.cython  

Anaconda
https://blogs.msdn.microsoft.com/pythonengineering/2018/02/15/visual-studio-code-is-now-shipping-with-anaconda/  

### SQL Workbench/J

http://www.sql-workbench.eu/downloads.html  
https://docs.databricks.com/user-guide/bi/workbenchj.html

### Apache Airflow

https://docs.databricks.com/user-guide/dev-tools/data-pipelines.html#apache-airflow  

### Cython

https://docs.databricks.com/user-guide/faq/cython.html  
https://github.com/cython/cython  
https://github.com/cython/cython/wiki/CythonExtensionsOnWindows  
https://github.com/cython/cython/wiki/InstallingOnWindows  

### MinGW
http://sebsauvage.net/python/mingw.html  
https://sourceforge.net/projects/mingw/files/  

## Secrets

https://docs.databricks.com/user-guide/faq/hiding-secrets-databricks.html  

## Thrift Server

jdbc:hive2://${Driver Public DNS}:10000/default;ssl=false;transportMode=binary;httpPath=cliservice  

## DBFS Cli

```
# List files in DBFS
dbfs ls
# Put local file ./apple.txt to dbfs:/apple.txt
dbfs cp ./apple.txt dbfs:/apple.txt
# Get dbfs:/apple.txt and save to local file ./apple.txt
dbfs cp dbfs:/apple.txt ./apple.txt
# Recursively put local dir ./banana to dbfs:/banana
dbfs cp -r ./banana dbfs:/banana
```

```
dbutils.fs.mkdirs("/foobar/")
dbutils.fs.put("/foobar/baz.txt", "Hello, World!")
dbutils.fs.head("/foobar/baz.txt")
dbutils.fs.rm("/foobar/baz.txt")
display(dbutils.fs.ls("dbfs:/foobar"))
dbutils.fs.ls("file:/foobar")

```

## Spark DBFS API

```
#python
sc.parallelize(range(0, 100)).saveAsTextFile("/tmp/foo.txt")

//python
sc.parallelize(0 until 100).saveAsTextFile("/tmp/bar.txt")
```


## DBFS API

Put

```
curl -u USER:PASS -F contents=@localsrc -F path="PATH"
        https://YOUR_DOMAIN/api/2.0/dbfs/put

```

```
curl -u USER:PASS -F contents="BASE64" -F path="PATH"
        https://YOUR_DOMAIN/api/2.0/dbfs/put

curl -u USER:PASS -H "Content-Type: application/json"
-d '{"path":"PATH","contents":"BASE64"}' https://YOUR_DOMAIN/api/2.0/dbfs/put``
```