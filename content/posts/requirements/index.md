---
title: "Creating and using a requirements.txt file to install Python Packages with PIP"
date: 2023-05-14T20:24:21-03:00
draft: false
tags: ["Python", "pip", "requirements.txt"]
categories: ["Python"]
ShowToc: true
---

## Introduction

If you're a Python developer looking to efficiently manage external packages in your projects, you've come to the right place! In this step-by-step guide, I'll teach you how to create and use a requirements.txt file, leveraging the powerful package manager PIP. This is a recommended practice that allows for easy sharing of development environments and ensures the proper installation of all required packages. So, let's get started!

## Step 1: Creating the requirements.txt file

To begin, open your terminal or command prompt and navigate to your project directory. Then, enter the following command to create the requirements.txt file:

```console
touch requirements.txt
```

The command "touch requirements.txt" creates an empty file named "requirements.txt" in the current directory. It updates the timestamp of the file or creates it if it doesn't exist, without adding any content to the file. This command is commonly used to create a file quickly before adding the desired content later.

## Step 2: Listing Packages in the requirements.txt file

Open the requirements.txt file in a text editor and add the necessary packages, one per line. Each line should follow the format package==version, where "package" is the name of the package you want to install, and "version" is the specific version of the package (optional but recommended to avoid compatibility issues). For example:

```plain
package1==1.0.0
package2==2.3.1
```
Add all the required packages in this manner, ensuring each package is on a separate line.

## Steps 1 & 2: Recommended alternative

Alternatively, if you already have the packages installed in your environment, you can use the following command to generate the requirements.txt file automatically:

```console
pip freeze > requirements.txt
```

The command "pip freeze > requirements.txt" generates a requirements.txt file by capturing the list of installed Python packages and their respective versions in the current environment. It redirects the output of the "pip freeze" command, which displays the package names and versions, and saves it to the requirements.txt file.

## Step 3: Installing Packages from the requirements.txt file

If you've just obtained the files of a project and need to install the packages from the requirements.txt file, you can use PIP to install all the packages at once. In your terminal or command prompt, execute the following command:

```console
pip install -r requirements.txt
```
PIP will read the requirements.txt file and automatically install all the listed packages.

## Step 4: Verifying Package installation

After the installation completes, it's good practice to verify if the packages were installed correctly. You can do this by executing the following command:

```console
pip freeze
```

This will display the list of installed packages, along with their versions, in the terminal.

## Conclusion

Congratulations! You now know how to create and use a requirements.txt file to install packages with PIP. This practice is extremely useful when sharing your development environment with other developers or setting up a new workspace.

Remember to keep the requirements.txt file updated whenever you add, remove, or update packages in your project. This ensures consistency across environments and avoids compatibility issues.

## Video tutorial

If you prefer a video, I have this content in a Youtube video (speaking in Brazilian Portuguese):

{{< youtube dSM32RLHwM8 >}}
