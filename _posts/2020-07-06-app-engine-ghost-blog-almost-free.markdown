---
layout: post
title: Ghost 3.x on Google App Engine Standard with Cloudinary and Secret Manager
image: google-free-tier.png
date:  2020-07-06 12:00:00
description: "Technology and Health are my passions. It is my life goal to conquer diabetes
              using technology. It has been twenty-four years since I wrote my first computer program on an old computer running GW-BASIC. Today, I no longer open my shoes outside my computer room to protect it from viruses, but I am always fascinated by programming as writing first \"print 10+20\". Since last December, I have been trying to document my triumphs
              and defeats over diabetes in a structured way. I want to create the easiest, best, fastest, and cheapest tool to control diabetes. After limited research, I zeroed in on ghost blogs
              hosted on google cloud to document my journey. However, there is no single source of
              truth that tells me correctly how to do it."  
---
## What to Build?

My thirteen years of diabetes has told me in no uncertain terms that medicine alone is not enough to control diabetes. I wanted to build a product to use technology to help manage diabetes.
I have a strong bias in building products with early adopters. To this end, I decided to make a website
to chronicle the product journey. This product was going to be a dense content blog.
I am a fan of Medium as a product. There is something about the simplicity of the entire experience.
However, Medium has long disabled the ability of mere mortals to host custom domains. Hence, I did the next best thing I googled "medium vs.." The ghost team had done an excellent job on SEO. It met most of my criteria. 

## How to host?

Having decided on "Ghost," the question was where to host to achieve the cheapest costs. In November 2019, it invariably
meant determining between which cloud to use. As a software engineer, I hate to pay for software. Of Google, AWS and Azure, only
Google had a perpetual free tier. Google was also distributing 300$ of free credit. So, I decided to go to google.

### Step 1: The Journey Starts with a google search.

Programming today is ninety percent knowing what to google. I started my journey with the search for *"How to host ghost CMS on AppEngine?"*
![Ghost Hosting Google Results](/assets/img/ghost-google-res.png "Ghost Hosting Result")

My first preference was to follow the default guide of the app engine. However, it severely restricts the ability to customize the installation. I discarded the approach. It did highlight a few issues.

* Default Ghost uses local file storage. It is something that is not practical in app-engine.
* The tutorial mentioned something about costs. Unfortunately, I did not pay much attention to it. I would regret it later. (
AppEngine is free only for the standard edition. All tutorials were for flex; this was not free and way too expensive for a 
hobby project.)

### Challenge 1: MySql Socket

I wanted to use Cloud SQL as a database. The recommended way to connect to cloud SQL from App Engine is via a Unix socket. At that time,
I was new to Unix socket, and most of the documentation was around connecting to TCP MySQL. It took me some time to realize that
I need to use the environment variable `database__connection__socketPath` instead of `database__connection__host` and 
`database__connection__port`.  

### Challenge 2: Using Cloudinary for images
I love Cloudinary to host images. It helps that they have a free tier in addition to amazing features. I was encouraged by
an official plugin. There is a quirk in ghost repository wherein adapters cannot be installed as npm plugins. Rather, one has to
move the entire plugin to the installation folder. I found this strange, but I shouldered on. 
```bash
$ yarn add ghost-storage-cloudinary@2
$ mv node_modules/ghost-storage-cloudinary core/server/adapters/storage
```
The above commands have been lifted verbatim from the official repository. At this point, I had decided to go with docker deployment, and the plugin came with a  public docker repository.

### Challenge 3: Masquerade mailing list feature for research

The whole reason behind creating this blog was to find early adopters of the platform. I decided to use the default subscribe
feature of Ghost. I also chose to use Mailgun as my backend for the mailing list. I had to do some minor edits to a few
templates and CSS. I was up and away with less than a hundred lines of code. I started documenting my battles. 



### Challenge 4: Environment Variables and Cost

In my excitement, I had missed reading the fine print of the free tier—the cheapest flex app engine costs over sixty dollars per month.
I had also punted on secret environment variables. At this moment my Dockerfile looked like

```docker
FROM ghost:3.0.3-alpine as cloudinary
WORKDIR $GHOST_INSTALL/current
ADD subscribe-form.hbs $GHOST_INSTALL/current/content/themes/casper/partials/subscribe-form.hbs
RUN chown node:node $GHOST_INSTALL/current/content/themes/casper/partials/subscribe-form.hbs  
ADD default.hbs $GHOST_INSTALL/current/content/themes/casper/default.hbs
RUN chown node:node $GHOST_INSTALL/current/content/themes/casper/default.hbs  
ADD site-nav.hbs $GHOST_INSTALL/current/content/themes/casper/partials/site-nav.hbs
RUN chown node:node $GHOST_INSTALL/current/content/themes/casper/partials/site-nav.hbs 
ADD screen.css   $GHOST_INSTALL/current/content/themes/casper/assets/built/screen.css 
RUN chown node:node $GHOST_INSTALL/current/content/themes/casper/assets/built/screen.css 
ADD screen.css   $GHOST_INSTALL/current/content/themes/casper/assets/built/screen.css.map
RUN chown node:node $GHOST_INSTALL/current/content/themes/casper/assets/built/screen.css.map 
RUN su-exec node yarn add ghost-storage-cloudinary@2
 
FROM ghost:3.0.3-alpine
COPY --chown=node:node --from=cloudinary $GHOST_INSTALL/current/node_modules $GHOST_INSTALL/current/node_modules
COPY --chown=node:node --from=cloudinary $GHOST_INSTALL/current/node_modules/ghost-storage-cloudinary $GHOST_INSTALL/current/core/server/adapters/storage/ghostStorageCloudinary
COPY --chown=node:node --from=cloudinary $GHOST_INSTALL/current/content/themes/casper/partials/subscribe-form.hbs $GHOST_INSTALL/current/content/themes/casper/partials/subscribe-form.hbs
COPY --chown=node:node --from=cloudinary $GHOST_INSTALL/current/content/themes/casper/default.hbs $GHOST_INSTALL/current/content/themes/casper/default.hbs
COPY --chown=node:node --from=cloudinary $GHOST_INSTALL/current/content/themes/casper/partials/site-nav.hbs $GHOST_INSTALL/current/content/themes/casper/partials/site-nav.hbs
COPY --chown=node:node --from=cloudinary $GHOST_INSTALL/current/content/themes/casper/assets/built/screen.css  $GHOST_INSTALL/current/content/themes/casper/assets/built/screen.css
COPY --chown=node:node --from=cloudinary $GHOST_INSTALL/current/content/themes/casper/assets/built/screen.css.map  $GHOST_INSTALL/current/content/themes/casper/assets/built/screen.css.map


RUN set -ex; \
    su-exec node ghost config storage.active ghost-storage-cloudinary; \
    su-exec node ghost config storage.ghostStorageCloudinary.upload.use_filename false; \
    su-exec node ghost config storage.ghostStorageCloudinary.upload.unique_filename true; \
    su-exec node ghost config storage.ghostStorageCloudinary.upload.overwrite false; \
    su-exec node ghost config storage.ghostStorageCloudinary.fetch.quality auto; \
    su-exec node ghost config storage.ghostStorageCloudinary.fetch.secure true; 
```

My app.yaml looked like
```yaml
env_variables:
  database__client: mysql
  url: "https://www.project1922.com"
  database__connection__user: "<hardcoded_user>"
  database__connection__password: "<hard_coded password>"
  database__connection__database: "ghost"
  database__connection__socketPath: "/cloudsql/project1922-ghost-blog:us-central1:project-1922-ghost-db"
  mail__transport: "SMTP"
  mail__options__service: "Maligun"
  mail__options__port: 465
  mail__options__host: smtp.mailgun.org
  mail__options__secureConnection: true
  mail__options__auth__user: postmaster@mg.project1922.com
  mail__options__auth__pass: <hardcoded_password>
  storage__active: "ghostStorageCloudinary"
  storage__ghostStorageCloudinary__auth__cloud_name: project1922-cloudinary-images
  storage__ghostStorageCloudinary__auth__api_key: <hard_coded>
  storage__ghostStorageCloudinary__auth__api_secret: <hard_coded>
  # The connection name of your instance on its Overview page in the Google
  # Cloud Platform Console, or use `YOUR_PROJECT_ID:YOUR_REGION:YOUR_INSTANCE_NAME`
```
It was a ticking time bomb. 

#### Strike 1: Wrong files got pushed 
One day, I pushed my app.yaml to a public repo, and my secret keys got exposed. Luckily,
MailGun monitors for exposures. On being alerted, I took evasive actions. I changed the keys and made the repository private.


#### Strike 2: Ran out of free google credits.

Five months elapsed, the blog was running in maintenance mode. Unknown to me, I was burning through free credits, and then I
saw a charge on my credit card. I was alarmed due to past incidents of exposed MailGun keys. I punted for one month. After two months, I decided to fix things. I found that free is only for the standard instance. I decided to move to the standard installation.

### Challenge 5: Let the Source be with you

Ghost documentation actively discourages building from the source.  
![Ghost Documentation ScreenShot](/assets/img/ghost-web.png "Ghost Documentation")

They have an excellent CLI tool to install the blog. However, I did not quite like black magic. I thought of installing
as an npm module and overriding the template files inside the node_modules as part of the  startup.
It worked for my localhost but would not work in the app engine. App Engine uses a read-only file system for code. So, I decided
to go with the source. By this time, I had familiarized myself with essential files and what the CLI did.  






### Challenge 6: What's in a (variable) name?

I followed documentation of *Install from Source* and managed to get the blog running locally. Now, it was time
to go for the app engine and slay the cost monster.  However, the app engine would not budge easily. It ran into issues with the node version. It was easy to fix. ```"node": "12.X.X",``` instead of ```"node": "^10.13.0 || ^12.10.0",``` fixed this. 
However, the blog was not reachable on the internet. Debug logs showed nothing. All I got was a warning 
*"App is listening on port 8080. We recommend your App listen on the port defined by the
 PORT environment variable to take advantage of an NGINX layer on port 8080"*
. All things pointed towards a PORT issue. I was running my App on port 8080, the default app for the app engine. The way ghost works are that
it listens on a port defined by the environment variable server__port. GAE had changed the behavior of the app engine to expose this variable as
"PORT".  I had to change the behavior of the ghost code to listen to this variable. I checked my hypothesis by adding the lines
```process.env.PORT || config.get('server').port,``` to *ghost-server.js*. Voila!! It worked. I did not want to change the internal implementation of 
Ghost. I decided to change the main index.js file.

```javascript
if(process.env.server__port || process.env.PORT){
    process.env.server__port = process.env.server__port || process.env.PORT ;; //Required for GAE
}
```

It worked, and the blog was working again with reduced costs. The PORT battles led me to believe that I could change default index.js to make the file
compatible with GAE. It will also allow me to take advantage of future changes to Ghost.

### Challenge 7: Secret Environment Variables

All my life, I had used environment variables to store secrets. This architecture left me with problems like exposed private keys. I thought of
solving this problem for my hobby project. I had already bought myself into the idea of injecting environment variables as part of
my startup script. Google had recently introduced "Secret Manager", it seemed like the recommended way to store secrets. I could
fetch the required environment variables as a startup. I could use IAM credentials to restrict access to the keys via service accounts.
As with everything new, there were a few things I had to learn my way.
* First, create secrets using UI.
* Give the default AppEngine service account access to access the secret keys.
* Every secret has version and secret value has to be fetched as payload of enabled version.
```javascript
//Final js code1
if(process.env.server__port || process.env.PORT){
    process.env.server__port = process.env.server__port || process.env.PORT ;; //Required for GA
}
const ReadSecret = require('./utils');

// This is what listen gets called on, it needs to be a full Express App
async function startUp(){
    // Use the request handler at the top level
// @TODO: decide if this should be here or in parent App - should it come after request id mw?
    const secretKeys= process.env.secretKeys ||
        "database__connection__user," +
        "database__connection__password," +
        "mail__options__auth__user," +
        "mail__options__auth__pass," +
        "storage__ghostStorageCloudinary__auth__api_key," +
        "storage__ghostStorageCloudinary__auth__api_secret";

    const secretKeysArr = secretKeys.split(",");
    if((process.env.GOOGLE_CLOUD_PROJECT)!=null){
        const project = process.env.GOOGLE_CLOUD_PROJECT;
        for(key of secretKeysArr){
            process.env[key] = await ReadSecret.readSecret(`projects/${project}/secrets/${key}/versions/1`);
        }
    }
    const startTime = Date.now();
    const debug = require('ghost-ignition').debug('boot:index');
// Sentry must be initialised early on
    const sentry = require('./core/shared/sentry');

    debug('First requires...');

    const ghost = require('./core');

    debug('Required ghost');

    const express = require('./core/shared/express');
    const logging = require('./core/shared/logging');
    const urlService = require('./core/frontend/services/url');
    const ghostApp = express('ghost');

    ghostApp.use(sentry.requestHandler);

    debug('Initialising Ghost');

    ghost().then(function (ghostServer) {
        // Mount our Ghost instance on our desired subdirectory path if it exists.
        ghostApp.use(urlService.utils.getSubdir(), ghostServer.rootApp);

        debug('Starting Ghost');
        // Let Ghost handle starting our server instance.
        return ghostServer.start(ghostApp)
            .then(function afterStart() {
                logging.info('Ghost boot', (Date.now() - startTime) / 1000 + 's');
            });
    }).catch(function (err) {
        logging.error(err);
        setTimeout(() => {
            process.exit(-1);
        }, 100);
    });
};
startUp();

```

```javascript
async function readSecret(name ) {

        // Imports the Secret Manager library
    const {SecretManagerServiceClient} = require('@google-cloud/secret-manager');

    // Instantiates a client
    const client = new SecretManagerServiceClient();

    async function getSecret() {
        const [version] = await client.accessSecretVersion({
            name: name,
        });
        const payload = version.payload.data.toString();

        return payload;
    }

    return getSecret();
    // [END secretmanager_get_secret]
};


module.exports = {readSecret:readSecret};
```
```yaml
runtime: nodejs12
env_variables:
  database__client: mysql
  url: "https://www.project1922.com"
  database__connection__database: ghost
  database__connection__socketPath: "/cloudsql/project1922-ghost-blog:us-central1:project-1922-ghost-db"
  mail__transport: "SMTP"
  mail__options__service: "Maligun"
  mail__options__port: 465
  mail__options__host: smtp.mailgun.org
  mail__options__secureConnection: true
```

It all worked, and I had a clean repository without any hardcoded secret variables.

## PostScript
When writing this post, I came across another post on Ghost 3.0. https://savviest.com/blog/hosting-a-ghost-3-0-blog-on-google-app-engine/. This tutorial was not available when I had started working on my first version and seems like a great option. 

### Why is it almost free?

* Cloud SQL is not free, and at its cheapest, it costs about 7 $ per month. Potentially, I could host F1 micro compute engine to run MySQL but
I did not want to deal with it.
* Secret Manager. It should cost less than 1 $ per month.



### References
* https://cloud.google.com/community/tutorials/ghost-on-app-engine-part-1-deploying
* https://ghost.org/docs/install/source/
* https://cloud.google.com/secret-manager/docs
* https://cloud.google.com/free/
* https://www.project1922.com




 

