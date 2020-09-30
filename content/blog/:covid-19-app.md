---
title: "UK COVID-19 stats application"
date: 2020-09-30T13:56:45+01:00
description: "A simple Flask app showing how the COVID-19 (SARS-CoV-2) pandemic is developing in the UK."
draft: true
tags: ["Flask", "Python", "COVID-19", "UK", "Azure", "App Service"]
math: false
toc: false
---

Hi! :wave:

I had some free time recently so I decided to develop something which was relevant to current worldwide situation. The ongoing Coronavirus (COVID-19) pandemic.

This is going to be a short blog post about the application, its aims and deploying it to Azure App Service.

# Aims

I had three aims that I wanted to achieve:

* I wanted to develop an application which was simple.

* I wanted to show data about how the COVID-19 pandemic is developing in the UK.

* I wanted to show relevant news related to COVID-19 in the UK.

# UK-COVID-19-Stats Python application

The application is open source on Github and developed in Python. Check it out [here](https://github.com/dbrennand/UK-COVID-19-Stats)! :smile:

The application uses a number of libraries including:

* [Flask](https://flask.palletsprojects.com/en/1.1.x/): I have had some previous experience with this web application framework for Python. It is well documented and this project is relatively small, so it made sense to pick Flask for this application.

* [UK-COVID-19](https://github.com/publichealthengland/coronavirus-dashboard-api-python-sdk): This library is maintained by the folks over at Public Health England (PHE) and allows my application to retrieve UK based data about the COVID-19 outbreak. More on this [below](#application-datasource).

* [Loguru](https://github.com/Delgan/loguru): This is personal preference but I like how this library makes Python logging hassle free and is very easy to integrate into my application.

* [Feedparser](https://pythonhosted.org/feedparser/): This library allows my application to parse BBC's health news RSS feed and show COVID-19 related articles in my application. This can be seen on this like in [app.py](https://github.com/dbrennand/UK-COVID-19-Stats/blob/master/app.py#L118).

## Application datasource

I began looking for a potential datasource for my application to retrieve UK based data for the COVID-19 outbreak.

I came across the [UK COVID-19 dashboard](https://coronavirus.data.gov.uk/) which provides an API and Python SDK (mentioned above) developed by PHE. The API is free (Thanks PHE! :heart:) and provides loads of data including (but not limited to):

* New cases.

* Hospital cases.

* Admissions.

* Substantial testing data (including data for each [pillar](https://www.gov.uk/government/publications/coronavirus-covid-19-testing-data-methodology/covid-19-testing-data-methodology-note)).

* Deaths.

Obviously, it was a perfect choice for the project :smile:

For more information on the API, you can look at the documentation yourself [here](https://coronavirus.data.gov.uk/developers-guide).

## Running locally

You can run the application locally and test it out for yourself if you have docker installed.

First, build the image from the dockerfile in the repository and then run the image. You can access the app at http://localhost:5000.

More detailed instructions on how to do this can be found [here](https://github.com/dbrennand/UK-COVID-19-Stats#docker).

## Deploying to Azure App Service

I wanted to host the application somewhere for free. Luckily for me, Microsoft's Azure App Service has a free tier instance (SKU: F1) which allowed me to host my application free of charge! :heart: :smile:

I deployed the application onto Azure App Service by using the [Azure CLI](https://docs.microsoft.com/en-us/cli/azure/what-is-azure-cli). It was super easy and involved the following steps:

1. Install the Azure CLI.

    - I'm using Windows so I used [chocolatey](https://chocolatey.org/) to install it using the following command: `choco install azure-cli -y`

2. Login to my Azure account using the command: `az login`.

3. From my project repository, run the following command: `az webapp up --sku F1 --name <app-name>`.

Thats it! The Azure CLI did all of the heavy lifting creating a resource group for my application in Azure, creating an App Service plan and the app service object, zipping up my application and deploying it! :smile:

You can access a live version of the application by heading to https://uk-covid-19-stats.azurewebsites.net.

For more information on the steps above, see the following Microsoft [documentation](https://docs.microsoft.com/en-us/azure/app-service/quickstart-python?tabs=bash&pivots=python-framework-flask).

Overall, this was a fun little project which I believe met the aims I set out to achieve. I also ended up learning how to deploy a Python application to Azure App Service :smiley:

Thanks for reading and I hope you enjoyed reading this short blog post. Until next time! :wave:
