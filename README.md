# Theater

Theater is a Puppeteer boiler-plate that allows you to quickly build functional web scrapers and automation tools. It has some amazing features like:

* Integrated micro-server (based on Express) which allows you to dispatch new jobs using a simple API.
* Sender module that offers saving job results in many different ways ([READ MORE](#sender-module)).
* Puppeteer Cluster for doing many jobs asynchronously.
* Clean and universal logs using Winston.

## Table of contents

1. [Quick start](#quick-start)
2. [Dispatching jobs](#dispatching-jobs)
3. [Sending job results](#sender-module)
4. [Creating job scripts](#job-scripts)
5. [Example scripts](#example-scripts)
6. [Configuration](#configuration)
7. [Storage Module](#storage-module)
8. [Roadmap](#roadmap)

-----

## Quick start

So, you don't want to read the full document?

![All right then. Here's a shorter version.](shorter.jpg)

After pulling this repository you have to install all required dependencies using **npm** or **yarn**.

```
npm install
```
or
```
yarn install
```

Then copy the **.env.example** file, change its name to **.env** and open it. Now you need to change some variables.

* Set **HEADLESS** to _true_ if you want to see what's currently going on runned jobs (It launches all Chromium instances in full-mode).
* Set **MAX_INSTANCES** to the number of concurrent instances you want to be runned.
* If you want to dispatch jobs using Theater's integrated API make sure **ENABLE_LISTENER** is set to _true_. 
* If you're going to send back job results to your API set the **CALLBACK_URL**.

More information on this topic can be found in the [configuration](#configuration) section.

Now you're ready to run Theater!

To run in **development** mode:
```
npm run start
```

To run in **production** mode:
```
npm run start:prod
```

**IMPORTANT** To run Theater in production mode you need to build it first:
```
npm run build
```

## Dispatching jobs

### Using terminal

Run the command below

```
npm run dispatch
```

This will execute the queue that is saved in **src/queue.json**.
Here's an example **queue.json** file:
```
[
    {
        "name": "takeScreenshot",
        "data": {
            "url": "https://google.com",
            "screenshotName": "screenshots/google.jpg"
        }
    },
    {
        "name": "takeScreenshot",
        "data": {
            "url": "https://bing.com",
            "screenshotName": "screenshots/bing.jpg"
        }
    },
    {
        "name": "takeScreenshot",
        "data": {
            "url": "https://amazon.com",
            "screenshotName": "screenshots/amazon.jpg"
        }
    }
]
```

Remember that dispatching jobs from a terminal launches Theater in **development** mode.

### Using listener

Just send a **POST** request to the _/job_ endpoint of the listener

**Example:**
```
curl --request POST http://localhost:3000/job -H "Content-Type: application/json" -d '{"name": "takeScreenshot", "data": {"url": "https://github.com", "screenshotName": "screenshots/github.jpg"}}'
```

## Sender module

The sender module is used to... send back data generated by your job scripts. Currently you can only send back data to your API, but there are other methods planned to be added (Check the [roadmap](#roadmap)).

How to configure the sender is covered in the [configuration](#configuration).
Usage of the sender is shown in the [job scripts](#job-scripts) section.

## Job scripts

## Example scripts

Theater is shipped with some example jobs which you can run out-of-the-box and test if your instance is working correctly. You can also use them as a cheat sheet to write your own scripts.

### Currently available examples:

* _takeScreenshot_ - Puppeteer goes to the provided URL, takes a screenshot of the page and saves it locally.
* _ipAddress_ - A simple script that goes to myip.com and fetches your current IP.
* _redditMemes_ - A job that goes to Reddit, fetches images from the first page and returns URLs to them. It was created to work on r/memes, but it should also work on other sub reddits.

### Anatomy of a Job script

```
import Job from '../core/job';

class ExampleJob extends Job {
  async run() {
    await this.page.goto(this.url);
    return this.done();
  }
}

export { ExampleJob as default };

```

Every job script needs to be a **class** extended by Theater's **Job** class.
The asynchronous **run** method is called by Theater any time the job is going to be executed.

The **Job** class contains some useful fields and methods:

#### Fields:
* **page** - It contains Puppeteer's **Page** instance.
* **url** - The URL that is passed to the job.
* **input** - Input object containing additional data passed to the job.

#### Methods:
* **done(_data_)** - Call it when your job is done. Using the _data_ paramater you can forward content from the job (like scraped informations) to the sender and then (depending on your configuration) it will be sent to your API or stored.
* **error(_msg_)** - Call it when you want your job to fail. With the _msg_ you can send a custom error message that is going to be logged.

## Configuration

This section will explain to you what all of the options do and how they should be set.

| Field | Description | Default Value |
|-------|-------------|:-------------:|
| **HEADLESS** | If set to _true_ it launches Chromium in headless mode. When set to _true_ it will launch in full-mode. | _true_ |
| **MAX_INSTANCES** | Maximum amount of Chromium browsers that will be runned in parallel. | _1_ |
| **GLOBAL_TIMEOUT** | Global timeout for Puppeteer (in ms). | _60000_ |
| **ENABLE_LISTENER** | Set to _true_ if you want to use Theater's API for job dispatching | _false_ |
| **LISTENER_PORT** | The port you want your listener to listen to. | _3000_ |
| **CALLBACK_URL** | The URL you want your job data to be sent back | _http://localhost:5555_ |

## Storage Module

The storage module allows you to download images and save job data as local files.

### Methods overview

#### Storage.downloadImage(_url_, _path_)

Downloads the provided image

* url {string} - URL to the image you want to download
* path {string} - where the image should be saved

Returns the created file name.

#### Storage.append(_file_, _data_, _newLine_)

Appends data to the selected file

* file {string} - Path to file
* data {string} - Data to be appended
* newLine {boolean} (Default: true) - Append from a new line?

#### Storage.appendJson(_file_, _data_)

Pushes a JSON objects to the array in the file. If it doesn't exists it creates.

* file {string} - Path to file
* data {object} - Data object to be appended

## Roadmap

There are some feature I want to add to Theater.

- [x] Storage module - Allows you to save job data locally and download images.
- [ ] Sender module - saving job data to database.
- [ ] Integrate easy proxy configuration.
- [ ] Create a "stealth" mode for web scraping.

-----

Copyright (c) 2021 Daniel Budziński