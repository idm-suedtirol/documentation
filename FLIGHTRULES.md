# Flight rules for ODH

#### What are "flight rules"?

A [guide for astronauts](https://www.jsc.nasa.gov/news/columbia/fr_generic.pdf) (now, programmers contributing to the
Open Data Hub) about what to do when things go wrong, _or must be executed with no delay_.

> *Flight Rules* are the hard-earned body of knowledge recorded in manuals that list, step-by-step, what to do if X
> occurs, and why. Essentially, they are extremely detailed, scenario-specific standard operating procedures. [...]

> NASA has been capturing our missteps, disasters and solutions since the early 1960s, when Mercury-era ground teams
> first started gathering "lessons learned" into a compendium that now lists thousands of problematic situations, from
> engine failure to busted hatch handles to computer glitches, and their solutions.

&mdash; Chris Hadfield, *An Astronaut's Guide to Life*.

_ps. Idea taken from the [GIT flight rules](https://github.com/k88hudson/git-flight-rules) project._

----

#### Table of Contents
<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->


- [Licensing and REUSE compliance](#licensing-and-reuse-compliance)
- [Work Flow and Release Management](#work-flow-and-release-management)
  - [I want to create a project for ODH](#i-want-to-create-a-project-for-odh)
  - [I want to know how the git flow works for a project of ODH](#i-want-to-know-how-the-git-flow-works-for-a-project-of-odh)
  - [I want to release a new version of an ODH-project](#i-want-to-release-a-new-version-of-an-odh-project)
  - [I found a small bug, and want to fix it immediately on a released version](#i-found-a-small-bug-and-want-to-fix-it-immediately-on-a-released-version)
  - [I want to update the database schema of bdp-core](#i-want-to-update-the-database-schema-of-bdp-core)
- [GitHub](#github)
  - [I want to create a repository](#i-want-to-create-a-repository)
  - [I want to give access to a public repository](#i-want-to-give-access-to-a-public-repository)
  - [I want to give access to a private repository](#i-want-to-give-access-to-a-private-repository)
  - [I want to prepare the CI for pull requests](#i-want-to-prepare-the-ci-for-pull-requests)
- [Pimcore](#pimcore)
  - [I want to update Pimcore](#i-want-to-update-pimcore)
- [Jenkins Pipelines](#jenkins-pipelines)
  - [I want to execute arbitrary commands on the remote server](#i-want-to-execute-arbitrary-commands-on-the-remote-server)
  - [I want to execute git commands on the remote server](#i-want-to-execute-git-commands-on-the-remote-server)
  - [I want to create a cron-job to clone AWS instances](#i-want-to-create-a-cron-job-to-clone-aws-instances)
- [Servers](#servers)
  - [I want to create a test clone of a server instance](#i-want-to-create-a-test-clone-of-a-server-instance)
  - [I want to give server-access via SSH to an external contributor](#i-want-to-give-server-access-via-ssh-to-an-external-contributor)
  - [I want to create a new Pimcore server instance on AWS](#i-want-to-create-a-new-pimcore-server-instance-on-aws)
  - [I want to use a swap file on my server](#i-want-to-use-a-swap-file-on-my-server)
  - [I want to make my web server HTTPS compliant](#i-want-to-make-my-web-server-https-compliant)
  - [I want to mount an aws s3 bucket on my server](#i-want-to-mount-an-aws-s3-bucket-on-my-server)
    - [Prerequisit](#prerequisit)
    - [Setup](#setup)
- [Documentation](#documentation)
  - [I want to add a table of contents to a markdown file](#i-want-to-add-a-table-of-contents-to-a-markdown-file)
- [Database](#database)
  - [I want to create a read-only user (aka role)](#i-want-to-create-a-read-only-user-aka-role)
  - [I want to diff two tables with the same schema](#i-want-to-diff-two-tables-with-the-same-schema)
  - [I want to change visibility of mobility data](#i-want-to-change-visibility-of-mobility-data)
    - [I want to declare some records as open data](#i-want-to-declare-some-records-as-open-data)
    - [I want to add a new user](#i-want-to-add-a-new-user)
    - [I want to disable an existing user](#i-want-to-disable-an-existing-user)
    - [I want to add a new role](#i-want-to-add-a-new-role)
    - [I want to combine a role with users](#i-want-to-combine-a-role-with-users)
    - [I want to define filter rules for a certain role](#i-want-to-define-filter-rules-for-a-certain-role)
    - [I want to debug my rules](#i-want-to-debug-my-rules)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

----

## Licensing and REUSE compliance

See REUSE-specific [flight rules](https://github.com/idm-suedtirol/reuse).

## Work Flow and Release Management

### I want to create a project for ODH

Each project must be on our github `idm-suedtirol`. So, if you want to create a new repository, do:

    cd your-project
    git init
    git add <some-files>
    git commit -m "Initial import"
    git remote add origin https://github.com/idm-suedtirol/<your-git-repository>.git
    git push -u origin master

...or push an existing repository:

    cd your-project
    git remote add origin https://github.com/idm-suedtirol/<your-git-repository>.git
    git push -u origin master

Then, create a development branch and protect the master branch (See Github>Settings>Branches>Protected branches).

Important notes:
  - We consider a `master` branch as production-ready at each new HEAD
  - We consider a `development` branch in an unstable-testing state
  - You can optionally also have additional branches for features or hotfixes

NB: We can make an exception of this repository structure, iff a project is all of the following...
  - on a private repository
  - has no external contributors
  - is a really simple project

...then you do not need to create a `development` branch and do not protect the `master` branch.

### I want to know how the git flow works for a project of ODH

This work is based on [Vincent Driessen's post](https://nvie.com/posts/a-successful-git-branching-model/), about a
successful Git branching model, and [Semantic Versioning](https://semver.org). We suggest, that you read these
articles to better understand what follows.

Please read [I want to create a project for ODH](#i-want-to-create-a-project-for-ODH) before you start.

We have two main branches:
  - `master` contains production-ready commits only
     - Each HEAD has a version bump commit, and a corresponding git tag
  - `development` contains the unstable work-in-progress commits
     - No version commits, except from historical ones

...and supporting branches:
  - `release-<version>` branches off from `development` during last steps before release
    - No new features are allowed here, just some minor bugfixes and adaptations
    - Bump versions during this stage
  - `hotfix-<version>` branches off from `master` and allow fast and uncomplicated bug fixes, without intermediate
    steps inside `development`
    - Must be merged to `master` and `development` at the end
    - no fast-forward (`--no-ff`)

### I want to release a new version of an ODH-project

We have a `master` branch with a release at `1.0.0` and want now to release a new `development` branch. The new version
should be `1.1.0`, since we add a new feature. (See https://semver.org to understand how to choose a new version)

First, we create a new `release-1.1.0` branch, and bump the version on that branch.

    git checkout -b release-1.1.0 development
    ./my-personal-script-to-bump-version 1.1.0
    git commit -am "Bump version to 1.1.0"

Assume, we find out, that there is a minor bug. This can be fixed immediately on this branch. This new branch may exist
there for a while, until the release may be rolled out definitely. During that time, bug fixes may be applied in this
branch. Adding large new features here is strictly prohibited. They must be merged into `development`, and therefore,
wait for the next big release. Finally, we merge this new release into `master`, and tag it with version `1.1.0`.

    git checkout master
    git merge --no-ff release-1.1.0
    git tag -a 1.1.0

We merge all changes also to `development`, and delete the release-branch, when done.

    git checkout development
    git merge --no-ff release-1.1.0
    git branch -d release-1.1.0

### I found a small bug, and want to fix it immediately on a released version

We have a `master` branch with a release at `1.0.0` and want now to fix a bug. The new version should be `1.0.1`, since
this is just a bugfix and not changing functionality. (See https://semver.org to understand how to choose a new version)

Create a `hotfix-1.0.1` branch, and bump the version.

    git checkout -b hotfix-1.0.1 master
    ./my-personal-script-to-bump-version 1.0.1

Do your work and commit the fix. Include an `issue-id` to the commit message, which you can get from the github
issue tracker. If the issue is not present now, create a new one and describe the issue briefly. Assume, we have an
issue on github with id `13`.

    git commit -am "Fix severe production issue\nFixes #13."

Merge it back into `master` (with a tag) and `development` (without tag).

    git checkout master
    git merge --no-ff hotfix-1.0.1
    git tag -a 1.0.1
    git checkout development
    git merge --no-ff hotfix-1.0.1

We do not need the `hotfix` branch any longer.

    git branch -d hotfix-1.0.1


### I want to update the database schema of bdp-core

I did something inside [bdp-core](https://github.com/idm-suedtirol/bdp-core), that involves a schema change.
First of all, you need to understand if Hibernate can handle this change with its `update` strategy. See
[bdp-core/dal](https://github.com/idm-suedtirol/bdp-core/blob/master/dal/src/main/resources/META-INF/persistence.xml)
for further details. Best thing is, to run the newest bdp-core instance on the test server, and check schema diffs. Use
a clone of the existing production database to be as similar as possible to the current production database state.

    pg_dump -U bdp -s bdp -h postgres-test-server.example.com -p 5432 -n intime > /tmp/schema-dump-postgres-test-server.sql
    pg_dump -U bdp -s bdp -h postgres-prod-server.example.com -p 5432 -n intime > /tmp/schema-dump-postgres-prod-server.sql
    diff /tmp/schema-dump-postgres-test-server.sql /tmp/schema-dump-postgres-prod-server.sql

If it is simple and log files do not show any issues with the new change, install the new bdp-core on the production environment
and let Hibernate handle all schema changes.

However, if the change is more complex, like new views or functions, dump the original schema from the production
environment, and create a migration script that updates that schema. See our official
[database documentation](http://opendatahub.readthedocs.io/en/latest/guidelines/database.html) for details.

    pg_dump -U bdp -s bdp -h postgres-prod-server.example.com -p 5432 -n intime > /tmp/schema-dump-postgres-prod-server.sql
    createdb -U postgres -h localhost -p 5432 __bdptest

We need also `postgis`:

    apt install postgresql-9.5-postgis-2.4
    psql -U postgres -h localhost -p 5432 -d __bdptest -c "CREATE EXTENSION postgis with schema public;"

Now, we are ready to import the production server schema, and test the migration script, for ex. `schema-1.0.0-1.1.0.sql`.

    psql -U postgres -h localhost -p 5432 -d __bdptest -1 -e -v ON_ERROR_STOP=1 < /tmp/schema-dump-postgres-prod-server.sql
    psql -U postgres -h localhost -p 5432 -d __bdptest -1 -e -v ON_ERROR_STOP=1 < schema-1.0.0-1.1.0.sql

NB: If something fails, solve each shown error by yourself, and repeat steps above. Update `schema-1.0.0-1.1.0.sql` accordingly.

If you are satisfied with your schema migration script, test it on our test environment and see if Hibernate fills logs with
errors. If not, your ready for production.

Finally, we no longer need the test database `__bdptest`, hence we can drop it:

    psql -U postgres -h localhost -p 5432 -c "DROP DATABASE __bdptest;"

## GitHub

### I want to create a repository

Create a new GitHub repository (private or public).

Create a `Jenkinsfile-CI` in the repository and specify the build steps.

Create a new Jenkins pipeline at [https://ci.opendatahub.bz.it](https://ci.opendatahub.bz.it)

- Type: **Multibranch Pipeline**
- Branch Sources: **GitHub**
    - Attention: For **Discover pull requests from forks** the trust part must be set to **From users with Admin or Write permission** to only allow users with Admin or Write permissions to modify the Jenkinsfile
- Build Configuration: Reference the Jenkinsfile with the continuous integration build steps
- Properties: Enable **Enable project-based security** and set the following permissions

### I want to give access to a public repository

Provide authenticated users access to the Continuous Integration Jenkins Server:

- Configure the Jenkins Pipeline for the repository at [https://ci.opendatahub.bz.it](https://ci.opendatahub.bz.it)
- Add **Read (Job)** permissions to authenticated users under Properties (project-based security)

### I want to give access to a private repository

Provide the contributor access to the GitHub repository:

- Add the contributor with **read** permissions to the GitHub private repository.

Provide the contributor access to the Continuous Integration Jenkins Server:

- Configure the Jenkins Pipeline for the repository at [https://ci.opendatahub.bz.it](https://ci.opendatahub.bz.it)
- Add the contributors GitHub username under Properties (project-based security) and give him **Read (Job)** permissions

### I want to prepare the CI for pull requests

Due to security constraints, during the build of a pull request the Jenkinsfile of the target branch will be executed (ex. master).

However, to still include CI steps for the pull request, the steps must be defined in the target branch (ex. master). To prevent that the CI is then failing for this target branch and succeeding for the pull request, an if statement can be added to only execute the tests when certain files exist.

```
stage('Test') {
    steps {
        sh 'if [ -d "path/to/file/of/new/part" ]; then ./testing-command-of-new-part; else true; fi'
    }
}
```

## Pimcore

### I want to update Pimcore

I want to update my Pimcore to version `5.3.1`.

Run `/var/www/html/bin/console --update=5.3.1`

It needs at least 2GB of RAM, consider to [enable a swap file](#i-want-to-use-a-swap-file-on-my-server), if it fails
due to low memory error.

## Jenkins Pipelines

### I want to execute arbitrary commands on the remote server

You need to use `ssh` for that. Your pipeline script looks similar to this:

    pipeline {
        agent any
        environment {
            KEY = credentials('private-key-123')
        }
        stages {
            stage('Whatever') {
                steps {
                   sh 'ssh -i ${KEY} user@1.2.3.4 "touch IT_WORKS"'
                }
            }
        }
    }

Upload the private key as secret file to Jenkins and give it the ID `private-key-123`.

Jenkins needs to add this host to its `known_hosts` file. Open a ssh session to your
Jenkins server, and allow that remote host.

    ssh admin@your-jenkins-server
    sudo -iu jenkins
    ssh user@1.2.3.4

This gives:

    The authenticity of host [...]
    Are you sure you want to continue connecting (yes/no)? yes

Finally you will see a `public key violation` error, but that's OK, because we do
not have the private key available outside Jenkins' credentials module.

Test it inside the Jenkins web-console and see what `Console Output` shows up.

### I want to execute git commands on the remote server

Please follow all instructions of
[I want to execute arbitrary commands on the remote server](#i-want-to-execute-arbitrary-commands-on-the-remote-server)
first.

Change your pipeline script, that is, exchange `touch IT_WORKS` with some git commands.

Connect to the remote server, and add an ssh-key for your git account.

    ssh user@1.2.3.4
    ssh-keygen
    cat ~/.ssh/id_rsa.pub

Copy `id_rsa.pub` contents into your allowed [SSH keys](https://github.com/settings/keys)
section of github. Alternatively, you can also copy an existing private key, that has already
been added to your github account to `user@1.2.3.4:/.ssh/`.

Test it with

     git pull

Make sure that `git config -l` shows a ssh protocol for `remote.origin.url`, i.e.,
`git@github.com:your_username/your_project.git`. If not, change it inside `.git/config`.

### I want to create a cron-job to clone AWS instances

I want to create a clone of an instance each night at 3am, to have a test
environment with the actual production state.

First, create a pipeline script inside `server-deployment` repository:

    git clone git@github.com:idm-suedtirol/server-deployment.git
    cat > Jenkinsfile-MyClone-Nightly-Clones << EOF
        pipeline {
            agent any
            environment {
                AWS_ACCESS_KEY_ID = credentials('aws_ec2scripts_secret_key_id')
                AWS_SECRET_ACCESS_KEY = credentials('aws_ec2scripts_secret_access_key')
                AWS_DEFAULT_OUTPUT = 'json'
                AWS_DEFAULT_REGION = 'eu-west-1'
            }
            stages {
                stage('Clone') {
                    steps {
                        sh '''
                            cd utils
                            SRC_INSTANCE_ID=i-0123456789abcde0 CLONE_INSTANCE_NAME=dolly ./aws-ec2-clone
                        '''
                    }
                }
            }
        }
    EOF
    git commit -am "Add Jenkinsfile-MyClone-Nightly-Clones"
    git push

Create a new pipeline inside `Server Deployment` from template `Pipeline`.
Activate `Build periodically` to a schedule `H 3 * * *` (`H` is a good idea to
distribute it evenly from 3am to 4am).

Use a `Pipeline script from SCM` definition, and set
`https://github.com/idm-suedtirol/server-deployment.git` as repository URL.

Set `Clean after/before checkout` and add your script path to
`Jenkinsfile-MyClone-Nightly-Clones`.


## Servers

### I want to create a test clone of a server instance

I want to clone an AWS instance with ID `i-0123456789abcde0` to a new instance
with name `dolly`.

Use `utils/aws-ec2-clone` from `server-deployment.git` which needs `aws ec2`
command-line tools. See this script's comments for further details and
configuration options.

    git clone git@github.com:idm-suedtirol/server-deployment.git
    cd utils
    SRC_INSTANCE_ID=i-0123456789abcde0 CLONE_INSTANCE_NAME=dolly ./aws-ec2-clone

### I want to give server-access via SSH to an external contributor

On AWS open ssh ports in inbound for the IP's you want to grant access.

Access the server with ssh

    ssh -i admin_key.pem admin@1.2.3.4

Create a user for the external contributor Chuck

    sudo adduser chuck

Store Chuck's public ssh key in the `authorized_keys` file

    sudo -u chuck bash
    mkdir .ssh
    chmod 700 .ssh
    touch .ssh/authorized_keys
    chmod 600 .ssh/authorized_keys
    cat chuck_id.pub >> .ssh/authorized_keys

Tell Chuck, that all is setup and that he should change his password as soon as possible.

In addition, if Chuck should be administrator on that server, add him to the sudoers group.

    sudo usermod -aG sudo chuck


### I want to create a new Pimcore server instance on AWS

Get our server-deployment repository

    git clone git@github.com:idm-suedtirol/server-deployment.git

Configure the `utils/aws-launch-debian-pimcore` script. All needed information can be found inside the script.

    vim utils/aws-launch-debian-pimcore

Run it

    ./utils/aws-launch-debian-pimcore

Go to the AWS console and name the new EC2 `Instance`, `Volume`, `Elastic IP`, and `Security Group`. Use a prefix for
all of them with a pattern like `<test|prod>-pimcore-detail`

    Instance = prod-pimcore-hackathon
    Volume = prod-pimcore-hackathon-volume
    Elastic IP = prod-pimcore-hackathon-eip
    Security Group = prod-pimcore-hackathon-sg

Get our pimcore-automation repository

    git clone git@github.com:idm-suedtirol/pimcore-automation.git

Copy `install-pimcore.sh` to your new server instance:

    scp -i ~/.ssh/your-key.pem install-pimcore.sh admin@3.4.5.6:

Access your server and make the script executable

    ssh -i ~/.ssh/your-key.pem admin@3.4.5.6
    chmod +x install-pimcore.sh

Configure the `install-pimcore.sh` script. All needed information can be found inside the script.

    vim install-pimcore.sh

Run it

    sudo ./install-pimcore.sh

Test it with a browser and see if `http://3.4.5.6` shows the [Pimcore](https://pimcore.com) welcome screen.

### I want to use a swap file on my server

I have a server, which does not have enough RAM, and I do not want to upgrade it now. However, some tools
run out of memory.

Check if you have a swap file installed:

    sudo swapon --summary

Enable a 4GB swap file:

    sudo fallocate -l 4G /mnt/4GB.swap
    sudo dd if=/dev/zero of=/mnt/4GB.swap bs=1024 count=4194304
    sudo mkswap /mnt/4GB.swap
    sudo chmod 0600 /mnt/4GB.swap
    sudo swapon /mnt/4GB.swap

A script can be found on our [server-deployment](https://github.com/idm-suedtirol/server-deployment/blob/master/utils/swap-file-create)
repository.

### I want to make my web server HTTPS compliant

I have a website reachable with http://www.davinci.bz.it or http://davinci.bz.it
on an IP `34.247.202.9`, and want to make it HTTPS only.

We use a proxy with Apache and [Let's Encrypt](https://letsencrypt.org/) for
that. Let's assume the IP of that proxy is `1.2.3.4`.

    ssh admin@1.2.3.4

Configure Apache on `1.2.3.4` with

    sudo vim /etc/apache2/sites-available/davinci.bz.it.conf

...with...

	<VirtualHost *:80>
		<Proxy *>
			Order Allow,Deny
			Allow from all
		</Proxy>
		ServerName davinci.bz.it
		ServerAlias www.davinci.bz.it
		ProxyPass / http://34.247.202.9/
		ProxyPassReverse / http://34.247.202.9/
		ProxyPreserveHost On
		ErrorLog /var/log/apache2/davinci.bz.it
	</VirtualHost>

Enable the new configuration

    cd /etc/apache2/sites-enabled/
    sudo ln -s /etc/apache2/sites-available/davinci.bz.it.conf davinci.bz.it.conf

Finally, get your certificates and final configuration. Execute...

	sudo /opt/certbot-auto --apache

	...
	2: davinci.bz.it
	3: www.davinci.bz.it
	...

Type, `2,3` to select both names for your new certificate, and choose
`2: Redirect - Make all requests redirect to secure HTTPS access`.

Define your firewall, such that only the proxy `1.2.3.4` can point to port `80` of
`34.247.202.9`. On AWS refine the corresponding Security Group's inbound rule:

    Type:         HTTP
    Protocol:     TCP
    Port:         80
    Source:       1.2.3.4/32
    Description:  LETS ENCRYPT PROXY


Finally, tell your ICT guys to point the DNS entries for `davinci.bz.it` to
the proxy.

### I want to mount an aws s3 bucket on my server

#### Prerequisit
- an aws s3 bucket with no dots in it's name
- [s3fs](https://github.com/s3fs-fuse/s3fs-fuse)
- AWS IAM user with (at least) read access to the specific bucket

#### Setup

Once everything is ready you should already be able to mount the bucket. The s3fs tool right now is not working properly when the bucket has dots in it's name.
In your home directory create a file `.passwd-s3fs` and insert your aws user credentials in this form `ACCESS_KEY_ID:SECRET_ACCESS_KEY`.

`umask` and `allow_other` are options which allow you to define permissions for other users, but can not break permissions set on the AWS IAM user.

To mount the bucket do as follows:
```
s3fs bucketname:/ /mountFolder -o passwd_file=~/.passwd-s3fs -o umask=0222 -o allow_other
```
For more specific configurations and mount options refer to the manual page and the official documentation

## Documentation

### I want to add a table of contents to a markdown file

Open the markdown file, i.e., README.md, where you want to add a the table of contents and
add the following lines after the main title and initial paragraphs:

    <!-- START doctoc -->
    <!-- END doctoc -->

Use the [doctoc](https://github.com/thlorenz/doctoc) command to update the markdown file. First install that command with

    sudo npm install -g doctoc

...then execute it

    doctoc README.md --github --title '#### Table of Contents'

Finally publish it with

    git add README.md
    git commit -m "Docs: Update table of content for README.md"
    git push

## Database

If not otherwise stated, all these chapters are about PostgreSQL.

### I want to create a read-only user (aka role)

    CREATE ROLE read_only_user WITH LOGIN PASSWORD 'your_strong_password';
    GRANT CONNECT ON DATABASE your_database TO read_only_user;
    GRANT USAGE ON SCHEMA your_schema TO read_only_user;
    GRANT SELECT ON ALL TABLES IN SCHEMA your_schema TO read_only_user;
    GRANT SELECT ON ALL SEQUENCES IN SCHEMA your_schema TO read_only_user;
    COMMENT ON ROLE read_only_user IS 'Read-only account for your_reason';

### I want to diff two tables with the same schema

The following SQL query gives the diff of two tables called `your-first-table` and `your-second-table`,
or an empty result if both are equal.

    WITH
        table1 AS (table your-first-table),
        table2 AS (table your-second-table)
    SELECT * FROM (
        (TABLE table1 EXCEPT ALL TABLE table2)
        UNION
        (TABLE table2 EXCEPT ALL TABLE table1)
    ) temp;


### I want to change visibility of mobility data

This chapter explains how to add users and roles, and to define which data can be seen as
open data, and what is closed.

_Please note: Only API methods regarding measurements are restricted. Methods regarding
stations and types are considered open-data due to their meta-data character, i.e.,
`get-station`, `get-station-details`, or `get-data-types` always expose all data._

#### I want to declare some records as open data

Ex.1: Declare all records related to station `station xyz` open data.

    INSERT INTO bdprules(role_id, station_id, type_id, period)
        SELECT r.id, s.id, null, null FROM bdprole r, station s
            WHERE r.name = 'GUEST' AND stationcode = 'station xyz';

Ex.2: Make atmospheric pressure measurements of all meteorological stations open-data.

    INSERT INTO bdprules(role_id, station_id, type_id, period)
        SELECT r.id, s.id, t.id, null FROM bdprole r, station s, type t
            WHERE r.name = 'GUEST' AND stationtype = 'Meteostation' AND cname = 'atmospheric-pressure';

> NB: `GUEST` contains all open-data sources, just add rules for that role and you
> will open it without the need of authentication through tokens, or username/password.

Finally, refresh the materialized view to reflect the new changes:

    REFRESH MATERIALIZED VIEW bdppermissions;

#### I want to add a new user

    INSERT INTO bdpuser(email, password)
        VALUES ('this_is_my@ema.il', crypt('top-S3CR3T', gen_salt('bf')));

#### I want to disable an existing user

    UPDATE bdpuser SET enabled = false WHERE email = 'this_is_my@ema.il';

#### I want to add a new role

    INSERT INTO bdprole(name) VALUES ('Role A');
    INSERT INTO bdprole(name, parent_id)
        VALUES ('Role B', (select id from bdprole where name = 'GUEST'));

> Make sure that you define at least parent as `GUEST` (which is always present
> and allows access to all open data sources); `ADMIN` is another default, but it
> sees all data.
>> `Role A` cannot see anything at the moment, whereas `Role B` inherits everything
>> from `GUEST`.

Finally, refresh the materialized view to reflect the new changes:

    REFRESH MATERIALIZED VIEW bdppermissions;

#### I want to combine a role with users

    INSERT INTO bdpusers_bdproles(user_id, role_id) VALUES (
        (SELECT id FROM bdpuser WHERE email = 'this_is_my@ema.il'),
        (SELECT id FROM bdprole WHERE name = 'Role A')
    );

...or, if you know primary keys...

    INSERT INTO bdpusers_bdproles(user_id, role_id) VALUES (2, 3);

#### I want to define filter rules for a certain role

    INSERT INTO bdprules(role_id, station_id, type_id, period)
        SELECT r.id, s.id, null, null FROM bdprole r, station s
            WHERE r.name = 'Role A' AND stationcode = 'station xyz';

> This means, `Role A` can see any `(type,period)` combination for `station xyz`.

Finally, refresh the materialized view to reflect the new changes:

    REFRESH MATERIALIZED VIEW bdppermissions;

Rule information:
- If you define new filter rules for parents, all children automatically inherit them.
- `(station, type, period)` is a hierarchical triple, that is, if you open `station`
to all (`= NULL`), `type` and `period` are no longer considered.

#### I want to debug my rules

    SELECT * FROM bdppermissions;

> This query, shows you the permissions view, which is a flattened representation of
> all rules.

*How to interpret the output?*
This table consists of five columns `(uuid, role_id, station_id, type_id, period)`. It is a minimal set of rules,
this means, that there are no overlapping rules. The `uuid` column is a primary key, not important for debugging. The
rest defines the rule itself, for example, `(1,null,null,null)` means that a role with id `1` can see everything. A cell
set to `null` stands for *no restriction*. Another example could be `(2,5,null,null)`, which means that role with id `2`
sees all types and periods of station `5`. However, since `(station, type, period)` is a hierarchical triple (read from
left-to-right), something like `(3,null,null,1)` is considered a erroneous permission rule.
