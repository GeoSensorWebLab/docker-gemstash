# docker-gemstash

Run [gemstash](https://github.com/bundler/gemstash) in a Docker container. Allows you to have a mirror/cache for Rubygems.org, and store local private gems.

## Install

You can build your Docker image from this repository:

    $ docker build -t geosensorweblab/gemstash:1.0.1 https://github.com/geosensorweblab/docker-gemstash

This will take a short while to download the base Ruby 2.3.1 images and set up.

I recommend you fork this repository and make changes to the gemstash configuration and Ruby versions, then build from your own repository.

## Usage

Once you have your local image, you can run gemstash in a container. Here is a quick test to make sure it works:

    $ docker run -it --rm -p 9292:9292 geosensorweblab/gemstash:1.0.1

It should print output to the screen, and if you load http://your-dockerhost:9292/ in a browser then it should redirect to https://rubygems.org. To use bundler with your gemstash on your development machine, run the [mirror command](http://bundler.io/man/bundle-config.1.html#MIRRORS-OF-GEM-SOURCES):

    $ bundle config mirror.https://rubygems.org http://your-dockerhost:9292

Then in a directory with a Gemfile, try installing a bundle. Your Docker container should output requests as Bundler accesses your cache. Note that if you are using non-HTTPS for rubygems.org then Bundler won't hit your cache. Next exit your Docker container with control-C and we will set it up to run longer-term:

    $ docker run -d -p 9292:9292 geosensorweblab/gemstash:1.0.1

The container will then run in the background, accessible on the Docker host on port 9292.

If you want to cache the gems in a local directory, a Docker volume is available for gemstash at `/gemstash/gem_cache`:

    $ docker run -d -p 9292:9292 -v /srv/gemstash/cache:/gemstash/gem_cache geosensorweblab/gemstash:1.0.1

This will allow you to destroy the container and re-create it, and not lose your gem cache in the process.

## Authentication

The above method means your gem cache will be accessible on port 9292 without any authentication. Anyone that can access that host will be able to use your gem cache. Instead, requests to the gem cache can be proxied through a server that provides [basic authentication](https://en.wikipedia.org/wiki/Basic_access_authentication).

I would recommend you publish your gemstash container's ports to localhost, then proxy requests to your container from nginx or HAProxy or similar. Once you have your proxy set up with basic authentication, you must then tell Bundler on your developer machine(s) to use that auth for access:

    $ bundle config dockerhost.example.com username:password

This must be configured for each developer.

## License

MIT License

## Authors

James Badger <jpbadger@ucalgary.ca>

## Acknowledgements

Also see [artemave/gemstash-docker](https://github.com/artemave/gemstash-docker) for a smaller implementation.
