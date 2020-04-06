## How to speed up your next js app (ssr) using GZip

Recently I was working on one of my projects built using nextjs . I was using semantic UI library. Even the minified version is too big for production. (631kb)  It was making my web app slow.  I searched through the internet for ways to reduce the size. Then I found out about gzip. I can encode my web app using gzip which reduced the size of the app significantly. Let’s see how to do it.

## What is needed on the prod server?

We need a reverse proxy. I was using NGINX. 


## Steps
1. Configure webpack to create gzip files.
2. Configure NGINX to serve gzip content.

### Step 1
1. We need to install a package   _compression-webpack-plugin_.
2. Go to next.config.js and add this plugin.

```javascript
 config.plugins.push(
      new CompressionPlugin({
        test: /\.js$|\.css$|\.html$/,
      }),
    );
```

The whole file will look like this

```javascript
const withSass = require(‘@zeit/next-sass’);
const withCSS = require(‘@zeit/next-css’);
const CompressionPlugin  = require(‘compression-webpack-plugin’)
const nextConfig = {
  onDemandEntries: {
    // period (in ms) where the server will keep pages in the buffer
    maxInactiveAge: 50 * 1000,
    // number of pages that should be kept simultaneously without being disposed
    pagesBufferLength: 5,
  }
}
module.exports = {
  …withCSS(withSass({
  target: ‘serverless’,
  webpack (config) {
    config.module.rules.push({
      test: /\.(png|svg|eot|otf|ttf|woff|woff2)$/,
      use: {
        loader: ‘url-loader’,
        options: {
          limit: 8192,
          publicPath: ‘/_next/static/‘,
          outputPath: ‘static/‘,
          name: '[name].[ext]'
        }
      }
    })
    config.plugins.push(
      new CompressionPlugin({
        test: /\.js$|\.css$|\.html$/,
      }),
    );
    return config;
  }
})
),
...nextConfig
}

```

3. Now you will see when you build your project, you will get _bundle.js.gz_ 
4. Now you only have to configure your NGINX Reverse proxy to serve .gz file first.

### Step 2

1. Run vim_etc_nginx_sites-available_default .
2. Add the below code in your server.

```bash
        # gzip config
        gzip on;
        gzip_static on;
        gzip_types text/plain text/css application/json 	application/x-javascript text/xml application/xml application/xml+rss text/javascript;
        gzip_proxied  any;
        gzip_vary on;
        gzip_comp_level 6;
        gzip_buffers 16 8k;
        gzip_http_version 1.1;
        #
```

3. Now restart NGINX `sudo systemctl restart nginx`.
4. Thats it you are done !