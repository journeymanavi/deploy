# deploy

This is a **generalized deployment automation script** to deploy 
**Node.js web applications** to a **Linux server**, by fetching deployment artifacts
**from GitHub Releases**.

The general flow of this scripts is as follows:

1. **Fetch deployment artefacts** from GitHub Releases. For this The script may 
  rely on command line arguments to identify and target a particular app deployment 
  or may use suitable hard coded values.

  For this step, _ssh agent forwarding_ will need to be setup, if release artefacts are fetched via git. 
  Or, it might need the use of GitHub's _Personal Access Token_.

2. **Extract the fetched artefacts** in an appropriate location. This should broadly 
  give you the follow items:
    * Application source (src/, client/, server/, build/, public/, assets/, lib/, .js, .css, .html etc.)
    * Dependencies file(s) (package.json and yarn.lock)
    * License file

3. **Install app dependencies** by running ``npm install``or ``yarn install`` as appropriate.

4. **Run backups as needed**. This might include backing up the DB, config, los, etc.

5. **Bring the application down** and replace with static maintenance notification page.

6. **Run any supporting deployment scripts** (db migration, env update, nginx config update etc.)

7. **Switch any symlinks** that need to be switched to point to the new deployment.

8. **Bring application back up** by restarting appropriate services (pm2/nginx/mongo).
  For this step, any app specific env or other config should be supplied via command line argument to the 
  deployment script. The script should use this accordingly to start the application with.

  > Check if above steps can be done _without causing application downtime_.

9. **Housekeep** by deleting - temp files/dirs, deployments older than some threshold a appropriate, etc.

The script has two modes `setup` and `deploy`. The `setup` mode can be used for 
to setup the required directory structure and config files on the server as a 
one-time step. The `deploy` mode can be used for all subsequent deployments 
of an application that's already setup.

# Usage

```sh
Usage: deploy [mode-option] <arguments>

Deployment is done relative to the present working directory.

Mode Options:
  -d                          deploy   : Perform fresh deployment (default)
  -s                          setup    : Perform one-time setup
  -b                          rollback : Rolls back to previous deployment

Arguments:
  ** Required for all modes **
  -n <app-name>               App name

  ** Reuqired only for 'setup' mode **
  -a <app-main-script>        Name of the entry point script of the app
  -e <file/"name val">        The environment variables to be set for the app
                              
                              The argument can either be a file containing all 
                              environment variables (format: name=val, one 
                              per line)
                              or
                              one or more instances of -e can be used to set
                              individual env variables (format: name=val)
  -u <github-user>            GitHub user
  -r <github-repo>            GithUb repo

  ** Required only for 'deploy' mode **    
  -t <github-tag>             GitHub tag to be deployed
```

# Future Enhancements

* Implement deployment steps that allow you to
  * Perform automated backups of important entities before deployment
  * Run arbitrary scripts as part of the deployment
  * Perform the deployment in a way that does not cause any application downtime
  * Check a 'status' endpoint of the application post deployment to make sure 
    deployment was successful
* Create a `trigger` component that can be used to trigger deployments remotely
* Feature to omit `-t <tag>` argument and fetch `latest` tagged release artefacts 
  in that case
