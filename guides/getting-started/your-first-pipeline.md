---
last_modified_on: "2021-04-16"
title: Introduction to Federated Learning.
description: Introduction to decentralized federated learning.
series_position: 1
author_github: https://github.com/quangduytran
tags: ["type: tutorial", "level: medium"]
---

import Step from '@site/src/components/Step';
import Steps from '@site/src/components/Steps';
import Tabs from '@theme/Tabs';
import TabItem from '@theme/TabItem';
import Alert from '@site/src/components/Alert';
import Jump from '@site/src/components/Jump';

Federated Learning is a fascinating and upsurging Machine Learning technique for learning on decentralized data. The core idea is that a training dataset can remain in the hands of its producers (also known as *workers*), which helps improve privacy and ownership while the model is shared between workers.

In this tutorial, we will give an overview of federated learning, definitions, and implementation.

**Summary:** Simple code examples make learning easy. Here, we use the MNIST training task to introduce Federated Learning the easy way.

## Prerequisites

<Step headingDepth={3}>
<ol>
<li>

### Install required dependencies

<Tabs
  block={true}
  defaultValue="macOS"
  values={[
    { label: 'ubuntu', value: 'ubuntu', },
    { label: 'macOS', value: 'macOS', },    
    { label: 'windows', value: 'windows', },
    { label: 'centos7', value: 'centos7', },    
  ]
}>
<TabItem value="macOS">  

```
$ /usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
$ export PATH="/usr/local/bin:/usr/local/sbin:$PATH"
$ brew update
$brew install python  # Python 3
$ sudo pip3 install --user --upgrade virtualenv
```

</TabItem>
<TabItem value="ubuntu">

```
$ sudo apt update
$ sudo apt install python3-dev python3-pip  # Python 3
$ sudo pip3 install --user --upgrade virtualenv
```

</TabItem>
<TabItem value="centos7">

```
$ yum install -y python
```

</TabItem>
<TabItem value="windows">

```
$ pip install numpy
```

</TabItem>
</Tabs>

</li>
<li>

### Install Tensorflow

Let's implement a federated learning algorithm in Python with `Tensorflow`:

```
$ virtualenv --python python3 "venv"
$ source "venv/bin/activate"
$ pip install --upgrade pip
$ pip install tensorflow
$ pip install --upgrade tensorflow-federated
```

</li>
</ol>
</Step>

Once the prerequisites are installed, you can start to build your AI Network.

## Compiling AI Network

Firstly, you need to compile the source code.

<Step headingDepth={3}>
<ol>
<li>

### Clone the source code

   ```
   git clone https://github.com/aioz-ai/LDR_ALDK.git
   ```

</li>
<li>

### Compile the source code

   ```
   cd LDR_ALDK
   pip install -r requirements.txt
   ```

<Alert type="info">

The compilation can take up to few minutes depending on your hardware.

</Alert>

</li>
</ol>
</Step>

<Steps headingDepth={3}>

</Steps>

## Next Step

You have got the basis of federated learning, let's keep moving.

Your next step may be:

* Develop your first federated learning algorithm.

<Jump to="/guides/getting-started/optim-network/">View Development Tutorial</Jump>

* Deploy your application.

<Jump to="/docs/next/dev/dev-overview/">View Deployment Guide</Jump>

If you experienced any issues with this tutorial or want to provide feedback, feel free to open a GitHub issue or reach out on Slack.

## References
* [Tensorflow Federated Learning](https://www.tensorflow.org/federated)
