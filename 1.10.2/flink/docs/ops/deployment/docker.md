---
title:  "Docker Setup"
nav-title: Docker
nav-parent_id: deployment
nav-pos: 4
---
<!--
Licensed to the Apache Software Foundation (ASF) under one
or more contributor license agreements.  See the NOTICE file
distributed with this work for additional information
regarding copyright ownership.  The ASF licenses this file
to you under the Apache License, Version 2.0 (the
"License"); you may not use this file except in compliance
with the License.  You may obtain a copy of the License at

  http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing,
software distributed under the License is distributed on an
"AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY
KIND, either express or implied.  See the License for the
specific language governing permissions and limitations
under the License.
-->

[Docker](https://www.docker.com) is a popular container runtime. 
There are Docker images for Apache Flink available on Docker Hub which can be used to deploy a session cluster.
The Flink repository also contains tooling to create container images to deploy a job cluster.

* This will be replaced by the TOC
{:toc}

## Flink session cluster

A Flink session cluster can be used to run multiple jobs. 
Each job needs to be submitted to the cluster after it has been deployed. 

### Docker images

The [Flink Docker repository](https://hub.docker.com/_/flink/) is hosted on
Docker Hub and serves images of Flink version 1.2.1 and later.

Images for each supported combination of Hadoop and Scala are available, and tag aliases are provided for convenience.

Beginning with Flink 1.5, image tags that omit a Hadoop version (e.g.
`-hadoop28`) correspond to Hadoop-free releases of Flink that do not include a
bundled Hadoop distribution.

For example, the following aliases can be used: *(`1.5.y` indicates the latest
release of Flink 1.5)*

* `flink:latest` → `flink:<latest-flink>-scala_<latest-scala>`
* `flink:1.5` → `flink:1.5.y-scala_2.11`
* `flink:1.5-hadoop27` → `flink:1.5.y-hadoop27-scala_2.11`

**Note:** The Docker images are provided as a community project by individuals
on a best-effort basis. They are not official releases by the Apache Flink PMC.

## Flink job cluster

A Flink job cluster is a dedicated cluster which runs a single job. 
The job is part of the image and, thus, there is no extra job submission needed. 

### Docker images

The Flink job cluster image needs to contain the user code jars of the job for which the cluster is started.
Therefore, one needs to build a dedicated container image for every job.
The `flink-container` module contains a `build.sh` script which can be used to create such an image.
Please see the [instructions](https://github.com/apache/flink/blob/{{ site.github_branch }}/flink-container/docker/README.md) for more details. 

## Using plugins
As described in the [plugins]({{ site.baseurl }}/ops/plugins.html) documentation page: in order to use plugins they must be
copied to the correct location in the Flink installation for them to work.

When running Flink from one of the provided Docker images by default no plugins have been activated.
The simplest way to enable plugins is to modify the provided official Flink docker images by adding
an additional layer. This does however assume you have a docker registry available where you can push images to and
that is accessible by your cluster.

As an example assume you want to enable the [S3]({{ site.baseurl }}/ops/filesystems/s3.html) plugins in your installation.

Create a Dockerfile with a content something like this: {% highlight dockerfile %}
# On which specific version of Flink is this based?
# Check https://hub.docker.com/_/flink?tab=tags for current options
FROM flink:{{ site.version }}-scala_2.12

# Install Flink S3 FS Presto plugin
RUN mkdir /opt/flink/plugins/s3-fs-presto && cp /opt/flink/opt/flink-s3-fs-presto* /opt/flink/plugins/s3-fs-presto

# Install Flink S3 FS Hadoop plugin
RUN mkdir /opt/flink/plugins/s3-fs-hadoop && cp /opt/flink/opt/flink-s3-fs-hadoop* /opt/flink/plugins/s3-fs-hadoop
{% endhighlight %}

Then build and push that image to your registry
{% highlight bash %}
docker build -t docker.example.nl/flink:{{ site.version }}-scala_2.12-s3 .
docker push     docker.example.nl/flink:{{ site.version }}-scala_2.12-s3
{% endhighlight %}

Now you can reference this image in your cluster deployment and the installed plugins are available for use.

## Flink with Docker Compose

[Docker Compose](https://docs.docker.com/compose/) is a convenient way to run a
group of Docker containers locally.

Example config files for a [session cluster](https://github.com/docker-flink/examples/blob/master/docker-compose.yml) and a [job cluster](https://github.com/apache/flink/blob/{{ site.github_branch }}/flink-container/docker/docker-compose.yml)
are available on GitHub.

### Usage

* Launch a cluster in the foreground

        docker-compose up

* Launch a cluster in the background

        docker-compose up -d

* Scale the cluster up or down to *N* TaskManagers

        docker-compose scale taskmanager=<N>

* Kill the cluster

        docker-compose kill

When the cluster is running, you can visit the web UI at [http://localhost:8081](http://localhost:8081). 
You can also use the web UI to submit a job to a session cluster.

To submit a job to a session cluster via the command line, you must copy the JAR to the JobManager
container and submit the job from there.

For example:

{% raw %}
    $ JOBMANAGER_CONTAINER=$(docker ps --filter name=jobmanager --format={{.ID}})
    $ docker cp path/to/jar "$JOBMANAGER_CONTAINER":/job.jar
    $ docker exec -t -i "$JOBMANAGER_CONTAINER" flink run /job.jar
{% endraw %}

{% top %}
