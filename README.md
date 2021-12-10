# ops-to-devops

My personal notes for Ops colleagues with tasks to do to dive into DevOps work.

* **practical cases to learn new tools by doing**;
* orienteers _where to go_, but you are free with _how to go_;
* but some of first tasks are more detailed to make the start easier.

[![GitHub Super-Linter](https://github.com/and1er/ops-to-devops/workflows/Lint%20Code%20Base/badge.svg)](https://github.com/marketplace/actions/super-linter)

## Contribution

Please, feel free to

* copy/fork/share/star this repository content;
* create issues / pull requests if you have any corrections, suggestions, ideas or if you want to discuss something.

## Before you start

Ensure you

1. are self-education ready;
2. don't afraid to make something work even in a simple/dirty way before making it better;
3. know what's burnout at work (please, avoid this!), if you know Russian, see [this](https://youtu.be/TeSOcYzwx9A) and [this](https://youtu.be/fppiS0gUX7A) videos from our IT colleagues about this;
4. have Linux/Unix environment to work (any Linux, macOS, Windows with WSL or Linux VM onboard), [here's my Linux WS setup example](https://github.com/and1er/linux-ws);
5. have an IDE (I recommend [VSCode](https://code.visualstudio.com/) with a lot of free plugins and Remote-SSH workspace feature);
6. saw the [DevOps Roadmap](https://roadmap.sh/devops) to know approximate path.

## Sign up for GitHub.com

Checklist.

1. I have personal GitHub.com account.
2. I setup 2FA for my account.
3. I generated and added to the account my _SSH key_ for repository authentication (for pull & push).
4. (optional) I generated and added to the account my _GPG key_ for commit signing to have _Verified_ badge for my commits to guarantee I'm an author of my commits (Git allows to set arbitrary `user.name` and `user.email` properties).
5. I set [email settings](https://github.com/settings/emails) to keep my email private in commits
    * `Keep my email addresses private`
    * `Block command line pushes that expose my email`

## English checklist

English language is very important for work. But English study is also a long way, so try to use it daily to improve your skills little by little.

* **Reading**: I use documentation in English as primary information sources.
* **Writing**: I write in English any of own docs, code and commit comments.
* **Listening:** I watch videos and courses by English speakers (at least with subtitles), e.g. on YouTube, Udemy.
* (recommended) **Speaking:** I have a English tutor or group for speaking practice at least for 1-2 hour 1-2 times in a week.

## `PYTHON_APP`: create simple web application

Create a simple web application using Flask web framework.

Key points and actions:

1. Create a new GitHub.com repository for this project.
2. Clone it using SSH.
3. Ensure you have `python3` available in `PATH`.
4. Create and commit **requirements.txt** file with pip package dependecies (`flask`).
5. Create Python virtual environment (venv) in `./venv` directory, activate in the terminal and install requirements there
6. Ensure `./venv` dir is in `.gitignore` — such non-source data **should never be committed**.
7. Write simplest Flask app which returns a string or HTML page on any (`/`) request.
8. Run the app in the venv and check it is available on [http://localhost:5000/](http://localhost:5000/).
9. Ensure you configured Git in your project dir

    ```bash
    git config --local user.name "<GITHUB_USER_NAME>"
    # In (https://github.com/settings/emails use private email
    git config --local user.email "<GITHUB_USER_EMAIL>"

    # In case of GPG signature usage
    git config --local commit.gpgsign true
    git config --local user.signingkey <KEY_ID>

    # Env var for GPG key password prompt in a terminal session
    export GPG_TTY=$(tty)
    ```

10. Commit and push all the changes.

## Get cloud infrastructure for manual deployment

1. Purchase an own domain (to reach your apps by name).
2. Create an AWS account with 1 year free tier plan.
3. **Setup billing alerts to avoid unexpected surprises!**
4. Check what's included to free tier. E.g. don't launch big instances and don't run more that 1 instance for a long time if you don't want to exceed the limits!
5. Go to EC2 service in web console.
6. Choose a region for work.
7. Create a security group with only `80`, `443` and `22` incoming ports opened (cloud firewall).
8. Create minimal EC2 instance with Linux: Ubuntu or Amazon Linux (rpm based distro).
9. Create and assign Elastic IPv4 address to the instance (this is static external IP address).
10. Create a DNS record for your domain pointing to the instance.

## `PYTHON_APP`: manual deploy to internet

It's OK to do everything manually in a dirty way: make it work first.

1. Connect to the instance via SSH using domain name as a host.
2. Setup there your `PYTHON_APP` as you did it locally: git clone (by HTTPS wihtout auth if project is public), venv setup.
3. Run manually using built-in Flask web server (as you did it locally) with `0.0.0.0` host and `80` port and enjoy your app available over global internet! Then stop it.
4. Run the project as systemd service using **gunicorn** or **uWSGI** web application server on `localhost:7000`.
5. Install NGINX and configure it to
   * listen ports `80` and `443`;
   * setup reverse proxy for all requests to application server on `localhost:7000`;
6. Install `certbot` and get Let's Encrypt free TLS certificate.
7. Configure NGINX to
   * use TLS certificate on `443` port;
   * redirect `80 --> 443` port;
8. Enjoy your application available over HTTPS in internet!

## Dockerize `PYTHON_APP`

### Create Docker image

1. Sign up for [Docker Hub](https://hub.docker.com/) — public images are free.
2. Create a `Dockerfile` in the repository, it should
   * start from a Python 3 docker image;
   * copy pip requirements file from the project into the image and install them;
   * copy source code;
   * run application web server as daemon on container start (read what the difference between `CMD` and `ENTRYPOINT` instructions);
3. Install Docker locally. On macOS or Windows Docker Desktop is the simplest solution and it's still free for individual engineers, but there are a lot of `run docker without docker desktop` free solutions.
4. Build the Docker image locally.
5. Run the image locally and map the port from a container to host to see the application in your web browser on `localhost:8080`.
6. Perform `docker login` to Docker Hub.
7. Tag and push the image to Docker Hub.

### Use Docker image

Be noted that Docker uses iptables firewall for own routing and you should be caution with firewall setup if you install and run Docker on hosts in internet.
Here it should be OK because AWS provides cloud firewall rules.

1. SSH to your host.
2. Install Docker (no any problems on Linux).
3. Run a container on `localhost:7001` (shoud differ from the port used by systemd app instance).
4. Reconfigure NGINX to serve the app from a Docker container.

## Setup CI `PYTHON_APP` with GitHub Actions

On push to any branch excepting `main` OR every pull request to `main` branch run following CI actions

* [GitHub SuperLinter](https://github.com/github/super-linter) — universal linter for many types of files. It helps to find out a lot of interesting details.
* Specific Python code linter (e.g. try `flake8`, configure it committing `.flake8` file into the repository).

    Here also recommended to split **requirements.txt** into a group of files

    ```bash
    requirements
    ├── base.txt  # common packages
    ├── dev.txt  # packages for development like linters that don't needed on production
    └── production.txt  # packages for production including app web server
    ```

    e.g.

    ```bash
    # base.txt
    flask==1.1.2

    # dev.txt
    -r base.txt

    flake8==3.8.4
    mccabe==0.6.1
    pycodestyle==2.6.0
    pyflakes==2.2.0

    # production.txt
    -r base.txt
    gunicorn==20.0.4
    ```

* If linters are passed, push the image to Docker Hub.
  * **Docker Hub credentials must be passed via repository secrets!!!**
  * Put `latest` tag if the branch is `main`, or `dev` tag otherwise.

## Future steps

* Terraform: create a new EC2 instance from code.
* Ansible: provision the instance after Terraform to make it run the application.
  * Run Ansible from local WS firstly;
  * Then make it as CD step in GitHub actions providing SSH access from a temporary GitHub Actions VM (via secrets), see [example](https://github.com/and1er/pet-java-api/blob/main/.github/workflows/release.yml).
* CD: In a GitHub actions replace EC2 instance with Terraform then tune the host with Ansible [example](https://github.com/and1er/pet-java-api/blob/main/.github/workflows/release.yml).
* To be continued...
