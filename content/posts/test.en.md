---
title: How to really using Hugo and GitHub action together
weight: 10
date: 2021-10-04T14:22:24+02:00
draft: false
---

This is my first time to create a blog. Yes, I know I know, I'm an software engineer, I had learn to create and deploy a website in my early days at my university. But back then (2007), there isn't much framework and way to do that like now (2021). And I have forgot alot of things, in order to learn new things. So it's like start at the beginning. 
I ask around and decided to use Hugo framework to build my static website and github action to deploy my website on the internet.
I have had some trouble when I follow the guides on the internet, even the official one, they are great, with alot of detail, but sometime I feel like it is more for someone who already know how to create and deploy a website. So I decided that my first blog will be talking about how to use Hugo framework to build my website and github action to deploy my website on my ```<your github account name>.github.io``` repository.
What is Hugo framework ?
: Hugo is a static site generator written in Go.

What is GitHub actions ?
: GitHub actions helps you build, test and deploy applications. In our case, I will use it to build and deploy my website automatically.

How did I do it ?
: In the next part, I will show you how did I do it, step by step with comment on things that I missed, and loss time to correct it. I also put links that I followed for guide with more informations.

- Step 1: [Choose a theme](https://themes.gohugo.io/)
Answer some questions in order to choose a appropriate theme 
: The look that you want for your website
: The features that it support (like multilinguages ? and so on.)
- Step 2: [Create and deploy your site on local](https://gohugo.io/getting-started/quick-start/)
    - [Installing Hugo](https://gohugo.io/getting-started/installing/): In my case, I'm on a Windows machine and use Chocolatey for package management, so first I open a CLI (Command Line Interface), then I install Hugo with the following command:    
        ```choco install hugo -confirm```

    To verify the version that you installed, use ```hugo version```.

    - Create a new site:
        First of all, you have to go to the folder where you would like you site folder to stay, then using command 
        ```cd <path/to/your/folder/>```

        After that, you can create your new site ```hugo new site <name_of_your_site>```. 
        Now you will have a folder named <name_of_your_site> at the path <path/to/your/folder/>. Ex: I will have a folder named 'MyBlog' in my path 'D:\Documents\' like this 'D:\Documents\MyBlog\'.

    - Add a theme: I choose the theme [Learn](https://themes.gohugo.io/themes/hugo-theme-learn/). But feel free to chose what ever you like.
        - Don't forget to go to the site folder by using ```cd MyBlog``` in the CLI.
        - Create a git repository inside of your folder 'MyBlog' ```git init```.
        - If you want the latest version with bug fix and
            - If you don't want to customize the theme later, use the following command ```git submodule add https://github.com/matcornic/hugo-theme-learn.git themes/learn```. This command will save your theme in a folder with path theme/learn and add it as a submodule in git.
            - If you want to customize the theme later, I sugest that you go on your github site, find your theme (ex: https://github.com/matcornic/hugo-theme-learn), folk it, then using that link to do the command ```git submodule add <link/to/theme/repository>.git themes/<themeName>```. Ex: 
                ```git submodule add https://github.com/ngocbichdao/hugo-theme-learn.git themes/learn```
        - If you are a non-git users: follow the guide in [here](https://gohugo.io/getting-started/quick-start/).

    - If you want to verify, you can start your local site by using this command ```hugo server```. Navigate to your new site at http://localhost:1313/.

- Step 3: Push Hugo site code on Github
    - Add a new remote for your branch ```git remote add <name_of_your_remote> <name_of_your_new_branch>```Ex: Add new remote branch named 'origin' for your repository ```git remote add origin git@github.com:<user_name>/<repo_name>```.
    - Add all files and folders ```git add --all```.
    - Commit to the branch 'origin' ```git commit -m "<your_commit_message>"```.
    - Create new branch on your computer ```git checkout <name_of_your_new_branch>```. Ex: ```git checkout -b source```.
    - Try and redefine the ssh url for remote origin: ```git remote set-url origin git@github.com:<user_name>/<repo_name>.git```
    - Push from 'origin' branch (on your computer) to 'source' branch (on github) ```git push origin source```. If your branch on github have different name, don't forget to change it in the command. Note that the Hugo site code have to be push on a branch that different than the default (main) one. Because your public static site will be posted on default branch (main branch in this case). Belive me, I have made this mistake, and it take time to repair it. At that time I didn't know that we can change the default branch on github so easily... (Go in your repository on github/Settings/Branches and change the default branch.)
- Step 4: Connecting to github repo with a SSH key
    - [Checking for existing ssh keys](https://docs.github.com/en/authentication/connecting-to-github-with-ssh/checking-for-existing-ssh-keys)
    - If you don't have any ssh key, now is the time to [generate a new ssh key and add it to the ssh agent](https://docs.github.com/en/authentication/connecting-to-github-with-ssh/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent). 
Once your connection is etablish, you can go to next step.
- Step 5: Create GitHub token. This step is well described in [here](https://ruddra.com/hugo-deploy-static-page-using-github-actions/#step-4-create-github-token).
- Step 6: Add token as Secret on GitHub. This step is well described in [here](https://ruddra.com/hugo-deploy-static-page-using-github-actions/#step-5-add-token-as-secret-in-github).
- Step 7: Create A GitHub Action. In your Hugo site folder, there is a folder named './github/', now is the time to create a folder named 'workflows' and a file named 'githubAction.yml'. My *.yml is looked like this 
        
        name: CI
        # Controls when the workflow will run
        on:
        # Triggers the workflow on push or pull request events but only for the main branch
        push:
            branches: [ source ]
        
        # Allows you to run this workflow manually from the Actions tab
        workflow_dispatch:

        # A workflow run is made up of one or more jobs that can run sequentially or in parallel
        jobs:
        # This workflow contains a single job called "build"
        build:
            # The type of runner that the job will run on
            runs-on: ubuntu-latest

            # Steps represent a sequence of tasks that will be executed as part of the job
            steps:
            # Checks-out your repository under $GITHUB_WORKSPACE, so your job can access it
            - name: Git checkout
                uses: actions/checkout@v2
                with:
                submodules: true  # Fetch Hugo themes (true OR recursive)
                fetch-depth: 0    # Fetch all history for .GitInfo and .Lastmod
                
            #- name: Update theme
                # (Optional)If you have the theme added as submodule, you can pull it and use the most updated version
            # run: git submodule update --init --recursive

            - name: Setup hugo
                uses: peaceiris/actions-hugo@v2
                with:
                hugo-version: "0.88.1"
                # extended: true

            - name: Build        
                run: hugo

            - name: Deploy
                uses: peaceiris/actions-gh-pages@v3
                if: github.ref == 'refs/heads/source'
                with:
                personal_token: ${{ secrets.TOKENHUGO }}
                publish_dir: ./public
                # keep_files: true
                publish_branch: main
                #   cname: example.com
    
    More explication for this code can be found in [this link](https://ruddra.com/hugo-deploy-static-page-using-github-actions/#step-6-create-a-github-action).
- Step 8: Push on github using these command and your action will start automatically. You can check its progress in the actions tab.
    ```
    git add --all
    git commit -m "your_commit_message"
    git push origin source
    ```

- Step 9: I prefer to change my site on local and then push new commit on source branch. But feel free to make change using the web interface of GitHub.

Thank you for reading my first blog. If you have any questions or remarks, feel free to contact me.
