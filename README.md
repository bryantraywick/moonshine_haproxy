# Moonshine Haproxy

### A plugin for [Moonshine](http://github.com/railsmachine/moonshine)

A plugin for installing and managing HAProxy. It can also manage Apache as an SSL proxy for a HAProxy backend.

### Instructions

* <tt>script/plugin install git://github.com/railsmachine/moonshine_haproxy.git</tt>
* Configure settings if needed
    configure(:haproxy => {:foo => true})
* Invoke the recipe(s) in your Moonshine manifest
    recipe :haproxy

### Example Configuration in moonshine.yml

    :haproxy:
      :default_backend: apps
      :backends:
        - {:name: apps, :balance: roundrobin, :servers: [{:url: app1.example.com:80, :name: app1, :maxconn: 1000}, {:url: app2.example.com:80, :name: app2, :maxconn: 1000}]}


    # If :ssl is present, SSL through Apache, parodying to HAPrroxy, will automatically be setup
    :ssl:
      :certificate_file: /srv/example/shared/config/ssl/example.com.crt
      :certificate_key_file: /srv/example/shared/config/ssl/example.com.key

### Notes about apache and apache2ctl status

'apache2ctl status' normally pulls from http://localhost/server-status?auto for its data. This would be served by haproxy though, and would be forwarded on to the configured backends. Hitting https wouldn't work either, since it's just forwarding to haproxy, which would forward to the backends.

To deal with this, moonshine_haproxy configures apache to listen on port 81 on localhost only. This means you can hit http://localhost:81/server-status?auto . 'apache2ctl status' is smart enough to do this by default.

### Custom Error Pages

moonshine_haproxy now has support for custom error pages!  If you create files in your app's public directory named 400.html (or 403, 408, 503 or 504).html, the plugin will pick them up on deploy and turn them into custom errorfiles and use them instead of the default (ugly) responses.  

The only gotcha with those pages is that they shouldn't use any external assets served from the app (images, css, javascript) - since they're served directly by HAProxy when something is wrong with the app servers.  If you need to display images or use other external assets, they should be hosted somewhere like S3.

### Maintenance mode

moonshine_haproxy support's capistrano's `deploy:web:disable` for putting the application into maintenance mode:

    # disable production
    cap production depoy:web:disable
    # disable production with reason and until
    cap production depoy:web:disable REASON="server maintenance" UNTIL="3AM"
    cap production depoy:web:enable

These cap tasks were moved out of the core capistrano gem into {capistrano-maintenance}[https://github.com/tvdeyen/capistrano-maintenance], so you may need to include that depending on your capistrano version.

### HAProxy 1.5 and SSL Support

moonshine_haproxy supports haproxy 1.5 and SSL, but you need to make quite a few changes to your configuration to replace the Apache SSL proxy and get SSL working with haproxy.  The first is adding the following to your moonshine.yml:

    :haproxy:
      :version: 1.5-dev22
      :use_ssl: true
      :keep_alive: true
  
You'll also need a new frontend for SSL things, and the bind line needs to look like this:

    bind 0.0.0.0:443 ssl crt /path/to/your/cert.pem

And your cert should be a pem with your key and certificate combined in a single file.

Deploying with that configuration will disable the SSL VirtualHost in Apache and set HAProxy up to serve SSL requests!

***

Unless otherwise specified, all content copyright &copy; 2014, [Rails Machine, LLC](http://railsmachine.com)
