# NSS Learning Platform

## Local Development

To run the application locally, follow these steps

1. Make sure you are able to run the backend project locally; instructions can be found [here](https://github.com/Artemis-Apprentice/learning-platform-api).
1. Clone this repository with the command

   ```
   git clone https://github.com/Artemis-Apprentice/learning-platform-client
   ```

1. Open the project directory
   ```
   cd learning-platform-client
   ```
1. Build the docker container
   ```
   docker compose up --build
   ```
   (you only need to use the `--build` flag the first time you run this repo; after that `docker compose up` should work fine)
1. Once the docker container has finished building, you'll need to sign into the application with your GitHub profile.
   1. Go to `localhost:3000`
   1. Sign in with GitHub.
      This creates a user in the learning platform database with the information from your GitHub profile.
1. Next, you need to make sure the user associated with your GitHub profile has staff user permissions in the admin database portal.
   1. Go to `localhost:8000/admin`
   1. Sign in using the admin username/password you chose when setting up the backend.
   1. Click "Users" and search for your name.
   1. Click your user record, check the box for Staff User, then scroll to the bottom and save.
1. Finally, go back to `localhost:3000` and log in. You should see a list of cohorts to join.
   1. If you don't see it, try closing all open tabs/windows with `localhost:3000` then opening it & signing in again in a new tab.

## Automatic Deploys

This walkthrough is for auto-deploying a React application to a Digital Ocean droplet.

### Initial Setup

1. Create the new DNS entry in the Networking tab on Digital Ocean. Point it to the droplet you are configuring.
1. SSH into your server in a new terminal.
1. `sudo apt install nginx git curl wget`

### Repository Setup

1. Create `.github/workflows/main.yml` file in your project directory.
1. Paste the following text.

   ```yml
   # This is a basic workflow to help you get started with Actions

   name: Continuous Deploy

   # Controls when the workflow will run
   on:
   # Triggers the workflow on push or pull request events but only for the main branch
   push:
       branches: [ main ]
   pull_request:
       branches: [ main ]

   # Allows you to run this workflow manually from the Actions tab
   workflow_dispatch:

   # A workflow run is made up of one or more jobs that can run sequentially or in parallel
   jobs:
   # This workflow contains a single job called "build"
   build:
       # The type of runner that the job will run on
       runs-on: self-hosted

       # Steps represent a sequence of tasks that will be executed as part of the job
       steps:
       # Checks-out your repository under $GITHUB_WORKSPACE, so your job can access it
       - uses: actions/checkout@v2

       # Runs a set of commands using the runners shell
       - name: Run a multi-line script
           run: |
           npm install
           npm run build --if-present
   ```

````

1. Commit and push that to Github.
1. Go to the Settings tab of your Github repo.
1. Go to Actions tab on left.
1. Click on Runners in the new General nav item that appears.
1. Click button to create new self-hosted runner.
1. Click the Linux radio button.
1. Follow the steps and accept all the defaults when asked.
1. When you get to `# Last step, run it!`, do not run the command listed. Instead of executing `./run.sh`, you will execute `sudo ./svc.sh install`.
1. Then execute `sudo ./svc.sh start`

### Trigger Action

Now your droplet will be the runner for building your application.

1. Go to your project directory and make some trivial change.
1. Commit and push change to Github.
1. You can watch the process by clicking on the Actions tab on Github and watch your app be built for production.
1. Once the Action is complete, go back to the terminal where you are in your droplet.
1. `cd ./_work/{project name}/{project name}`
1. Stay in this directory. You'll need to run `pwd` later.
1. Run `ls` and you'll see your project. It will also have a `build` directory since your Action ran `npm run build`.

### Server Setup

1. `cd /etc/nginx/sites-available`
1. `sudo vim react-app`
1. Go back to the other terminal where you are in the directory that the Github Action created for you.
1. Run `pwd` and copy the path.
1. Put the following text into the `react-app` file. Replace everthing inside `{}` with the appropriate value.

   ```
   server {
       server_name {your DNS entry - e.g. app.domain.com};

       root {paste what you copied in the previous step}/build;

       index index.html index.htm;

       location / {
           try_files $uri $uri/ /index.html$is_args$args;
       }

       access_log /var/log/nginx/client.log;
   }
   ```

1. Save and exit.
1. `sudo ln -s /etc/nginx/sites-available/react-app /etc/nginx/sites-enabled/react-app`
1. `sudo systemctl daemon-reload`
1. `sudo service nginx reload`
````
