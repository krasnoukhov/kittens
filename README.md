Kittens
=======

A simple django application to show a random image fetched from Flickr resized to a specified size.

`/kittens/x/y/js` gets javascript to show one of the locally cached images in x pixels witdh and y pixels height. This is the preferred way to show pictures you got from Flickr because it creates a link to the Flickr page of the original image.

`/kittens/x/y` gets one of the locally cached images in x pixels witdh and y pixels height directly. Flickr author ID and image ID get encoded into the redirected image URL, so this also conforms with Creative Commons license requirement for attribution.

`/kittens/fetch` fetches a new image from Flickr and stores it in local cache.

You can see live demo at http://theoldreader.com/kittens/

Installation
============

It's quite easy to launch the app in Debian/Ubuntu environment using gunirorn and nginx:

    mkdir -p /var/www/kittens && git clone git@github.com:knyar/kittens.git /var/www/kittens/current
    aptitude install -R python-imaging python-virtualenv gunicorn
    virtualenv --system-site-packages /var/www/kittens/virtualenv
    /var/www/kittens/virtualenv/bin/pip install -r /var/www/kittens/current/dependencies.txt
    mkdir -p /var/www/kittens/current/files/images && chown www-data /var/www/kittens/current/files/images
    echo "FLICKR_API_KEY = 'xxxxxxxxxxxxxxx'" > /var/www/kittens/current/project/local_settings.py
    cp /var/www/kittens/current/project/gunicorn-d.conf /etc/gunicorn.d/kittens
    service gunicorn restart

Then add the following to your nginx virtualhost:

    location /kittens/ {
      proxy_pass        http://127.0.0.1:8000;
      proxy_redirect     off;
      proxy_set_header   Host             $host;
      proxy_set_header   X-Forwarded-For  $proxy_add_x_forwarded_for;
      proxy_set_header   X-Forwarded-Proto $scheme;
    }
    location /kittens/files {
      alias /var/www/kittens/current/files;
      expires epoch;
      add_header Last-Modified "";
    }

By default new image is fetched from Flickr when somebody visits the index page. However, you can add `curl http://localhost:8000/kittens/fetch > /dev/null` to your crontab to add new images regularly.

