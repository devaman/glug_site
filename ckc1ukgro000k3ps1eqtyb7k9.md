## AutoDeploy your projects using Github WebHooks

Hey Everyone,

Many of us has this problem of deploying our github repo to EC2 machines or Digital Ocean's droplets or etc. We can automate this process by running a simple webhook script on our machine.

Lets start !

# The script

```js
const secret = "secret-from-github";
const repo = "path-to-repo-on-machine(eg ~/react-project)";
const http = require('http');
const crypto = require('crypto');
const exec = require('child_process').exec;
const child = require('child_process');
http.createServer(function (req, res) {
        let data= []

        req.on('data', function(chunk) {
                data.push(chunk);
        });

        req.on('end', () => {
                let sig = "sha1=" + crypto.createHmac('sha1', secret).update(data.toString()).digest('hex');
                if (req.headers['x-hub-signature'] == sig) {
                        if(JSON.parse(data).ref==='refs/heads/master'){
                                console.log('Deploying commit - ',JSON.parse(data).head_commit.message)
                                exec('cd ' + repo + ' && git pull origin master && npm install && npm run build && pm2 start npm -- start');
                        }
                }

        })

    res.end();
}).listen(8080);
```
- This script is first generating a sha signature using sceret and verifiying the request.
- If request signature matches our generated signature then we our parsing the payload to JSON.
- In this script we are receiving all the events that are being generated on our Github repo like commit , pull request, merge.
- I have a develop branch and a master branch.
- I am triggering the build when an event occurs on a master branch. ```if(JSON.parse(data).ref==='refs/heads/master')```
- When we have a merge event to our master branch it will first go to your directory on server and run 

👉🏻 git pull origin master

👉🏻 npm install 

👉🏻 npm run build 

👉🏻 pm2 start [pm2 is process manager for nodejs. you can replace it with npm start also ]
> Note: git pull will ask for username and password.The prompt should not come thats our motive here. You need to either use ssh or use ```git config credential.helper store```. Check this out [here](https://stackoverflow.com/a/51327559/8461016).I am using credential store as i am the only one accessing that server. 

Now start the script using node or pm2.

# Configuring the Machine.

If you are using nginx , create a proxy pass for it. You just need to open the port 8080 for communication.
- NGINX is a better option as you can assign subdomain to it and also ssl certificate.

# Configuring Repo on Github

1. Go to Setting 👉🏻 Webhooks.
2. ![Webhook page](https://dev-to-uploads.s3.amazonaws.com/i/5jy9s52wkstxp6j0frq2.png)
3. After adding the webhook , edit it and enforce ssl for better security.

That's it . Your are done 🎉.

You can find me on other platforms 👇

%[https://twitter.com/amit_chambial/status/1276500269027622918]
