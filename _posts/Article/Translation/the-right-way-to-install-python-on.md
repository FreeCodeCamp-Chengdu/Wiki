---
title: Marvelous MLOps Substack
authorURL: ""
originalURL: https://marvelousmlops.substack.com/p/the-right-way-to-install-python-on
translator: ""
reviewer: ""
---

Share this post

![](https://substackcdn.com/image/fetch/w_120,c_limit,f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2Fd2b714b0-c85c-4b90-90d5-c650138efccc_759x517.png)

#### The right way to install Python on a Mac

marvelousmlops.substack.com

Copy link

Facebook

Email

Note

Other

![](https://substackcdn.com/image/fetch/w_128,h_128,c_fill,f_auto,q_auto:good,fl_progressive:steep,g_auto/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2Fe931dbd6-0894-4f67-8bab-ebcc02e3072e_1286x830.png)![](https://substackcdn.com/image/fetch/w_48,h_48,c_fill,f_auto,q_auto:good,fl_progressive:steep,g_auto/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F52063997-b4b8-467e-9412-6aac8f390211_830x830.png)

#### Discover more from Marvelous MLOps Substack

Serving you expertise MLOps content

Over 1,000 subscribers

Subscribe

Continue reading

Sign in

# The right way to install Python on a Mac

### Carefree and flexible Python use awaits you!

[![](https://substackcdn.com/image/fetch/w_80,c_limit,f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2Fa40acb35-bd6b-42f7-ba88-03b69d19860a_512x512.jpeg)](https://substack.com/profile/153357949-raphael-hoogvliets)

[Raphaël Hoogvliets](https://substack.com/@hoogvliets)

Nov 4, 2023

7

Share this post

![](https://substackcdn.com/image/fetch/w_120,c_limit,f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2Fd2b714b0-c85c-4b90-90d5-c650138efccc_759x517.png)

#### The right way to install Python on a Mac

marvelousmlops.substack.com

Copy link

Facebook

Email

Note

Other

[](https://marvelousmlops.substack.com/p/the-right-way-to-install-python-on/comments)

[

Share

](javascript:void(0))

I saw a discussion a while ago on twitter in reply to a question by Chris Albon.

[

![](https://substackcdn.com/image/fetch/w_1456,c_limit,f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2Fdb4d747c-77f9-49d4-b230-9cfe2ba110ba_526x184.png)



](https://substackcdn.com/image/fetch/f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2Fdb4d747c-77f9-49d4-b230-9cfe2ba110ba_526x184.png)

The question got a whopping 521 replies. What a hot topic! I read through a lot of them. I mean, hey, it’s summer and the sun is shining. What else is an ML Engineer to do? The answers were pretty wild I thought, here are a few examples:

-   “Use miniconda”
    
-   “Docker 😛”
    
-   “There’s no right way, it will get f\*\*\*ed beyond repair somewhere down the road regardless”
    
-   “The right way is to don’t and ssh to a remote server with Linux and GPUs”
    
-   And lots and lots of crazy stories of people who just use Python until it stops working. Some of whom even have to reinstall their entire MacBook when it breaks. Yikes!
    
-   One guy even had to get a new life and identity, with that vacuum seller guy from Breaking Bad, after breaking his Python.
    

So what is the right way to install Python on a MacBook? Luckily I saw the right answer pop up quite a few times in the replies as well.

Subscribe

### Pyenv + pyenv-virtualenv

You read that right, this is what you want: pyenv has emerged as an indispensable tool for Python developers, primarily because of its capacity to manage multiple Python versions seamlessly on the same system. This functionality proves invaluable for developers juggling various projects, each demanding its unique Python version. What sets pyenv apart is that it allows developers to pin a specific Python version to a project, ensuring unwavering consistency in the project’s Python environment — even when sharing work with collaborators.

Another advantage of pyenv is its user-centric design. Traditionally, installing global packages might require `sudo` permissions, potentially altering the system configuration. However, pyenv installations and packages are nestled safely in the user’s directory, keeping system-wide settings untouched. This ensures that pyenv doesn’t meddle with the system Python — a crucial feature since many system utilities are intertwined with the default Python, and altering it can usher in a myriad of problems.

[

![](https://substackcdn.com/image/fetch/w_1456,c_limit,f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F09bf2c99-a91a-47fd-bac7-72727414e40d_1280x720.png)



](https://substackcdn.com/image/fetch/f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F09bf2c99-a91a-47fd-bac7-72727414e40d_1280x720.png)

The `pyenv-virtualenv` plugin on top of things is the cherry on the cake. It augments pyenv by weaving in virtual environment (virtualenv) capabilities. This synergy allows for meticulous dependency management on a per-project basis, avoiding any package version conflicts. Such isolation offers another layer of protection against system updates that might modify the system Python or its associated libraries. With pyenv, your projects remain insulated from these shifts, preserving their stability.

Switching between Python versions is an area where pyenv shines. With just a few commands, developers can toggle between versions — be it globally, per session, or per project. Additionally, the wide array of Python versions pyenv offers is impressive! From the standard CPython to the Java-based Jython or the JIT compiler-based PyPy. It gives developers a rich palette of Python implementations at their fingertips. I mean, as a data scientist and ML Engineer I don’t really need these, but still, fingerlickin’ good!

For those in the realm of Python development, pyenv serves as a robust and flexible companion. Whether it’s for maintaining consistent project environments, testing across various Python versions, or simply keeping projects isolated from system changes, \`pyenv\` stands out as the top choice.

So how do we get there?

### 1\. Install XCode Command Line Tools

The first step is to install the command line tools for mac. This includes all kinds of useful stuff for developers, such as a C compiler. You can also use the app store to install it, but you’re a developer now. So we’ll be doing things from the command line in this tutorial.

```
xcode-select --install
```

### 2\. Install HomeBrew

Next, install HomeBrew — the open source package manager for mac. Browse to

https://brew.sh/ and copy and run the command at the top of the page. It should look something like:

```
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

Note that HomeBrew does not automatically update itself nor any of the packages installed. Do this yourself once in a while by executing:

```
brew update
brew upgrade
```

### 3\. Install Pyenv

Pyenv is a package manager specifically for python itself. It allows you to install multiple versions of python on one machine. The pyenv-virtualenv plugin helps create virtualenvs for a specific python version.

```
brew install pyenv pyenv-virtualenv
```

I’m a Z Shell (zsh) user, so this step is a bit specific. Why zsh over regular bash? Well, you can see [a comparison here](https://www.educba.com/zsh-vs-bash/) and decide for yourself. In short: more features. The drawback is that your commands locally can differ from the ones needed on your cloud deployments, where you might have to deal with just regular bash. But I digress

For this step below make sure you properly initalise pyenv and pyenv-virtualenv. In order to do this in zshell, place the following lines in .zshrc in your home directory.

```
eval "$(pyenv init -)"
if which pyenv-virtualenv-init > /dev/null; then eval "$(pyenv virtualenv-init -)"; fi
```

Next, restart all your terminals to ensure pyenv is initialized.

### 4\. Install your desired version of python

Great, you now have pyenv + pyenv-virtualenv! Let’s get some python, no, let’s go crazy, let’s go all kinds of versions of python. Just because we can! But we’ll start with one. To see what’s on offer you can list all available python versions by running.

```
pyenv install -l
```

Note that the base versions (denoted by just a number) are listed at the top. Also note that newer versions only become available when you upgrade pyenv (see updating HomeBrew and its packages above).

Install your favorite version of python with the command below.

```
pyenv install 3.12.0
```

Substitute 3.12.0 for your desired version here of course. You can have multiple versions installed simultaneously by using this command. No worries mate!

### 5\. Create a virtualenv

Change directory to the root of your project. Decide which version of python you want to use. Have a look at which version you have installed and are available to you.

```
pyenv versions
```

You should see the version or versions here that you chose to install. Next you can create a virtualenv with a version of choice. I usually have identical names for my projects, repositories and virtualenvs. I like to keep a clean 1 virtualenv per project / repository. This allows me to easily manage my dependencies.

```
pyenv virtualenv 3.12.0 myproject
```

This will create a virtualenv named “myproject” with python version 3.12.0. Next, activate the environment for your project by running.

```
pyenv local myproject
```

This will create a file named .python-version that contains the name of the virtualenv. Make sure to never commit this file to git! Now, your prompt should probably start with (myproject), unless you have configured your terminal in some funky customized way.

**Deleting environments**

If you want to delete a python environment use the project name as an argument in the following code.

```
pyenv virtualenv-delete myproject
```

### Carefree be your days!

There you have it. No more burning MacBooks, because you needed Python. And I don’t want to hear the name of that other snake ever again. The one that shall not be mentioned. Happy carefree coding everyone!

**Do I really need this?**

But, but, I am just a data scientist who wants to run their notebook? Why all this command line stuff? Ahhh, but there’s the beauty of it all. Even across your notebooks you want to manage your dependencies well. This not only ensures seamless handover for deployment, but also keeps your code working. My advice would be to have a separate project / repository for each data science solution and give it its own environment like mentioned above here (you can still have multiple notebooks per solution of course).

Just install Jupyter Lab in each environment, launch it from the command line using \`jupyter lab\` and you will find that Jupyter Lab will detect your available environments and let you pick one.

[

![](https://substackcdn.com/image/fetch/w_1456,c_limit,f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2Ff4f31977-6d33-49c4-ad39-4da4409b31ac_556x423.png)



](https://substackcdn.com/image/fetch/f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2Ff4f31977-6d33-49c4-ad39-4da4409b31ac_556x423.png)

Beautiful! Want to be smart and efficient and use one environment across multiple projects? Don’t do it.

You might not want to re-download all packages for each separate environment, because you’re behind a slow connection or pay as you use. That’s fair. I would then recommend to keep local copies of your most used packages, but still run separate environments.

[

![](https://substackcdn.com/image/fetch/w_1456,c_limit,f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F0bbd52ba-9801-4e53-a0c1-644220875f34_1200x630.png)



](https://substackcdn.com/image/fetch/f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F0bbd52ba-9801-4e53-a0c1-644220875f34_1200x630.png)

### Bonuscontent: switch to pyenv interpreter in PyCharm

Bonus content and bonus controversy; this guy uses pyenv-virtualenv AND PyCharm? Yeah baby, read and weep. Or meet me in the comments section 🤠

Click your python version in the bottom navigation menu, below your tabs even (standard location)

[

![](https://substackcdn.com/image/fetch/w_1456,c_limit,f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F4c3a638b-0539-42e1-add9-a727ab9df9df_759x294.png)



](https://substackcdn.com/image/fetch/f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F4c3a638b-0539-42e1-add9-a727ab9df9df_759x294.png)

Now choose Interpreter Settings…

[

![](https://substackcdn.com/image/fetch/w_1456,c_limit,f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F02fe5f9f-cdb5-420d-866b-bb2dcd55074a_273x129.png)



](https://substackcdn.com/image/fetch/f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F02fe5f9f-cdb5-420d-866b-bb2dcd55074a_273x129.png)

In the following menu click the settings wheel behind Python Interpreter and choose Add…

[

![](https://substackcdn.com/image/fetch/w_1456,c_limit,f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2Fd2b714b0-c85c-4b90-90d5-c650138efccc_759x517.png)



](https://substackcdn.com/image/fetch/f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2Fd2b714b0-c85c-4b90-90d5-c650138efccc_759x517.png)

Now click the … button behind Existing environment

[

![](https://substackcdn.com/image/fetch/w_1456,c_limit,f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F18d5e4a4-a424-4efe-98ff-5e7b2ab25030_759x498.png)



](https://substackcdn.com/image/fetch/f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F18d5e4a4-a424-4efe-98ff-5e7b2ab25030_759x498.png)

Here it gets interesting, so many pythons to choose from!!! Which one are you picking today?

[

![](https://substackcdn.com/image/fetch/w_1456,c_limit,f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2Fb909c8d2-7c86-4e9a-8915-a9cba5755925_759x1139.png)



](https://substackcdn.com/image/fetch/f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2Fb909c8d2-7c86-4e9a-8915-a9cba5755925_759x1139.png)

In the file browser that is called \`Select Python Interpreter\` navigate to **\`./Users/<username>/.pyenv/versions/<myproject>/bin/python\` …**

[

![](https://substackcdn.com/image/fetch/w_1456,c_limit,f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F617b1fe2-1526-4594-8210-58e5312a2c21_447x481.png)



](https://substackcdn.com/image/fetch/f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F617b1fe2-1526-4594-8210-58e5312a2c21_447x481.png)

…and click **OK, OK, Apply, OK**. You should now be back in your IDE’s main work area!

Note: the name of your project corresponds to the name you have used above in the command

```
pyenv virtualenv 3.10.9 myproject
```

So in the example below it’s **conversion-em-py**

[

![](https://substackcdn.com/image/fetch/w_1456,c_limit,f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2Fa6a4c83e-5d8b-462f-89d1-428d422be5ff_759x206.png)



](https://substackcdn.com/image/fetch/f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2Fa6a4c83e-5d8b-462f-89d1-428d422be5ff_759x206.png)

Check if you switched to the right environment by looking at the button below in your IDE!

That’s it, you are all set!

Thanks for reading Marvelous MLOps Substack! Subscribe for free to receive new posts and support our work.

Subscribe

7

Share this post

![](https://substackcdn.com/image/fetch/w_120,c_limit,f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2Fd2b714b0-c85c-4b90-90d5-c650138efccc_759x517.png)

#### The right way to install Python on a Mac

marvelousmlops.substack.com

Copy link

Facebook

Email

Note

Other

[](https://marvelousmlops.substack.com/p/the-right-way-to-install-python-on/comments)

[

Share

](javascript:void(0))

Previous