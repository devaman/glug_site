## SSR vs CSR vs Static Render

Hey everyone,

I recently saw a post asking for some help on this topic. So i am writing on this topic.Lets start.

## First off all there are two type of data in website :

1. Static data ( for everyone it is same eg site name )
2. Dynamic data ( for everyone it is different eg username ). Dynamic data is sent by server means we need a server here which will call database, fetch data from it and sends it to browser.There are two methods to send this data to browser or client !


### Ssr: server side rendering 

The server creates a HTML page with data in it and sends in as a response to browser.

![ssr.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1591694282240/nasL5mUjB.png)

eg Hashnode, Github, etc (Most of the big companies uses this)

> Advantages : SEO Friendly, Easy.

### Csr: Client side rendering

Here client is our browser.
When we first open a site , the whole site HTML code is fetched to the browser but there is no data in it( means missing dynamic data like user details etc). After the code is fetched , we load the data by calling the server api which in return send the data as json object.

![csr.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1591694287135/tKTfhW_gt.png)

eg GetMakerLog

> Advantages : Behaves like an app.(Single page application, Fast (First time load is slow as it downloads all the html css,js for website)

### Hybrid Rendering:

Server side rendering + Client Side Rendering.

On first load the server sends the HTML and data for that site , and the HTML code of  all website pages.
So first time the browser open the site , its SSR and when we navigate to other pages its CSR. 

## Static Websites

There is no dynamic data. It uses only static data. We cannot add data according to user logged in.
No Server is required.
eg Gatsby, Jekyll.