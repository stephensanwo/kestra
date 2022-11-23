# Getting Started

## What is Kestra?

Kestra is an open-source orchestration & scheduler platform that makes it easy to build, schedule,run, and monitor complex pipelines. Kestra focuses on simplicity and scalability, so that anyone (Data Engineers, SOftware Developers, Business Analysts, etc) can quickly and easily automate their workflows, create a shcedule to run the workflows, and extend the workflows to cover more complex use cases as the business requirement changes. [This Article](https://kestra.io/blogs/2022-02-01-kestra-opensource.html) discusses the motivation behind Kestra.

## Pre Requisites

Kestra is shipped with Docker, to start with Kestra, you need to have docker installed and running on your system. If you don't have Docker installed already, follow the instructions [here](https://docs.docker.com/desktop/install/mac-install/) to install docker for the operating system of your choice.

## Installing Kestra

On, your local machine, create a directory for kestra and download the docker-compose.yml file from [docker-compose.yml](https://github.com/kestra-io/kestra/blob/develop/docker-compose.yml) byt running `wget https://raw.githubusercontent.com/kestra-io/kestra/develop/docker-compose.yml` on mac and linux or create a `docker-compose.yml` file, then copy the contents into the file.

Now, in the terminal, run `docker-compose pull` in the root of your kestra project, where you downloaded the docker-compose.yml file. This will pull all the remote images, then run `docker-compose up -d` to build the images

In docker desktop, you should see botk kestra and postgres images running within the kestra container like below ![Image](../assets/images/Screenshot%202022-11-23%20at%2021.00.06.png)
