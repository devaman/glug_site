## GitHub auto README with NODEJS, github-actions and Hashnode API

Hi everyone. As you all know Github has introduced new feature to add a README to your Github profile. We will use this feature to show our latest blogs on our Github Profile. 
We are using:

1.  Github Actions
2. A Nodejs Script to Call Hashnode Api.

## Node js Script

Hashnode api uses Graphql query to fetch data. So the quer to fetch our posts is:

```js
const query = `
    {
      user(username: "amitchambial") {
        publication {
          posts{
            slug
            title
            brief
            coverImage
          }
        }
      }
    }
  `;
```

To call the api we will uses node-fetch package
```js
const result = await fetch('https://api.hashnode.com', {
        method: 'POST',
        headers: {
            'Content-type': 'application/json',
        },
        body: JSON.stringify({ query }),
    })
const ApiResponse = await result.json();
console.log(ApiResponse.data.user.publication.posts);
```
We will get a json response . Now we want to parse is json to markdown.

```js
const posts = ApiResponse.data.user.publication.posts.map(d => {
        return `
### [${d.title}](https://blog.devaman.dev/${d.slug})
<img src="${d.coverImage}" height="100" />
<p>${d.brief}</p>
`
 });
```
Next we will have a defined format for our README.md and we will add this post string there
```js
const markdown = `
![Hello everyone 👋](https://img.devaman.dev/2/?title=Hello%20Everyone%20%F0%9F%91%8B&website=github.com/devaman&back=f1d15b&textFill=fefefe&height=200)
I am Amit Chambial , A Full stack developer. 

I love to contribute to open-source software on GitHub. I am working for JP Morgan Chase as a Software Engineer I. 

I have also worked as a Freelancer and I am a five star freelancer. 

My moto is Make to Learn thats why i have made lots of projects as you can see. I love to make side projects. See my [Producthunt](https://www.producthunt.com/@amitchambial) and [Gumroad](https://gumroad.com/amit_chambial) seller page

# My Blogs

${posts.join("\n----\n")}
`;
```
Last step is to write this makdown to our README.md
```js
fs.writeFileSync('./README.md', markdown)
```
That's it.

[Final Script](https://github.com/devaman/devaman/blob/master/scripts/js/update_readme.js)

## Configuring the Repo

Create a folder ```scripts/js```.
Add this script ```update_readme.js``` there.

## Github Action

1. Go to actions tab 👉 New Workflow 👉 set up a workflow yourself .
2. Now add the below code there

```yml
# Name of your workflow
name: README Update
# Triggers to run workflow
on:
  # workflow_dispatch allows to run your workflow manually
  workflow_dispatch:
  # Run workflow based on specific schedule
  schedule:
  # This workflow will run every day at 00:00 UTC.
  # You can use https://crontab.guru/ if cron syntax is
  # looking weird for you
  - cron: "0 0 * * *"

jobs:
  # This workflow contains a single job called "perform"
  perform:
    # The type of runner that the job will run on
    # ubuntu-latest is default
    runs-on: ubuntu-latest

    # Steps represent a sequence of tasks that will be executed as part of the job
    steps:
    # This line to work properly with repo
    - uses: actions/checkout@v2
    # This one to activate node magic
    - uses: actions/setup-node@v1
      with:
        node-version: '12.x'
    - run: npm install node-fetch
    - name: Run script
      # Here we are setting our secret API Key
      # Details: https://docs.github.com/en/actions/configuring-and-managing-workflows/using-variables-and-secrets-in-a-workflow
#       env:
#         HASHNODE_API_KEY: ${{ secrets.HASHNODE_API_KEY }}
      run: node ./scripts/js/update_readme.js
    # Our script updated README.md, but we need to commit all changes
    - name: Commit and push if changed
      run: |
        git add .
        git diff
        git config --global user.email "github-action-bot@example.com"
        git config --global user.name "GitHub Action Bot"
        git commit -m "Updated README" -a || echo "No changes to commit"
        git push

```

Start commit.

Now return to actions tab 👉 click on your Workflow 👉 run your workflow.

![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1602158371596/ipinL6kC1.png)

After workflow will be finished, go to your account page and enjoy it!

You can follow my repo and grab all ready to use code:
https://github.com/devaman/devaman

## Demo


![demo.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1602161201233/wMU56vbYX.png)



