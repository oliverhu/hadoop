<!---
  Licensed under the Apache License, Version 2.0 (the "License");
  you may not use this file except in compliance with the License.
  You may obtain a copy of the License at

   http://www.apache.org/licenses/LICENSE-2.0

  Unless required by applicable law or agreed to in writing, software
  distributed under the License is distributed on an "AS IS" BASIS,
  WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
  See the License for the specific language governing permissions and
  limitations under the License. See accompanying LICENSE file.
-->

# Quick Start Guide

## Prerequisite

Must:

- Apache Hadoop 2.7 or above.

Optional:

- Enable GPU on YARN support (when GPU-based training is required, Hadoop 3.1 and above).
- Enable Docker support on Hadoop (Hadoop 2.9 and above).

## Run jobs

### Commandline options

```$xslt
usage:
 -docker_image <arg>          Docker image name/tag
 -env <arg>                   Common environment variable of worker/ps
 -name <arg>                  Name of the job
 -num_ps <arg>                Number of PS tasks of the job, by default
                              it's 0
 -num_workers <arg>           Numnber of worker tasks of the job, by
                              default it's 1
 -ps_docker_image <arg>       Specify docker image for PS, when this is
                              not specified, PS uses --docker_image as
                              default.
 -ps_launch_cmd <arg>         Commandline of worker, arguments will be
                              directly used to launch the PS
 -ps_resources <arg>          Resource of each PS, for example
                              memory-mb=2048,vcores=2,yarn.io/gpu=2
 -queue <arg>                 Name of queue to run the job, by default it
                              uses default queue
 -saved_model_path <arg>      Model exported path (savedmodel) of the job,
                              which is needed when exported model is not
                              placed under ${checkpoint_path}could be
                              local or other FS directory. This will be
                              used to serve.
 -tensorboard <arg>           Should we run TensorBoard for this job? By
                              default it's true
 -verbose                     Print verbose log for troubleshooting
 -wait_job_finish             Specified when user want to wait the job
                              finish
 -worker_docker_image <arg>   Specify docker image for WORKER, when this
                              is not specified, WORKER uses --docker_image
                              as default.
 -worker_launch_cmd <arg>     Commandline of worker, arguments will be
                              directly used to launch the worker
 -worker_resources <arg>      Resource of each worker, for example
                              memory-mb=2048,vcores=2,yarn.io/gpu=2
 -localization <arg>          Specify localization to remote/local
                              file/directory available to all container(Docker).
                              Argument format is "RemoteUri:LocalFilePath[:rw]"
                              (ro permission is not supported yet).
                              The RemoteUri can be a file or directory in local
                              or HDFS or s3 or abfs or http .etc.
                              The LocalFilePath can be absolute or relative.
                              If relative, it'll be under container's implied
                              working directory.
                              This option can be set mutiple times.
                              Examples are
                              -localization "hdfs:///user/yarn/mydir2:/opt/data"
                              -localization "s3a:///a/b/myfile1:./"
                              -localization "https:///a/b/myfile2:./myfile"
                              -localization "/user/yarn/mydir3:/opt/mydir3"
                              -localization "./mydir1:."
 -insecure                    Whether running in an insecure cluster
 -conf                        Override configurations via commandline
```

### Submarine Configuration

For submarine internal configuration, please create a `submarine.xml` which should be placed under `$HADOOP_CONF_DIR`.
Make sure you set `submarine.runtime.class` to `org.apache.hadoop.yarn.submarine.runtimes.tony.TonyRuntimeFactory`

|Configuration Name | Description |
|:---- |:---- |
| `submarine.runtime.class` | org.apache.hadoop.yarn.submarine.runtimes.tony.TonyRuntimeFactory
| `submarine.localization.max-allowed-file-size-mb` | Optional. This sets a size limit to the file/directory to be localized in "-localization" CLI option. 2GB by default. |



### Launch TensorFlow Application:

#### Commandline

### Without Docker
```
CLASSPATH=$(hadoop classpath --glob): \
./hadoop-submarine-core/target/hadoop-submarine-core-0.2.0-SNAPSHOT.jar: \
./hadoop-submarine-yarnservice-runtime/target/hadoop-submarine-score-yarnservice-runtime-0.2.0-SNAPSHOT.jar: \
./hadoop-submarine-tony-runtime/target/hadoop-submarine-tony-runtime-0.2.0-SNAPSHOT.jar: \
/home/pi/hadoop/TonY/tony-cli/build/libs/tony-cli-0.3.0-all.jar \

java org.apache.hadoop.yarn.submarine.client.cli.Cli job run --name tf-job-001 \
 --input_path hdfs://pi-aw:9000/dataset/cifar-10-data \
 --worker_resources memory=3G,vcores=2 \
 --worker_launch_cmd "export CLASSPATH=\$(/hadoop-3.1.0/bin/hadoop classpath --glob) && cd /test/models/tutorials/image/cifar10_estimator && python cifar10_main.py --data-dir=%input_path% --job-dir=%checkpoint_path% --train-steps=10000 --eval-batch-size=16 --train-batch-size=16 --variable-strategy=CPU --num-gpus=0 --sync" \
 --container_resources /home/pi/hadoop/TonY/tony-cli/build/libs/tony-cli-0.3.0-all.jar, \
 --env JAVA_HOME=/usr/lib/jvm/java-8-openjdk-amd64 \
 --env HADOOP_HOME=/hadoop-3.1.0 \
 --env HADOOP_YARN_HOME=/hadoop-3.1.0 \
 --env HADOOP_COMMON_HOME=/hadoop-3.1.0 \
 --env HADOOP_HDFS_HOME=/hadoop-3.1.0 \
 --env HADOOP_CONF_DIR=/hadoop-3.1.0/etc/hadoop
```

### With Docker
```
CLASSPATH=$(hadoop classpath --glob): \
./hadoop-submarine-core/target/hadoop-submarine-core-0.2.0-SNAPSHOT.jar: \
./hadoop-submarine-yarnservice-runtime/target/hadoop-submarine-score-yarnservice-runtime-0.2.0-SNAPSHOT.jar: \
./hadoop-submarine-tony-runtime/target/hadoop-submarine-tony-runtime-0.2.0-SNAPSHOT.jar: \
/home/pi/hadoop/TonY/tony-cli/build/libs/tony-cli-0.3.0-all.jar \

java org.apache.hadoop.yarn.submarine.client.cli.Cli job run --name tf-job-001 \
 --docker_image hadoopsubmarine/tf-1.8.0-cpu:0.0.3 \
 --input_path hdfs://pi-aw:9000/dataset/cifar-10-data \
 --worker_resources memory=3G,vcores=2 \
 --worker_launch_cmd "export CLASSPATH=\$(/hadoop-3.1.0/bin/hadoop classpath --glob) && cd /test/models/tutorials/image/cifar10_estimator && python cifar10_main.py --data-dir=%input_path% --job-dir=%checkpoint_path% --train-steps=10000 --eval-batch-size=16 --train-batch-size=16 --variable-strategy=CPU --num-gpus=0 --sync" \
 --env JAVA_HOME=/usr/lib/jvm/java-8-openjdk-amd64 
 --env DOCKER_HADOOP_HDFS_HOME=/hadoop-3.1.0 \
 --env DOCKER_JAVA_HOME=/usr/lib/jvm/java-8-openjdk-amd64 \
 --container_resources /home/pi/hadoop/TonY/tony-cli/build/libs/tony-cli-0.3.0-all.jar \
 --env HADOOP_HOME=/hadoop-3.1.0 \
 --env HADOOP_YARN_HOME=/hadoop-3.1.0 \
 --env HADOOP_COMMON_HOME=/hadoop-3.1.0 \
 --env HADOOP_HDFS_HOME=/hadoop-3.1.0 \
 --env HADOOP_CONF_DIR=/hadoop-3.1.0/etc/hadoop
```
