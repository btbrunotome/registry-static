static npm registry
-------------------

This module follows the `skimdb` from npm and creates a flat file copy of the
registry on your local machine. This can then be served up behind `nginx` to
give you a local, read-only copy of the npm registry.


usage
-----

    registry-static -d my.registry.com -o /var/www/registry

This is the most common use, this will start following the registry
and make copies of the modules and their tarballs into `/var/www/registry`.
When it does, it will replace the `tarball` key in the `package.json`
with the url provided with `-d` (so the downloads resolve locally).

This module also uses the sequence file, so you can kill it and it should
restart from where it left off.

_WARNING: This may take quite a while to complete and a large amount of disk space_

directory structure
-------------------

When it pulls the `package.json` for a module, it will create a directory structure similar to this:

    ./davargs
    ├── -
    │   ├── davargs-0.0.1.tgz
    │   └── davargs-0.0.2.tgz
    ├── 0.0.1
    │   └── index.json
    ├── 0.0.2
    │   └── index.json
    ├── index.json
    └── latest
        └── index.json

This allows for the following url, styles to work:

    my.registry.com/davargs
    my.registry.com/davargs/0.0.1
    my.registry.com/davargs/0.0.2
    my.registry.com/davargs/latest



nginx configuration
-------------------

    server {
        listen       9000;
        server_name  localhost;
        charset utf-8;
        location / {
            root   /var/www/registry/;
            index  index.json;
        }
        error_page  404              /404.json;
    }

logic
-----

First, no files are ever deleted. The reference to the tarball may be removed from the local `package.json` but
the tarball itself is not removed. This is for things like `npm shrinkwrap`.

Each download is verified against the `shasum` in the package.json. If the verification fails, the file is retried
up to 4 times. If it fails all of those, it is skipped and not stored locally.

Each `change` request will process the entire module, not just the change alone. This is to make sure that tags
and new versions are all in sync.

