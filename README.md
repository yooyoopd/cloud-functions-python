# cloud-functions-python
`py-cloud-fn` is a CLI tool that allows you to write and deploy [Google cloud functions](https://cloud.google.com/functions/) in pure python. No javascript allowed!

[![PyPI version](https://badge.fury.io/py/pycloudfn.svg)](https://badge.fury.io/py/pycloudfn)

[![PyPI](https://img.shields.io/pypi/dm/pycloudfn.svg)]()

Run `pip install pycloudfn` to get it.
You need to have [Google cloud SDK](https://cloud.google.com/sdk/downloads) installed, as well as
the [Cloud functions emulator](https://github.com/GoogleCloudPlatform/cloud-functions-emulator/).
Currently the emulator does not seem to work though :(

You also need **Docker** installed and running as well as the **gcloud** CLI. Docker is needed to build for the production environment, regardless of you local development environment. It's only tested on my mac and currently only for python 2.7. It should be not too difficult to make it work with python 3, pull requests welcome!

Currently, only `http`, `pubsub` and `bucket` events are supported.

# Usage
Usage is meant to be pretty idiomatic:

Run `py-cloud-fn <function_name> <trigger_type>` to build your finished function.
Run with `-h` to get some guidance on options. The library will assume that you have a file named `main.py` if not specified.

The library will create a `cloudfn` folder wherever it is used, which can safely be put in `.gitignore`. It contains build files and cache for python packages.

### Handling a http request
```python
from cloudfn.http import handle_http_event, Response


def handle_http(req):
      return Response(
        status_code=200,
        body={'key': 2},
        headers={'content-type': 'application/json'},
    )


handle_http_event(handle_http)

```

If you don't return anything, or return something different than a `cloudfn.http.Response` object, the function will return a `200 OK` with an empty body. The body can be either a string, list or dictionary, other values will be forced to a string.

### Handling a bucket event

look at the [Object](https://github.com/MartinSahlen/cloud-functions-python/blob/master/cloudfn/storage.py)
for the structure, it follows the convention in the [Storage API](https://cloud.google.com/storage/docs/json_api/v1/objects)

```python
from cloudfn.storage import handle_bucket_event
import jsonpickle


def bucket_handler(obj):
    print jsonpickle.encode(obj)


handle_bucket_event(bucket_handler)
```

### Handling a pubsub message

look at the [Message](https://github.com/MartinSahlen/cloud-functions-python/blob/master/cloudfn/pubsub.py)
for the structure, it follows the convention in the [Pubsub API](https://cloud.google.com/pubsub/docs/reference/rest/v1/PubsubMessage)

```python
from cloudfn.pubsub import handle_pubsub_event
import jsonpickle


def pubsub_handler(message):
    print jsonpickle.encode(message)


handle_pubsub_event(pubsub_handler)
```

## Deploying a function
I have previously built [go-cloud-fn](https://github.com/MartinSahlen/go-cloud-fn/), in which there is a complete CLI available for you to deploy a function. I did not want to go there now, but rather be concerned about `building` the function and be super light weight. Deploying a function can be done like this:

```sh
py-cloud-fn my-function http --production && \
cd cloudfn/target && gcloud beta functions deploy my-function \
--trigger-http --stage-bucket <bucket> && cd ../..
```


## License

Copyright © 2017 Martin Sahlen

Distributed under the MIT License
