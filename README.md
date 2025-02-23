# Amazon Kindle eBook Bulk Downloader

Designed for downloading your Kindle eBooks in a more automated fashion than is typically permitted, this tool allows you to create backup copies of the books you've already purchased.

## Pre-Requisites

The most important pre-requisite is that you have a physical e-ink Kindle or an Amazon Fire Tablet linked to your Amazon account. This is a requirement from Amazon's side and this tool does not offer a way to bypass this.

An important distinction is that the physical Kindle you have linked cannot be one of the latest 2024 models. For whatever reason, Amazon has decided to block the ability to download books from these devices. This tool will not work with these devices.

The easiest way to check if this tool will work for you is to log into your Amazon account manually and navigate to `Manage Your Content and Devices`. From there when clicking on `More Actions > Download & Transfer via USB` on a book, you should see your Kindle or Fire Tablet device listed. If you see the message `You do not have any compatible devices registered for this content` then it means you:

1. either don't have a Kindle or Fire Tablet device linked to your account, OR
2. have a Kindle or Fire Tablet device that is not compatible with this tool

If you are able to proceed to the next screen and download the book, then this tool should work for you.

## Setup

You need to have Bun installed before you can run this too. Please see here for the most up to date instructions for your system: https://bun.sh/docs/installation

Once Bun is installed, clone this repository, and run the following command from the root of the downloaded repository to install dependencies.

```bash
bun install
```

Then run the following command to install a special version of chrome for use by this tool

```bash
bunx puppeteer browsers install chrome
```

### Docker

1. Clone this repo
2. Copy the .env template then edit it
3. Build the image
4. Run the container, passing the OTP on the commandline

```bash
cp .env.template .env
vi .env
docker compose build
OTP=???? docker compose run
```

## Running

Note that amazon credentials will need to be provided. Currently this script expects them to be in the following ENV variables:

| Variable      | Description               | Required                    |
| ------------- | ------------------------- | --------------------------- |
| `AMAZON_USER` | Account username          | Yes                         |
| `PASSWORD`    | Account password          | Yes                         |
| `OTP`         | 6 digit one-time password | If 2 factor auth is enabled |

Note this tool is already configured to read environment variables from an `.env` file in the root of the repo. You can make a copy of the `.env.template` file called `.env` and fill in your details there for an easy way to get started.

### CLI Arguments

The following CLI arguments are made available to customise the downloader to your needs

| Argument              | Default Value                          | Description                                                                                                                      |
| --------------------- | -------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------- |
| `--baseUrl`           | N/A (Will be prompted if not provided) | Which Amazon base URL to use. Note, this MUST include www. in order to work properly                                             |
| `--totalDownloads`    | Infinity                               | Total number of downloads to do                                                                                                  |
| `--maxConcurrency`    | 10                                     | Maximum number of concurrent downloads                                                                                           |
| `--startFromOffset`   | 0                                      | Index offset to begin downloading from. Allows resuming of previous partially finished attempts.                                 |
| `--manualAuth`        | false                                  | Allows user to manually login using the pupeteer UI instead of automatically using ENV vars. Use when auto login is not working. |
| `--duplicateHandling` | skip                                   | How to handle files of the same name/size on the filesystem. Options: skip, overwrite                                            |

### Running

You can run this tool using the following command

```bash
bun run start
```

Command line arguments can be provided as follows

```bash
bun run start --baseUrl "https://www.amazon.com.au"
```

## Docker

### Build docker image

```bash
docker build . \
   -t amazon-kindle-bulk-downloader
```

### Run in docker

Run the built docker image ensuring to pass in all the required ENV vars and any CLI flags you wish to override. See below for an example:

```bash
docker run \
   --rm \
   -ti \
   -v ./downloads:/app/downloads \
   -e AMAZON_USER=userName \
   -e PASSWORD=pass \
   -e OTP=otpCode \
   amazon-kindle-bulk-downloader \
   --baseUrl "https://www.amazon.com"
```

### Docker specific env variable

| Variable             | Description                        | Required           |
| -------------------- | ---------------------------------- | ------------------ |
| `PUPPETEER_HEADLESS` | run puppeteer in headless mode     | no (default false) |
| `PUPPETEER_ARGS`     | additional arguments for puppeteer | no                 |

### Important Notes

In docker `--manualAuth` does not work you must provide credentials via env

If you are on arm64 (i.e. a Mac with an Apple Silicon chip) you mast add `--platform linux/x86_64` when running your `docker run` and `docker build

## Troubleshooting

If you're having any issues with the tool, please first ensure that you have the latest version of the code by running `git pull` or redownloading the repository and then running `bun install` from the root of the repository.

In addition, please ensure that you have followed the [pre-requisites](https://github.com/treetrum/amazon-kindle-bulk-downloader?tab=readme-ov-file#pre-requisites) section carefully and can download a book manually through the website.

### Common Errors

#### Error: Found 0 books in total

This error _may_ indicate that you are authenticated to the wrong Amazon account for your locale.

You need to ensure that you provide the base URL for the Amazon region where your purchases were made. For example, if you are in Australia, you should use `https://www.amazon.com.au`, if you are in the USA, you should use `https://www.amazon.com`, etc.

This base URL should be provided as a CLI argument when running the tool. For example:

```bash
# For USA based accounts
bun run start --baseUrl "https://www.amazon.com"

# For Australia based accounts
bun run start --baseUrl "https://www.amazon.com.au"

# ... etc.
```

#### Error: 404 errors when downloading books

From my testing, 404 errors on book downloads largely occur when the tool is attempting to download an item that is simply not downloadable. From what I can tell, this includes certain comic book purchases. Users with comic book libraries and knowledge of javascript are encouraged to contribute to the project to help test and fix this issue.

There has also [been an indication](https://github.com/treetrum/amazon-kindle-bulk-downloader/issues/162#issue-2864124279) that 404s can ocurr when a book is present on too many devices. If that's the case, you can try deleting the book of some of your devices and try the tool again.

#### Error: Failed to parse CRSF token

This indicates that your login did not fully succeed. There are a multitude of reasons why this could happen however the simplest fix is often to login using the --manualAuth CLI flag instead of using the automated process.

#### Error: `Script not found "start"` when running `bun run start`

This error occurs when you are running `bun run start` from outside the root of the repository. You need to be in the root of the repository to run this command.

After cloning the you will need to change directory (`cd`) into the repo folder by doing:

```bash
cd amazon-kindle-bulk-downloader
```

After which you should be able to run the following without getting errors.

```bash
bun install
bun run start
```

#### All downloads are ~100kb files that are not valid books

The 100kb files are actually error web pages instead of book files and likely indicates that the book files were purchased in another region (even if they show up!).

See [here](https://github.com/treetrum/amazon-kindle-bulk-downloader/issues/192#issuecomment-2676081558) for original report of this.

The fix is to ensure the correct baseUrl is passed for the region that the books were purchased in.

### Docker errors

to disable 'headless' mode for debugging

1. edit `docker-compose.yml` and `.env`, uncomment the relevant lines
2. authorize Docker to talk to the local X server  
   `xhost +local:docker`
3. run the container again
