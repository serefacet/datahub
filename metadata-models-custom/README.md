# A Custom Metadata Model

This module hosts a gradle project where you can store your custom metadata model. It contains an example extension for you to follow.

### Caveats

Currently, this project only supports adding new aspects to existing entities. You cannot add new entities to the metadata model yet.

## Pre-Requisites

Before proceeding further, make sure you understand the DataHub Metadata Model concepts defined [here](docs/modeling/metadata-model.md) and extending the model defined [here](docs/modeling/extending-the-metadata-model.md). 

## Create your new aspect(s)

Follow the regular process in creating a new aspect by adding it to the [`src/main/pegasus`](metadata-models-custom/src/main/pegasus) folder. e.g. This repository has an Aspect called `customDataQualityRules` hosted in the [`DataQualityRules.pdl`](metadata-models-custom/src/main/pegasus/com/mycompany/dq/DataQualityRules.pdl) file that you can follow.
Feel free to delete the sample aspects that are stored in this repo using a simple `rm -rf src/main/pegasus/*`

## Add your aspect(s) to the entity registry

Add your new aspect(s) to the entity registry by editing the yaml file located under `registry/entity-registry.yaml`.
**_Note_**: The registry file must be called `entity-registry.yaml` or `entity-registry.yml` for it to be recognized.

## Understanding the entity registry

Here is a sample entity-registry file
```yaml
id: mycompany-dq-model
entities:
  - name: dataset
    aspects:
      - customDataQualityRules
```

The entity registry has a few important fields to pay attention to: 
- id: this is the name of your registry. This drives naming, artifact generation, so make sure you pick a unique name that will not conflict with other names you might create for other registries. 
- entities: this is a list of entities with aspects attached to them that you are creating additional aspects for. In this example, we are adding the aspect `customDataQualityRules` to the `dataset` entity. 

## Build your new model 

```
../gradlew build
```

### Build a versioned artifact
```
../gradlew -PprojVersion=0.0.1 build
```

This will deposit an artifact called `metadata-models-custom-<version>.zip` under the `build/dist` directory. 

### Deploy your versioned artifact to DataHub

```
../gradlew -PprojVersion=0.0.1 install
```

This will unpack the artifact and deposit it under `~/.datahub/plugins/models/<registry-name>/<registry-version>/`. 

### Check if your model got loaded successfully 

Assuming that you are running DataHub on localhost, you can curl the config endpoint to see the model load status. 

```
curl -s http://localhost:8080/config | jq .
```

```
{
  "models": {
    "mycompany-dq-model": {
      "0.0.1": {
        "loadResult": "SUCCESS",
        "registryLocation": "/Users/username/.datahub/plugins/models/mycompany-dq-model/0.0.1",
        "failureCount": 0
      }
    }
  },
  "noCode": "true"
}
```

### Add some metadata with your new model 

We have included some sample scripts that you can modify to upload data corresponding to your new data model. 
The `scripts/insert_one.sh` script takes the `scripts/data/dq_rule.json` file and attaches it to the `dataset_urn` entity using the `datahub` cli. 

```console
cd scripts
./insert_one.sh
```
results in 
```console
Update succeeded with status 200
```


## Advanced Guide

A few things that you will likely do as you start creating new models and creating metadata that conforms to those models. 

### Deleting metadata associated with a model 

The `datahub` cli supports deleting metadata associated with a model as a customization of the `delete` command. 

e.g. `datahub delete --registry-id=mycompany-dq-model:0.0.1` will delete all data written using this registry name and version pair. 

### Evolve the metadata model

As you evolve the metadata model, you can publish new versions of the repository and deploy it into DataHub as well using the same steps outlined above. DataHub will check whether your new models are backwards compatible with the previous versioned model and decline loading models that are backwards incompatible. 

## The Future

Hopefully this repository shows you how easily you can extend and customize DataHub's metadata model. 
We will be adding support for adding new entities soon, and programmatically generating Python classes to make it even easier to interact with the extended metadata model. 




