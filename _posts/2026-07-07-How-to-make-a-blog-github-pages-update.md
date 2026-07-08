---
layout: post
title: "How to Create A Blog in GitHub Pages (Updated)"
---
# What I Wanted
I wanted to create a blog to publish some of the things I've be working on. I knew that GitHub has this feature `GitHub Pages` which allows anyone to create and serve static web pages. The best part is that you can use html or markdown with GitHub Pages! I already like working with markdown and html so I decided this would be a great option. 


# Understanding GitHub Pages
GitHub Pages is a feature that can be enabled for free on public repositories. It can be used to serve static web pages ( web pages that do not take user input or require special server-side processing ). The default tool used to render these pages is Jekyll. 

Jekyll uses a `_config.yaml` to manage settings including the theme of the site, plugins, and other settings. You can reference the official GitHub documentation [here](https://docs.github.com/en/pages/setting-up-a-github-pages-site-with-jekyll/about-github-pages-and-jekyll) about using Jekyll. 

To use the GitHub pages feature 
1. First create public repository that ends in `.github.io`.  The example used in the official guide is  `username.github.io`. The official quick-start guide  can be found [here]( https://docs.github.com/en/pages/quickstart ).  
2. To enable GitHub Pages, go to  the repository's `Settings`. Click the `Pages` tab under `Code and automation`. Then select one of your branches to build and deploy ( this will likely be `main`). 
Follow the official quick-start guide if you get lost. Once this is configured properly you will have your own static website on GitHub. GitHub will process any changes on the branch you selected and render them. 

> Note: It may take up to 10 minutes for your site to be rendered after changes to the branch

You will be able to visit your site at `https://repo-name.gethub.io` were `repo-name.gethub.io` is replaced by your repository's name. If you receive a 404 error, the site has not finished rendering.
# Testing Jekyll Locally
GitHub recommends testing your Jekyll pages locally. One of the advantages of testing locally is that you can view your page before committing it to the public site. This is great for catching errors and typos. Additionally, you can save time testing locally because you will not have to wait for GitHub to render your pages. 

Jekyll is not officially supported in Windows. There is documentation that exists to get it working but I have heard mixed results. Jekyll has its own installation guide for Windows [here](https://jekyllrb.com/docs/installation/windows/), but I believe there is an easier way to use this tool. If you haven't heard of Docker, it will change your life.

Docker is containerization technology. This allows you to place all files for a tool or application in a box or container to be run. This container is mostly isolated from your main operating system (OS), but it uses resources from it like RAM and processing power. This is a very high level view of what Docker can do, but it should be sufficient for now.

Docker is available on both Linux and Windows. For Linux I'd recommend installing the official Docker tools, but you should be fine with your distribution's managed version of Docker. For Windows you will have to install `wsl`. You can find an installation guide [here](https://learn.microsoft.com/en-us/windows/wsl/install). I recommend installing Docker Desktop for Windows. 

> Note: Docker also exists for Mac but I don't own one 

## Setting up Jekyll site with a Container
An official image of Jekyll is available on Docker Hub. We'll use this image to create our Docker container. If you are unfamiliar, Docker images are templates to create a container. You will need to interact directly with a Jekyll container to initially create your site.

> Note: The Jekyll Docker Image is in AMD64 architecture by default. If you have an ARM64 processor there is an alternate Docker Image that can be used. If you have trouble consider following the official [guide](https://docs.github.com/en/pages/setting-up-a-github-pages-site-with-jekyll/creating-a-github-pages-site-with-jekyll#prerequisites) to setup Jekyll locally.

> Ensure Docker Desktop or daemon is active before continuing

1. Navigate to your GitHub repo's root directory. Clone the repo to your machine if you have not already. If you don't know how to clone a repo, learn [here](https://docs.github.com/en/repositories/creating-and-managing-repositories/cloning-a-repository). 
2. In a Linux terminal run the following code snippet. This code with spin up a Jekyll container based off the image, bind your current directory to the container allowing changes in the container to update your repo, and open a shell into the container. You may need to use `sudo`. On Windows you can get away with a PowerShell terminal.

>> The maintainer of the Jekyll docker image has made some recent changes. The main item to note is that there is a dedicated tag for GitHub Pages `pages`. The `latest` tag does not contain the gems needed for a GitHub Pages project to function.

>> Additionally, there are now ARM64 versions of the Jekyll docker image for tags `latest` and `pages`. If you only use the tag and not the digest for a specific architecture, the AMD64 version will be used. The AMD64 architecture will work on an ARM processor but your local site will load extremely slow. To specifically use the ARM64 version you must append the sha256 digest to the tag. For example `jekyll/jekyll:pages` becomes `jekyll/jekyll:pages@sha256:<current-value>`, where `<current-value>` is the current digest for the arm64 version of the tag. You can lookup the current digest on Docker Hub or after you pull the docker image `jekyll/jekyll:pages` with the command `sudo docker buildx imagetools inspect jekyll/jekyll:pages`.

```sh
docker run -it  --name setupjekyll  --mount type=bind,source="$(pwd)",target=/srv/jekyll  jekyll/jekyll:pages sh 
```


> The Next few steps  ( 3 - 10) will follow steps 7 - 14 from the GitHub `Create site with Jekyll` [page](https://docs.github.com/en/pages/setting-up-a-github-pages-site-with-jekyll/creating-a-github-pages-site-with-jekyll). 
> Steps 3 - 9 are needed to set up your Jekyll site and will only need to be done once 

3. Now start a new Jekyll project. Run `jekyll new --skip-bundle . --force` This will create some new files. You must use the force flag because the current directory will not be empty. Default Jekyll files will be overwritten but other files will not be.
> Note: If you used PowerShell you will need to run the command in step 3 in a different directory of the container and copy all the files back to `/srv/jekyll`
4. Open `Gemfile` in an editor. You can install nano with `apt install nano` in the container.
5. Comment out the line starting with `gem jekyll`. Use `#`
6. Uncomment the line starting with `# gem "github-pages`  and makes sure to specify the correct version number like `gem "github-pages", "~> 232", group: :jekyll_plugins` where 232 is the current version. The current version for GitHub Pages can be found [here](https://pages.github.com/versions.json). If you're wondering why the link goes to a json object rather than a regular web page, I have no answers for you. Previously, there was a github page with all current versions displayed, but now the page auto redirects away.  
> Note: The Official Jekyll container has recieved many updates recently. It still works for the current version of GitHub pages. If you find that the Jekyll version is too old run `apt update` and `apt upgrade` inside the container.
7. Save and close the file 
8. Run `bundle install` This will install the dependencies.
9. Open the `.gitignore` file and add `Gemfile.lock`
10.  Make changes to `_config.yaml`
11. Now you can exit your container and all your file changes will persist
12. Optional: Run `docker rm setupjekyll` to remove the container. May need to use `sudo`
## Actually Running the Site Locally
There are two ways you can run the site locally with a container. The first is to use a slight modification of the docker run command previously given in step 2 of <b> Setting up Jekyll Site with a Container</b>. Simply publish the ports that Jekyll serves on. It is important to note that publishing ports from a container bypasses UFW rules. The Second option is to use Docker Compose. This tool uses a `.yaml` file to bring up a container quickly without the need to memorize run options. Both of these methods bind the current working directory to the container. This means your edits on your local machine will be reflected instantly in the container. Jekyll will hot reload allowing you to view and test your site in real-time.  

> Note: Jekyll will hot reload markdown files and other files that are served if the livereload option is specified. You may need to restart the server for configuration changes to take affect.

> Note: Both methods assume you are currently in the directory of your project.

### Method 1 Using a Run Command
1. Run the following. This command spins up a Jekyll container, names it `setupjekyll`, binds the current directory to the container allowing for updates, and opens a shell. May need to use `sudo`. May need to delete container of the same name if it already exists. 
```sh
docker run -it  --name setupjekyll -p 4000:4000 -p 35729:35729 --mount type=bind,source="$(pwd)",target=/srv/jekyll  jekyll/jekyll:pages sh 
```
2. Run `jekyll serve --host 0.0.0.0 --livereload`. This will start the Jekyll server. You should now be able to reach your site from your browser. Depending on your firewall it will be visible on the local network. The site will be served at `http://localhost:4000`

>> Jekyll can no longer hot reload with the basic command `jekyll serve`. Now there is a separate argument `--livereload` this will serve the live reload version of the site. There is a JavaScript method on the blog site at port 4000 that makes a web socket connection to port 35729 of the container for updates. You must expose both port 4000 and 35729 for live reloads of the site. 

>> An additional host parameter `--host 0.0.0.0` must be set or the container will reject any incoming requests, including from the machine hosting the docker container. 

#### Subsequent Deployments
After you create the docker container using method 1, it becomes easy to spin up the container again and begin working. You simply start the container, open a shell into it and start the Jekyll server.
1. Run `docker start setupjekyll`. You may need `sudo`. If you get an error make sure that the container is present on your device with `docker ps -a` and that Docker Desktop or the Docker daemon is running. 
2. Then run `docker exec -it setupjekyll sh`. This will pop a shell into the container. 
3. Run `jekyll serve --host 0.0.0.0 --livereload`
4. After finishing work with the container shut it down with `docker stop setupjekyll` or `docker kill setupjekyll`. 

> Note: Docker will bypass your UFW rules when publishing to a port

### Method 2 Using Docker Compose
1. Make sure you have Docker Compose. Docker Compose is actually a separate tool than the standard docker commands. Test by running `docker compose` or `docker-compose` The version without the hyphen is the newer version. 
2. Create a file `docker-compose.yaml`. This file will contain the instructions to run our container.
3. Copy the following code block into `docker-compose.yaml`. The configuration tells Docker Compose to use the Jekyll image to create a container (ct), bind the current directory to `/srv/jekyll` in the ct, map port 4000 in the host to port 4000 on the ct, and port 35729 in the host to port 35729 in the container, and start the Jekyll server.

```yaml
 services:
  jekyll:
    image: jekyll/jekyll:pages
    volumes:
      - type: bind
        source: ./
        target: /srv/jekyll
    ports:
      - 4000:4000
      - 35729:35729
    command: jekyll serve --host 0.0.0.0 --livereload

```
> If your processor is ARM64 architecture, use the ARM64 version of the tag `jekyll/jekyll:pages@sha256:<current-value>`, where `<current-value>` is the current digest for the ARM64 version of the tag.

4. To start the container, run `docker-compose up` or `docker compose up` depending on your version. You will see the container building. You can also use `-d` at the end of the command to detach the process from your terminal. Your site will be at `http://localhost:4000/`
5. To stop the container in detached mode run `docker-compose stop` in the same directory as the Docker Compose file. Alternatively you can run `docker-compose down`. The down option will destroy the container and the associated docker network. 

#### Subsequent Deployments
1. Navigate to the directory with the Docker Compose file. 
2. Use `docker-compose start` if the container still exits, or use `docker-compose up`
3. To shutdown, use `docker-compose stop` in the same directory as the Docker Compose file.

# Other Notes
- You can learn more about working with Jekyll at their [site](https://jekyllrb.com/). 
