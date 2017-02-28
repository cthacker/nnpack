
_Work in progress. Feedback on this very welcome_

# NNPack Specification 1.0-DRAFT

## Abstract

Most modern Deep Learning frameworks provide great tools for building Neural Networks or other Machine Learning models. Unfortunately, these tools are very low-level and there are many ways to persist and use a given model.

NNPack provides a high-level structure for persisting and sharing Neural Networks. The goal is to provide a developer friendly interface for packaging and distributing Neural Networks including the data needed to create the Neural Network (Model Scaffold)

# Definitions

## 1. Model Package

### 1.1 Purpose

A _Model_ contains all information needed to load, run and operate a Machine Learning model. There is meta-data about what the model's purpose is, what the version is, what engine is required and what the expected behavior is.

### 1.2 Model Package Directory Structure

|Path               | Required? | Description                            |
|-------------------|-----------|---------------------------------------|
|`/`                | Required  | Model Package Root |
|`/nnpackage.json`  | Required  | Model Package Meta-Data File |
|`/labels.json`    | Optional  | Default Labels Definition File |
|`/state`           | Optional  | Default Model State Directory |

### 1.3 Model Package Meta-Data File: `nnpackage.json`

Example:

```json
{
  "id": "58745c9fbd17c82dd4ff7c9c",
  "name": "Scene Type",
  "description": "Classify images into Indoor VS Outdoor scenes",
  "version": 0.1,
  "engines": {
    "tensorflow": ">=0.11"
  },
  "nodes": {
    "softmaxOutput": {
      "type": "tensor",
      "id": "retrained_layer:0"
    },
    "convolutionalRepresentation": {
      "type": "tensor",
      "id": "pool_3/_reshape:0"
    }
  },
  "author": {
    "name": "Dominiek Ter Heide",
    "email": "info@dominiek.com"
  },
  "labelsDefinitionFile": "labels.json",
  "stateDir": "state"
}
```

All attribute options:

| Attribute                | Description      | Required? |
|--------------------------|------------------|-----------|
|id                        |Unique model ID, must be lowercase, no spaces   |Required   |
|name                      |Human friendly name of model |Required   |
|version                   |Version of this model and state |Required   |
|engines                   |Required engines and versions. See _1.4 Model Engines Information_ |Required   |
|description               |Description of what the model does |Optional   |
|nodes                     |Information about key nodes/layers in neural net. See _1.5 Model Nodes Information_ |Optional   |
|author                    |Author information. See _1.6 MOdel Author Information_ |Optional   |
|labelsDefinitionFile      |Path of labels definition. Defaults to `labels.json`. See _3. Label Definitions_ |Optional   |
|stateDir                  |Directory where engine keeps state of model. Defaults to `state` |Optional   |

### 1.4 Model Engines Information

The `engines` attribute is used for specifying dependencies for the model. For example, to specify Tensorflow version `1.0`:

```json
{
  "engines": {
    "tensorflow": "1.0"
  }  
}
```

The version value can have the following values:

|Version Specifier|Description|
|-----------------|-----------|
|1.0   |Exact version match|
|>=1.0 |Exact version match or above|
|<=1.0 |Exact version match or below|
|\<1.0 |Exact version above|
|\>1.0 |Exact version below|
|~1.0  |Approximate (non-strict) version match|

### 1.5 Model Nodes Information

The `nodes` attribute can be used to embed information about specific layers/nodes/tensors in your neural network. For example tensors for transfer learning, image conversion or the final softmax output layer.

Current list of standardized keys:

|Node ID        |Description|
|---------------|-----------|
|softmaxOutput                 |Softmax output layer, for example for a NN classifier|
|convolutionalRepresentation   |Representation of input after convolutional layers, before it passes through fully connected layer|

Example usage:

```json
{
  "nodes": {
    "softmaxOutput": {
      "type": "tensor",
      "id": "retrained_layer:0"
    },
    "convolutionalRepresentation": {
      "type": "tensor",
      "id": "pool_3/_reshape:0"
    }
  }
}
```

### 1.6 Model Author Information

| Attribute                | Description      |
|--------------------------|------------------|
|email                     |Author email address |
|name                      |Author full name |
|url                       |Author website |

## 2. Model Scaffold Package

### 2.1 Purpose

A _Model Scaffold_ contains all the data necessary to train a Machine Learning model. It can contain images, text, audio or other data. It has information about the different labels, where the labeled data can be found and the context of this data.

### 2.2 Model Scaffold Package Directory Structure

|Path               |Description                            |
|-------------------|---------------------------------------|
|`/`                | Model Scaffold Root |
|`/nnscaffold.json` | Model Scaffold Meta-Data File |
|`/labels.json`    | Labels Definition File |
|`/images`          | Default Images Training Data Folder |
|`/cache`           | Default Scaffold Cache Directory |

### 2.3 Model Scaffold Package Meta-Data File: `nnscaffold.json`

Example:

```json
{
  "id": "58745c9fbd17c82dd4ff7c9c",
  "name": "Scene Type",
  "version": 0.1,
  "description": "Classify images into Indoor VS Outdoor scenes",
  "author": {
    "name": "Dominiek Ter Heide",
    "email": "info@dominiek.com"
  },
  "labelsDefinitionFile": "labels.json",
  "imagesTrainingDir": "images",
  "cacheDir": "cache"
}
```

| Attribute                | Description      | Required? |
|--------------------------|------------------|-----------|
|id                        |Unique model ID, must be lowercase, no spaces   |Required   |
|name                      |Human friendly name of model |Required   |
|version                   |Version of this scaffold and its labeled data |Required   |
|description               |Description of what the model does |Optional   |
|author                    |Author information. See _1.6 Author Information_ |Optional   |
|labelsDefinitionFile      |Path of labels definition. Defaults to `labels.jsons`. See _3. Label Definitions_ |Optional   |
|imagesTrainingDir      |Path of images training dir. Defaults to `images`. See _3. Label Definitions_ |Optional   |

### 2.4 Images Training Dir

For Computer Vision an `imagesTrainingDir` which defaults to `images/` is expected. In this directory images can be stored for each Label ID (corresponding to the labels in `labels.jsons`). E.g.:

```bash
images/58745e65bd17c82ec1545a64 # Indoor images root
images/58745e65bd17c82ec1545a64/restaurant.jpg
images/58745e69bd17c82ec1545a65 # Outdoor images root
images/58745e69bd17c82ec1545a65/sunset.jpg
```

### 2.4.1 Bounding Boxes for Object Detection

For Computer Vision, bounding boxes can be specified for each set of images. This requires a file called `bounding_boxes.json`. Here's an example:

```json
{
  "images": [
    {
      "image_path": "00000000_640x480.png",
      "bounding_boxes": [
        {
          "type": "rect",
          "x1": 152,
          "x2": 167,
          "y1": 115,
          "y2": 135
        }
      ]
    }
  ]
}
```

### 2.5 Model Scaffold Cache Directory

The scaffold cache directory can be used by engines to store temporary files used in the training process. For example: the practice of generating "bottleneck" files when retraining Inception V3.

## 3. Label

A label contains information about a possible output/prediction that a model can produce. Also known as class, classification or softmax id.

### 3.1 Label Files

All labels are in a JSON format:

Example:

```json
{
  "labels": [
    {"id": "58745e65bd17c82ec1545a64", "name": "Indoor", "node_id": 0},
    {"id": "58745e69bd17c82ec1545a65", "name": "Outdoor", "node_id": 1}
  ]
}
```

### 3.2 Label Definition

In the example below, we have a label definition of name "Indoor" which represents a potential classification by the model. The `node_id` can refer to the output value of the `softmaxOutput` node. (See _1.5 Model Nodes Information_)

```json
{
  "id": "58745e65bd17c82ec1545a64",
  "name": "Indoor",
  "description": "Indoor Scene Types including buildings, rooms, fully covered patios, greenhouses, etc.",
  "node_id": 0
}
```

List of all attributes:

| Attribute                | Description      | Required? |
|--------------------------|------------------|-----------|
|id                        |Unique model ID, must be lowercase, no spaces                   |Required   |
|name                      |Human friendly name of model                                    |Required   |
|description               |Description of what the model does                              |Optional   |
|node_id                   |Output value of `softmaxOutput` for network                     |Optional   |
|level                     |Level of abstraction for hierarchical layers. See _3.3 Hierarchical Labels_ |Optional   |
|parents                   |Parent labels. See _3.3 Hierarchical Labels_ |Optional   |

### 3.3 Hierarchical Labels

Some models have labels derrived from ontologies such as Wordnet/Imagenet. Each label can include a `parents` attribute that can include an array of different labels each referring to a different label in the hierarchy.

Here's an example of a hierarchical label definition for "Killer Whale":

```json
{
  "level": 0,
  "id": "/wordnet/n02071294",
  "node_id": 22,
  "name": "killer whale, killer, orca, grampus, sea wolf, Orcinus orca",
  "parents": [
    {
      "level": 1,
      "id": "/wordnet/n2068974",
      "expanded": true,
      "name": "dolphin"
    },
    {
      "level": 2,
      "id": "/wordnet/n2066707",
      "expanded": true,
      "name": "toothed whale"
    },
    {
      "level": 3,
      "id": "/wordnet/n2062744",
      "expanded": true,
      "name": "whale"
    },
    {
      "level": 4,
      "id": "/wordnet/n2062430",
      "expanded": true,
      "name": "cetacean, cetacean mammal, blower"
    },
    {
      "level": 5,
      "id": "/wordnet/n2062017",
      "expanded": true,
      "name": "aquatic mammal"
    },
    {
      "level": 6,
      "id": "/wordnet/n1886756",
      "expanded": true,
      "name": "placental, placental mammal, eutherian, eutherian mammal"
    },
    {
      "level": 7,
      "id": "/wordnet/n1861778",
      "expanded": true,
      "name": "mammal, mammalian"
    },
    {
      "level": 8,
      "id": "/wordnet/n1471682",
      "expanded": true,
      "name": "vertebrate, craniate"
    },
    {
      "level": 9,
      "id": "/wordnet/n1466257",
      "expanded": true,
      "name": "chordate"
    }
  ]
}
```
