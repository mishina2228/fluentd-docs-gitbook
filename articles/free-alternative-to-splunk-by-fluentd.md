Free Alternative to Splunk Using Fluentd
========================================

[Splunk](http://www.splunk.com/) is a great tool for searching logs, but
its high cost makes it prohibitive for many teams. In this article, we
present a free and open source alternative to Splunk by combining three
open source projects: Elasticsearch, Kibana, and Fluentd.

![](/images/kibana5-screenshot.png)


[Elasticsearch](https://www.elastic.co/products/elasticsearch) is an
open source search engine known for its ease of use.
[Kibana](https://www.elastic.co/products/kibana) is an open source Web
UI that makes Elasticsearch user friendly for marketers, engineers and
data scientists alike.

By combining these three tools (Fluentd + Elasticsearch + Kibana) we get
a scalable, flexible, easy to use log search engine with a great Web UI
that provides an open-source Splunk alternative, all for free.

![](/images/fluentd-elasticsearch-kibana.png)


In this guide, we will go over installation, setup, and basic use of
this combined log search solution. This article was tested on Mac OS X
Mountain Lion. **If you're not familiar with Fluentd**, please learn
more about Fluentd first.


[What is Fluentd?](/articles/architecture.md)


Prerequisites
-------------

### Java for Elasticsearch

Please confirm that your Java version is 8 or higher.

``` {.CodeRay}
$ java -version
java version "1.8.0_111"
Java(TM) SE Runtime Environment (build 1.8.0_111-b14)
Java HotSpot(TM) 64-Bit Server VM (build 25.111-b14, mixed mode)
```

Now that we've checked for prerequisites, we're now ready to install and
set up the three open source tools.

Set Up Elasticsearch
--------------------

To install Elasticsearch, please download and extract the Elasticsearch
package as shown below.

``` {.CodeRay}
$ curl -O https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-5.0.2.tar.gz
$ tar zxvf elasticsearch-5.0.2.tar.gz
$ cd elasticsearch-5.0.2
```

Once installation is complete, start Elasticsearch.

``` {.CodeRay}
$ ./bin/elasticsearch
```

Set Up Kibana
-------------

To install Kibana, download it via the official webpage and extract it.
Kibana is a HTML / CSS / JavaScript application. Download page is
[here](https://www.elastic.co/downloads/kibana). In this article, we
download Mac OS X binary.

``` {.CodeRay}
$ curl -O https://artifacts.elastic.co/downloads/kibana/kibana-5.0.2-darwin-x86_64.tar.gz
$ tar zxvf kibana-5.0.2-darwin-x86_64.tar.gz
$ cd kibana-5.0.2-darwin-x86_64
```

Once installation is complete, start Kibana and run `./bin/kibana`. You
can modify Kibana's configuration via `config/kibana.yml`.

``` {.CodeRay}
$ ./bin/kibana
```

Access `http://localhost:5601` in your browser.

Set Up Fluentd (td-agent)
-------------------------

In this guide We'll install td-agent, the stable release of Fluentd.
Please refer to the guides below for detailed installation steps.

-   [Debian Package](/articles/install-by-deb.md)
-   [RPM Package](/articles/install-by-rpm.md)
-   [Ruby gem](/articles/install-by-gem.md)

Next, we'll install the Elasticsearch plugin for Fluentd:
fluent-plugin-elasticsearch. Then, install fluent-plugin-elasticsearch
as follows.

``` {.CodeRay}
$ sudo /usr/sbin/td-agent-gem install fluent-plugin-elasticsearch --no-document
```

We'll configure td-agent (Fluentd) to interface properly with
Elasticsearch. Please modify `/etc/td-agent/td-agent.conf` as shown
below:

``` {.CodeRay}
# get logs from syslog
<source>
  @type syslog
  port 42185
  tag syslog
</source>

# get logs from fluent-logger, fluent-cat or other fluentd instances
<source>
  @type forward
</source>

<match syslog.**>
  @type elasticsearch
  logstash_format true
  flush_interval 10s # for testing
</match>
```

fluent-plugin-elasticsearch comes with a logstash\_format option that
allows Kibana to search stored event logs in Elasticsearch.

Once everything has been set up and configured, we'll start td-agent.

``` {.CodeRay}
$ sudo /etc/init.d/td-agent start
```

Set Up rsyslogd
---------------

In our final step, we'll forward the logs from your rsyslogd to Fluentd.
Please add the following line to your `/etc/rsyslog.conf`, and restart
rsyslog. This will forward your local syslog to Fluentd, and Fluentd in
turn will forward the logs to Elasticsearch.

``` {.CodeRay}
*.* @127.0.0.1:42185
```

Please restart the rsyslog service once the modification is complete.

``` {.CodeRay}
$ sudo /etc/init.d/rsyslog restart
```

Store and Search Event Logs
---------------------------

Once Fluentd receives some event logs from rsyslog and has flushed them
to Elasticsearch, you can search the stored logs using Kibana by
accessing Kibana's index.html in your browser. Here is an image example.

![](/images/kibana5-screenshot.png)


To manually send logs to Elasticsearch, please use the `logger` command.

``` {.CodeRay}
$ logger -t test foobar
```

When debugging your td-agent configuration, using
[filter\_stdout](/articles/filter_stdout.md) will be useful. All the logs including
errors can be found at `/etc/td-agent/td-agent.log`.

``` {.CodeRay}
<filter syslog.**>
  @type stdout
</filter>

<match syslog.**>
  @type elasticsearch
  logstash_format true
  flush_interval 10s # for testing
</match>
```

Conclusion
----------

This article introduced the combination of Fluentd and Kibana (with
Elasticsearch) which achieves a free alternative to Splunk: storing and
searching machine logs. The examples provided in this article have not
been tuned.

If you will be using these components in production, you may want to
modify some of the configurations (e.g. JVM, Elasticsearch, Fluentd
buffer, etc.) according to your needs.

Learn More
----------

-   [Fluentd Architecture](/articles/architecture.md)
-   [Fluentd Get Started](/articles/quickstart.md)
-   [Downloading Fluentd](http://www.fluentd.org/download)




If this article is incorrect or outdated, or omits critical information,
please [let us know](https://github.com/fluent/fluentd-docs/issues?state=open).
[Fluentd](http://www.fluentd.org/) is a open source project under [Cloud
Native Computing Foundation (CNCF)](https://cncf.io/). All components
are available under the Apache 2 License.