# OCI Image Resource helper

This is a helper for working with OCI image resources in the charm operator
framework.

## Installation

Add it to your `requirements.txt`.  Since it's not in PyPI, you'll need to use
the GitHub archive URL (or `git+` URL, if you want to pin to a specific commit):

```
https://github.com/juju-solutions/resource-oci-image/archive/master.zip
```

or 

```
git+https://github.com/juju-solutions/resource-oci-image#egg=oci-image
```

## Usage

The `OCIImageResource` class will wrap the framework resource for the given
resource name, and calling `fetch` on it will either return the image info
or raise an `OCIImageResourceError` if it can't fetch or parse the image
info. The exception will have a `status` attribute you can use directly,
or a `status_message` attribute if you just want that.

Example usage:

```python
from ops.charm import CharmBase
from ops.main import main
from oci_image import OCIImageResource, OCIImageResourceError


class MyCharm(CharmBase):
    def __init__(self, *args):
        super().__init__(*args)
        self.image = OCIImageResource(self, 'resource-name')
        self.framework.observe(self.on.start, self.on_start)

    def on_start(self, event):
        try:
            image_info = self.image.fetch()
        except OCIImageResourceError as e:
            self.model.unit.status = e.status
            event.defer()
            return

        self.model.pod.set_spec({'containers': [{
            'name': 'my-charm',
            'imageDetails': image_info,
        }]})


if __name__ == "__main__":
    main(MyCharm)
```

## Developing

Create and activate a virtualenv with the development requirements:

    virtualenv -p python3 venv
    source venv/bin/activate
    pip install -r requirements-dev.txt

## Running tests

```
bash run_tests
```