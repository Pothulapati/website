+++
title= "Setting up a blog with Wyam and Github Pages"
date= "2017-07-26"
tags= ["github-pages", "wyam", "static-site"]
+++

 I've always wanted a blog and everytime I chose the *hard-way* as I wanted to manage the whole stack myself on a Windows server (*because .Net Framework*) and then with the advent of .Net Core I moved onto a linux server which decreased my costs slightly but I wasn't happy as my Azure credit couldn't afford either of the options.
  
Now here I'am with a blog which costs only my time (which I have in plenty).

### Wyam

>[Wyam](https://wyam.io/) is a simple to use, highly modular, and extremely configurable static content generator that can be used to generate web sites, produce documentation, create ebooks, and much more.

It convert's your dynamic sites into static sites which can then be **freely** hosted on services like [Github Pages](https://pages.github.com/),[Netlify](https://www.netlify.com/). You can also convert Asp.net MVC sites,Razor Views,etc to static sites using Wyam.

You can check the wyam documentation [here](www.wyam.io/docs)
### Github Pages
>[GitHub Pages](https://pages.github.com/) is a static site hosting service.GitHub Pages is designed to host your personal, organization, or project pages directly from a GitHub repository. 

It let's you host static content ranging anything from websites to documentation. Custom Domain support is also available. 
## Wyam and Github Pages to rescue

Now let's start setting up a blog with Wyam using the in built Blog recipe and CleanBlog theme.
Download the latest  version of Wyam by going [here](https://github.com/Wyamio/Wyam/releases). Once you complete installing Wyam, Make sure you add the environment variables for the commands to work.

Now Run the following command in a directory to scaffold the Blog template.
```
wyam new --recipe Blog
```
Now you can see the following files in the folder.

![](/images/directory.PNG)


The input directory contains the content files.These files can be markdown files,razor files,etc. Links in the header on the homepage are added by directly adding files into the directory and for adding posts, markdown files are added directly into the posts folder.

These files act as input to wyam which converts these files into static HTML files along with the theme you specified. Now Let's convert our markdown,razor files into static files.

```
wyam --recipe Blog --theme CleanBlog
```

**BOOM!** Now, You can see a output folder with all the Static content files like HTML,CSS,etc.

You can now preview the site using the command ```wyam preivew```.

Now that, We have a site running locally. Push the site to a github repository(public only) and setup the github pages by going into the repo settings. There are also further instructions available to setup a custom domain in the settings page.
Everytime we make some changes to the input files,we have to run wyam on those files and push the output files onto github. 

It's 2017, We have put people on the moon, Can't we automate this process?

So, Let's use appveyor to read the input from a github branch, let it run wyam on it and then push the output to a branch which has github pages setup on it.

Looks Cool, Dosen't it? Here is the Appveyor script to set it up.
<script src="https://gist.github.com/Pothulapati/2f4c6b0c8b7c0063df2586180ef2c362.js"></script>

Replace the ```$name$``` , ```$email$``` with corresponding details. You also need to setup an access token in the developer settings to let appveyor access your github repository.

The Above script does the following
* Gets the input files from the source branch specified.
* Gets and Installs the latest version of wyam.
* Runs wyam on the specified branch.
* Saves the output of wyam to the specified branch ( on which github-pages is set up).

Now all you have to do is push the input files onto a github repository and appveyor does the building and publishes your blog.

Instructions to setup a custom domain are [here](https://help.github.com/articles/adding-or-removing-a-custom-domain-for-your-github-pages-site/)

So,We have just setup a blog through wyam and github pages which dosen't cost us a penny. [Dave Glick](https://twitter.com/daveaglick) has really did some great work through [Wyam](https://wyam.io/). Things we did is just the tip of the iceberg that wyam can do.



























