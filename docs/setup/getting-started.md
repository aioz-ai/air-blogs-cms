---
title: Getting Started
description: Understand basic concepts of building a federated learning algorithm
---

import Step from '@site/src/components/Step';
import Steps from '@site/src/components/Steps';
import Tabs from '@theme/Tabs';
import TabItem from '@theme/TabItem';
import Alert from '@site/src/components/Alert';
import Jump from '@site/src/components/Jump';

In this tutorial, we will cover how to setup a federated learning algorithm.

## Prerequisites

<Step headingDepth={3}>
<ol>
<li>

### Install required dependencies

<Tabs
  block={true}
  defaultValue="macOS"
  values={[
    { label: 'macOS', value: 'macOS', },
    { label: 'ubuntu', value: 'ubuntu', },
    { label: 'centos7', value: 'centos7', },
    { label: 'windows', value: 'windows', },
  ]
}>
<TabItem value="macOS">  

```
$ brew install python
```

</TabItem>
<TabItem value="ubuntu">

```
$ sudo apt update
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
pip install tensorflow
```

</li>
</ol>
</Step>

Once the prerequisites are installed, you can start to build your algorithm.

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

The compilation can take up to few minutes depending on your hardware. Why don't use this time to glance through the [key concept in AIOZ AI Network][concepts].

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

<Jump to="../setup/develop/">View Development Tutorial</Jump>

* Deploy your application.

<Jump to="../setup/deploy">View Deployment Guide</Jump>

If you experienced any issues with this tutorial or want to provide feedback, feel free to open a GitHub issue or reach out on Slack.

[concepts]: ../../about/concepts
