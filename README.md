wheelhouse-uploader
===================

Script to help maintain wheelhouse folders on cloud storage containers such as
Amazon S3, Rackspace Cloud Files, Google Storage or Azure Storage.

## Installation

    pip install wheelhouse-uploader

## Usage

The canonical use case is:

1- Continuous Integration (CI) servers build and test the project packages for
   various platforms and versions of Python, for instance using the commands:

        pip install wheel
        python setup.py bdist_wheel

2- All CI servers upload the resulting packages using `wheelhouse-uploader`
   to upload the generated artifacts to one or more cloud storage containers
   (e.g. one container per platform, or one for the master branch and the other
   for release tags).

3- The project maintainer uses the `wheelhouse-uploader` distutils extensions
   to fetch all the generated build artifacts for a specific version number to
   its local `dist` folder and upload them all at once to PyPI when
   making a release.


### Uploading artifact to a cloud storage container

Use the following command:

    python -m wheelhouse_uploader upload \
        --username=mycloudaccountid --secret=xxx \
        --local-folder=dist/ my_wheelhouse

or:

    export WHEELHOUSE_UPLOADER_USERNAME=mycloudaccountid
    export WHEELHOUSE_UPLOADER_SECRET=xxx
    python -m wheelhouse_uploader upload --local-folder dist/ my_wheelhouse

When used in a CI setup such as http://travis-ci.org or http://appveyor.com,
the environment variables are typically configured in the CI configuration
files such as `.travis.yml` or `appveyor.yml`. The secret API key is typically
encrypted and exposed with a `secure:` prefix in those files.

The files in the `dist/` folder will be uploaded to a container named
`my_wheelhouse` on the `CLOUDFILES` (Rackspace) cloud storage provider.

You can pass a custom `--provider` param to select the cloud storage from
the list of [supported providers](
https://libcloud.readthedocs.org/en/latest/storage/supported_providers.html).

Assuming the container will be published as a static website using the cloud
provider CDN options, the `upload` command also maintains an `index.html` file
with links to all the files previously uploaded to the container.

It is recommended to configure the container CDN cache TTL to a shorter than
usual duration such as 15 minutes to be able to quickly perform a release once
all artifacts have been uploaded by the CI servers.


### Fetching artifacts manually

The following command downloads items that have been previously published to a
web page with an index with HTML links to the project files:

    python -m wheelhouse_uploader fetch \
        --version=X.Y.Z --local-folder=dist/ \
        project-name http://wheelhouse.example.org/


### Uploading previously archived artifacts to PyPI

Ensure that the `setup.py` file of the project registers the
`wheelhouse-uploader` distutils extensions:

    try:
        # Only used by the release manager of the project
        import wheelhouse_uploader.cmd
        cmdclass = vars(wheelhouse_uploader.cmd)
    except ImportError:
        cmdclass = {}
    ...

    setup(
        ...
        cmdclass=vars(wheelhouse_uploader.cmd)
    )

Put the URL of the public artifact repositories populated by the CI workers:

    [wheelhouse_uploader]
    artifact_indexes=
          http://wheelhouse.site1.org/
          http://wheelhouse.site2.org/


Fetch all the artifacts matching the current version of the project as
configured in the local `setup.py` file and upload them all to PyPI:

    python setup.py fetch_artifacts upload_all

Note: this will reuse PyPI credentials stored in `$HOME/.pypirc` if
`python setup.py register` or `upload` were called previously.


### TODO

- test on as many cloud storage providers as possible (please send an email to
  olivier.grisel@ensta.org if you can make it work on a non-Rackspace provider),
- check that CDN activation works everywhere (it's failing on Rackspace
  currently: need to investigate) otherwise the workaround is to enable CDN
  manually in the management web UI,
- make it possible to fetch private artifacts using the cloud storage protocol
  instead of HTML index pages.
