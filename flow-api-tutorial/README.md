# Flow API Tutorial 
<p align="center">
 
[![Jina](https://github.com/jina-ai/jina/blob/master/.github/badges/jina-badge.svg "We fully commit to open-source")](https://jina.ai)
[![Jina](https://github.com/jina-ai/jina/blob/master/.github/badges/jina-hello-world-badge.svg "Run Jina 'Hello, World!' without installing anything")](https://github.com/jina-ai/jina#jina-hello-world-)
[![Jina](https://github.com/jina-ai/jina/blob/master/.github/badges/license-badge.svg "Jina is licensed under Apache-2.0")](#license)
[![Jina Docs](https://github.com/jina-ai/jina/blob/master/.github/badges/docs-badge.svg "Checkout our docs and learn Jina")](https://docs.jina.ai)
[![We are hiring](https://github.com/jina-ai/jina/blob/master/.github/badges/jina-corp-badge-hiring.svg "We are hiring full-time position at Jina")](https://jobs.jina.ai)
<a href="https://twitter.com/intent/tweet?text=%F0%9F%91%8DCheck+out+Jina%3A+the+New+Open-Source+Solution+for+Neural+Information+Retrieval+%F0%9F%94%8D%40JinaAI_&url=https%3A%2F%2Fgithub.com%2Fjina-ai%2Fjina&hashtags=JinaSearch&original_referer=http%3A%2F%2Fgithub.com%2F&tw_p=tweetbutton" target="_blank">
  <img src="https://github.com/jina-ai/jina/blob/master/.github/badges/twitter-badge.svg"
       alt="tweet button" title="👍Share Jina with your friends on Twitter"></img>
</a>
[![Python 3.7 3.8](https://github.com/jina-ai/jina/blob/master/.github/badges/python-badge.svg "Jina supports Python 3.7 and above")](#)
[![Docker](https://github.com/jina-ai/jina/blob/master/.github/badges/docker-badge.svg "Jina is multi-arch ready, can run on differnt architectures")](https://hub.docker.com/r/jinaai/jina/tags)

</p>

In this demo, we'll use some code snippets to show you how to use Jina's Flow API to index or search different data. Make sure you've gone through [Jina 101](https://github.com/jina-ai/jina/tree/master/docs/chapters/101) and understood Pods, Flows, and YAML continuing.

<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->
**Table of Contents**

- [Overview](#overview)
- [3 APIs for Indexing and Searching Your Data](#3-apis-for-indexing-and-searching-your-data)
- [Wrap Up](#wrap-up)
- [Next Steps](#next-steps)
- [Documentation](#documentation)
- [Community](#community)
- [License](#license)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->
## Overview

Jina lets you index or search your data in a simple way. Whether you're indexing or searching, there are 3 APIs for each task, which cover *np.ndarray*, *files*, or *text*.

## 3 APIs for Indexing and Searching Your Data

### 1. API for `ndarray`

`index_ndarray()` is the API for indexing `np.ndarray` 

```python
import numpy as np
from jina.flow import Flow
input_data = np.random.random((3,8))
f = Flow().add(uses='_logforward')
with f:
    f.index_ndarray(input_data)
```
    
* Add a dummy Pod with config `_logforward` to the Flow. [`_logforward`](https://docs.jina.ai/chapters/simple_exec.html) is a built-in YAML, which just forwards input data to the results and prints it to the log. It is located in `jina/resources/executors._forward.yml`. You can also use your own [YAML](https://docs.jina.ai/chapters/yaml/yaml.html) to organize `pods`.
* Use the Flow to index an `ndarray` by calling the `index_ndarray()` API. 

Calling the `index_ndarray()` API generates requests with the following message:

```protobuf
envelope {
  receiver_id: "7b6015cb12"
  request_id: 1
  timeout: 5000
  routes {
    pod: "gateway"
    pod_id: "7b6015cb12"
    start_time {
      seconds: 1597931714
      nanos: 343076000
    }
  }
  routes {
    pod: "pod0"
    pod_id: "9cdca90c5a"
    start_time {
      seconds: 1597931714
      nanos: 347768000
    }
  }
  version {
    jina: "0.4.1"
    proto: "0.0.55"
  }
  num_part: 1
}
request {
  request_id: 1
  index {
    docs {
      id: 1
      weight: 1.0
      length: 100
      blob {
        buffer: "\004@\316\362/D\333?\244>\235\305\027\311\336?\267\210\251\311^\260\345?\366\n(\014\022m\356?\374\262\017\030\036\357\351?-c\300\337\217V\345?\241G\241\352\233\024\356?\340\346lUf\353\350?"
        shape: 8
        dtype: "float64"
      }
    }
    docs {
      id: 2
      weight: 1.0
      length: 100
      blob {
        buffer: "\312Wm\337\250\217\354?t\212\326\020\261\r\320?\254\262\300u<O\323?\340\210\222$\321\216\314?\310.q,+\347\311?&\316\361\310\252R\331?\214\016\201a\231\262\330?\342\231\262\221\343%\324?"
        shape: 8
        dtype: "float64"
      }
    }
    docs {
      id: 3
      weight: 1.0
      length: 100
      blob {
        buffer: "kT\250\372K%\345?\237\017+u\300\227\353?\3668\256\340\251\227\350?\327\006$\032$\002\341?\274\300\3573\371\262\343?\346\371\265dV\330\342?\370\210\360\002P3\340?\022i-\016\374\320\331?"
        shape: 8
        dtype: "float64"
      }
    }
  }
}
```

The structure of this message is defined in the format of [protobuf](https://docs.jina.ai/chapters/proto/docs.html). Check more details of the data structure at [`jina.proto`](https://docs.jina.ai/chapters/proto/docs.html#jina.proto). Messages are passed between the Pods in the Flow.

`envelope` and `request` are at the top of the data structure:

* `envelope` includes metadata and control data. 
* `request` contains input data and related metadata. The input is a 3*8 matrix that is sent to the Flow, which matches 3 `request.index.docs`, and the `request.index.docs.blog.shape` is 8. The vector of the matrix is stored in `request.index.docs.blob`, and the `request.index.docs.blob.dtype` indicates the type of the vector.

`search_ndarray()` is the API for searching `np.ndarray`. The data structure will be replaced from `request.index` to `request.search`, and the other nodes stay the same.

```python
import numpy as np
from jina.flow import Flow
input_data = np.random.random((3,8))
f = Flow().add(uses='_logforward')
with f:
    f.search_ndarray(input_data)
```

### 2. API for Files

`index_files()` is the API for indexing files. 

```python
from jina.flow import Flow
f = Flow().add(uses='_logforward')
with f:
    f.index_files(f'../pokedex-with-bit/pods/*.yml')
```

API `index_files()` reads input data from `../pokedex-with-bit/pods/*.yml`. In this directory, there are 5 YAML files, so you in the protobuf you can see:

* 5 `docs` under `request.index`
* Each file's path in a `request.index.doc.uri`

```protobuf
envelope {
  receiver_id: "4c5eff0d35"
  request_id: 1
  timeout: 5000
  routes {
    pod: "gateway"
    pod_id: "4c5eff0d35"
    start_time {
      seconds: 1597935647
      nanos: 100468000
    }
  }
  routes {
    pod: "pod0"
    pod_id: "b387944ecf"
    start_time {
      seconds: 1597935647
      nanos: 104758000
    }
  }
  version {
    jina: "0.4.1"
    proto: "0.0.55"
  }
  num_part: 1
}
request {
  request_id: 1
  index {
    docs {
      id: 1
      weight: 1.0
      length: 100
      uri: "../pokedex-with-bit/pods/encode-baseline.yml"
    }
    docs {
      id: 2
      weight: 1.0
      length: 100
      uri: "../pokedex-with-bit/pods/chunk.yml"
    }
    docs {
      id: 3
      weight: 1.0
      length: 100
      uri: "../pokedex-with-bit/pods/doc.yml"
    }
    docs {
      id: 4
      weight: 1.0
      length: 100
      uri: "../pokedex-with-bit/pods/encode.yml"
    }
    docs {
      id: 5
      weight: 1.0
      length: 100
      uri: "../pokedex-with-bit/pods/craft.yml"
    }
  }
}

```

`search_files()` is the API for searching `files`. 

```python
from jina.flow import Flow
f = Flow().add(uses='_logforward')
with f:
    f.search_files(f'../pokedex-with-bit/pods/chunk.yml')
```

### 3. API for Text

`index_lines()` is the API for indexing `text`. 

```python
from jina.flow import Flow
input_str = ['aaa','bbb']
f = Flow().add(uses='_logforward')
with f:
    f.index_lines(lines=input_str)
``` 

`index_lines()` reads input data from `input_str`. As you can see above, there are 2 elements in `input_str`, so in the protobuf you can see:

* 2 `docs` under `request.index.docs`
* Each individual string in `request.index.docs.text`. 

```protobuf
envelope {
  receiver_id: "e3c2f8ea9c"
  request_id: 1
  timeout: 5000
  routes {
    pod: "gateway"
    pod_id: "e3c2f8ea9c"
    start_time {
      seconds: 1597937249
      nanos: 486299000
    }
  }
  routes {
    pod: "pod0"
    pod_id: "67c4660afe"
    start_time {
      seconds: 1597937249
      nanos: 490460000
    }
  }
  version {
    jina: "0.4.1"
    proto: "0.0.55"
  }
  num_part: 1
}
request {
  request_id: 1
  index {
    docs {
      id: 1
      weight: 1.0
      length: 100
      mime_type: "text/plain"
      text: "aaa"
    }
    docs {
      id: 2
      weight: 1.0
      length: 100
      mime_type: "text/plain"
      text: "bbb"
    }
  }
}
```
`search_lines()` is the API for searching `text`. 
```python
from jina.flow import Flow
text = input('please type a sentence: ')
f = Flow().add(uses='_logforward')
with f:   
    f.search_lines(lines=[text, ])
```
## Wrap Up

Now you've got a simple Flow to index or search different data. Let's wrap up what we've covered in the demo:

`Flow` offers three different types of API for indexing/searching data:

| API                                  | Input        | Storage Location                            |
| ---                                  | ---          | ---                                         |
| `index_ndarray` and `search_ndarray` | `np.ndarray` | `request.[index.search].docs.blob`   |
| `index_files` and `search_files`     | Files        | `request.[index.search].docs.uri`         |
| `index_lines` and `search_lines`     | Text data    | `request.[index.search]docs.text`         |

## Next Steps

- Write your own Flows.
- Try more examples on `index` and `search`

| API | Links  |
| --- | --- |
|`ndarray` | [Faiss Search](https://github.com/jina-ai/examples/tree/master/faiss-search) |
|`files`  |[Flower Search](https://github.com/jina-ai/examples/tree/master/flower-search)|
|`lines` | [Southpark Search](https://github.com/jina-ai/examples/tree/master/southpark-search) |

<!---
- Check out the Docker images at Jina hub. 
--->

## Documentation 

<a href="https://docs.jina.ai/">
<img align="right" width="350px" src="https://github.com/jina-ai/jina/blob/master/.github/jina-docs.png" />
</a>

The best way to learn Jina in-depth is to read our documentation. Documentation is built on every push, merge, and release event of the master branch. For more details, check out:

- [Jina command line interface arguments explained](https://docs.jina.ai/chapters/cli/index.html)
- [Jina Python API interface](https://docs.jina.ai/api/jina.html)
- [Jina YAML syntax for executor, driver and flow](https://docs.jina.ai/chapters/yaml/yaml.html)
- [Jina Protobuf schema](https://docs.jina.ai/chapters/proto/index.html)
- [Environment variables used in Jina](https://docs.jina.ai/chapters/envs.html)
- ... [and more](https://docs.jina.ai/index.html)

## Community

- [Slack channel](https://join.slack.com/t/jina-ai/shared_invite/zt-dkl7x8p0-rVCv~3Fdc3~Dpwx7T7XG8w) - a communication platform for developers to discuss Jina
- [Community newsletter](mailto:newsletter+subscribe@jina.ai) - subscribe to the latest update, release and event news of Jina
- [LinkedIn](https://www.linkedin.com/company/jinaai/) - get to know Jina AI as a company and find job opportunities
- [![Twitter Follow](https://img.shields.io/twitter/follow/JinaAI_?label=Follow%20%40JinaAI_&style=social)](https://twitter.com/JinaAI_) - follow us and interact with us using hashtag `#JinaSearch`  
- [Company](https://jina.ai) - know more about our company, we are fully committed to open-source!



## License

Copyright (c) 2020 Jina AI Limited. All rights reserved.

Jina is licensed under the Apache License, Version 2.0. See [LICENSE](https://github.com/jina-ai/jina/blob/master/LICENSE) for the full license text.
