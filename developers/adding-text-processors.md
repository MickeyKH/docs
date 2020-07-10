---
description: >-
  This guide will show how to add a new service to the document processing
  pipeline.
---

# Adding text processors

Some deployments of Aleph may wish to implement additional processing stages for content from text documents. A common example of this is automatic translation. This can be done by adding an extra service with custom code and adding it to the document processing pipeline.

## Conceptual overview

All processing in Aleph is coordinated via a set of queues kept in redis, an in-memory data store. When a document is processed, the instruction to handle it is passed through a sequence of independent services, e.g. the `ingestors` \(which extracts text from a file\), the `analyze` stage \(which performs NER and language detection\) and finally the `index` stage \(which adds the document to Aleph's search index\).

A new stage can be added to this pipeline at runtime, either by extending the code of the aleph worker service, the ingest-file service, or by adding a new service that stands on its own. The last option is the cleanest, and can be based on `servicelayer`, the Python package that implements the queue and worker code.

## Developing the service

An example implementation of a `servicelayer`-based processing service is available in the Aleph repository at `services/translate`. The code should be short enough to study in full, especially the worker module.

{% hint style="info" %}
The translation service that is provided is a tech demo. Don't use it in production.
{% endhint %}

In order to enable the service, you need to add it to the `docker-compose.dev.yml` file, and make one of the other services depend on it. An example section is present in the file, but commented out.

{% hint style="info" %}
To run the example service, you will need to create a service account JSON key with permission to use the Google Cloud Translation API, and configure both the project ID and the path for the service account file.
{% endhint %}

Once you have added the service to the compose configuration, you can include the stage in the pipeline by adding the following variable to the `aleph.env` configuration file:

```text
ALEPH_INGEST_PIPELINE=analyze:translate
```

You may need to restart the compose environment in order for the environment changes to take effect.

During development, you can restart the container like this:

```text
docker-compose -f docker-compose.dev.yml up \
    --force-recreate --no-deps translate
```



