# Workshop Guide: Automated GraphQL Security Testing

This repo contains a guidebook for the StackHawk workshop, "Automated GraphQL Security Testing." In the workshop, we help walk you through scanning a simple GraphQL application in GitGub Actions (a CI/CD platform) with the HawkScan DAST scanner.

While attending the workshop, you should use this guidebook as a reference for code samples and commands as they come up. It's not cheating to copy and paste!

Not attending our workshop right now? Watch the [June '21 GraphQL workshop](https://www.youtube.com/watch?v=7SiYpZYDlEg) in the [StackHawk YouTube channel](https://www.youtube.com/c/StackHawk) on your own schedule. As our marketing department likes to say, "like and subscribe!"

---

## Prerequisites

To get the most out of this workshop, make sure you have the following prerequisites before getting started.

* Discord 
  * [Install](https://discord.com/) Discord
  * [Join](https://discord.gg/gAqPJPva) the StackHawk Server
  * Find us in the [#graphql-security-testing](https://discord.gg/vXbm3VmE) channel
* Docker -- [install](https://docs.docker.com/get-docker) the latest version
* HawkScan -- `docker pull stackhawk/hawkscan`
* The GitHub CLI
  * [Install](https://github.com/cli/cli#installation)
  * Login -- `gh auth login`

## Step 1: Fork the Test Application

Fork the vuln-graphql-api app.

```shell
# From the website
open https://github.com/kaakaww/vuln-graphql-api

# Or with the GitHub CLI
gh repo fork kaakaww/vuln-graphql-api
```

Clone it to your workstaion.

```shell
# With git:
git clone <YOUR-GITHUB-ORG>/vuln-graphql-api

# Or with the GitHub CLI:
gh repo clone vuln-graphql-api

# Enter your cloned project directory
cd vuln-graphql-api
```

Prepare the vuln-graphql-api project directory for the workshop

```shell
./scripts/workshop-prep.sh
```

## Step 2: Run the Test App

Build and run

```shell
export SERVER_PORT=3000
docker compose up --build --detach
```

Browse to the test app

```shell
open http://localhost:3000
```

## Step 3: My First HawkScan

Sign up for a StackHawk Developer Account

```shell
open https://app.stackhawk.com
```

Scan vuln-graphql-api

```shell
export SERVER_PORT=3000
export API_KEY=hawk.XXXXXXXxXXXXXXXxxxxXXXXXxxxx
docker run -t -e API_KEY -v $(pwd):/hawk --network host stackhawk/hawkscan
```

## Step 4: Tune for GraphQL

Update your stackhawk.yml configuration:

```yaml
# ./stackhawk.yml
app:
 applicationId: ${APP_ID}
  env: Development
  host: http://localhost:3000
  graphqlConf:
    enabled: true
    operation: QUERY
  autoPolicy: true
  autoInputVectors: true
hawk:
  spider:
    base: false
```

Scan again

```shell
docker run -t -e API_KEY -v $(pwd):/hawk --network host stackhawk/hawkscan
```

## Step 5: Automate in GitHub Actions

Create `./.github/workflows/hawkscan.yml`

```yaml
# .github/workflows/hawkscan.yml
name: HawkScan
on:
  push:
jobs:
  hawkscan:
    name: HawkScan
    runs-on: ubuntu-20.04
    steps:
      - name: Clone repo
        uses: actions/checkout@v2
      - name: Build and run vuln-graphql-api
        run: >
          SERVER_PORT=3000
          docker-compose up --build --detach
      - name: Run HawkScan
        uses: stackhawk/hawkscan-action@v1.3.1
        with:
          apiKey: ${{ secrets.HAWK_API_KEY }}
```

Commit. Push. Scan.

```shell
git add .
git commit -m "bring the noise"
git push
```

## All Done

Congratulations!

Now try it on *your* GraphQL application! Here are some additional resources to help you on your way.

* [HawkDocs](https://docs.stackhawk.com), where you can read all the details on how to configure and run HawkScan in your environment.
* [GraphQL Configuration](https://docs.stackhawk.com/hawkscan/configuration/graphql-configuration.html), where you can find more details on tuning your GraphQL scan
* [Continuous Integration](https://docs.stackhawk.com/continuous-integration/), where you can see our guides for integrating HawkScan with the most popular CI/CD systems
