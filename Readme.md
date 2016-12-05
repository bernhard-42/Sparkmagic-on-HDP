# 1 Configure Livy in Ambari

Until [https://github.com/jupyter-incubator/sparkmagic/issues/285](https://github.com/jupyter-incubator/sparkmagic/issues/285) is fixed, set 

	livy.server.csrf_protection.enabled ==> false

in Ambari under `Spark Config` - `Advanced livy-conf`

# 2 Install and Configure Sparkmagic

Details see [https://github.com/jupyter-incubator/sparkmagic](https://github.com/jupyter-incubator/sparkmagic)

## 2.1 Install sparkmagic

```bash
$ sudo -H pip install sparkmagic
```

## 2.2 Install Kernels

```bash
$ sudo -H pip show sparkmagic # check path, e.g /usr/local/lib/python2.7/site-packages

$ cd /usr/local/lib/python2.7/site-packages

$ sudo -H jupyter-kernelspec install --user sparkmagic/kernels/sparkkernel
$ sudo -H jupyter-kernelspec install --user sparkmagic/kernels/pysparkkernel

```

## 2.3 Install widgets

```bash
$ sudo -H jupyter nbextension enable --py --sys-prefix widgetsnbextension
```

## 2.4 Install config

- To avoid timeouts connecting to HDP 2.5 it is important to add

	```json
	"livy_server_heartbeat_timeout_seconds": 0,
	```

- To ensure the Spark job will run on the cluster (livy default is local), `spark.master` needs to be set to `yarn-cluster`, a `conf` object needs to be provided. Here you can also add extra jars for the session:

	```json
	"session_configs": {
		"driverMemory": "2G",
		"executorCores": 4,
		"proxyUser": "bernhard",
		"conf": {
			"spark.master": "yarn-cluster",
			"spark.jars.packages": "com.databricks:spark-csv_2.10:1.5.0"
		}
	}
	```

Here is an example [config.json](config.json). Adapt and copy to `~/.sparkmagic`


# 3 Run Notebooks with Sparkmagic

## 3.1 Start Jupyter Notebooks

Start Jupyter

```bash
$ cd <project-dir>
$ jupyter notebook
```

In Notebook Home select *New -> Spark* or *New -> Spark* or *New Python*

## 3.2 Load Sparkmagic

Add into your Notebook after the Kernel started

```python
%load_ext sparkmagic.magics
%manage_spark
```

## 3.3 Configure Spark Access

### 3.3.1 Select *Add Endpoint*

![Add Endpoint](images/Add-Endpoint.png)

**Result:**

![Add Endpoint - Success](images/Add-Endpoint-Success.png)

### 3.3.2 Create Session

Select *Create Session*. If you have copied [config.json](config.json) to `~/.sparkmagic` before you start `jupyter` the properties section will be prefilled with the `session_configs` JSON object

![Create Session](images/Create-Session.png)

**Result:**

![Create Session - Success](images/Create-Session-Success.png)

# 4 Notes

- Livy on HDP 2.5 currently does not return YARN Application ID
- Jupyter session name provided under *Create session* is notebook intenal and not used by Livy Server on the cluster. Livy-Server will create sessions on YARN called livy-session-###, e.g. livy-session-10. The session in Jupyter will have session id ###, e.g. 10.
- For multiline Scala code in the Notebook you have to add the dot at the end, as in

	```scala
	val df = sqlContext.read.
                    format("com.databricks.spark.csv").
                    option("header", "true").
                    option("inferSchema", "true").
                    load("/tmp/iris.csv")
	```

# 5 Examples

[Sparkmagic for Scala](Sparkmagic-Scala.ipynb)

[Spark in Python Notebook](Spark-in-Python-Notebook.ipynb)
