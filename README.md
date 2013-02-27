wizzlern-drupal-phing
=====================

Deployment of Drupal installations using Phing and Jenkins.

# (future) charateristics
* Supports different deployment strategies.
* Can be used with Jenkins and at the command line.
* Can be used in multiple user teams with different individual setups.
* Includes tasks for deployment, rollback and cleanup.

# Supported tasks
* Deploy to live
* Deploy to acceptance, test (future)
* Sync live to development, acceptance, test (future)
* Sync acceptance, test to development, test (future)
* Rollback previous live to live (future)
* Cleanup previous live (future)

## Task: Deploy to live
### SSH Strategy
Current live and future live have separated code base and separated databases.
Use symlink to switch live (SSH access required, not possible via FTP).
Rollback by changing symbolic links.
Using Git checkout or rsync to deploy the code base on the live server.
* Build future live code
  - Git scenario: No action required
  - Rsync scenario: Create a build in Jenkins workspace based on git branch or git tag
* Prepare server file system for deployment
  - If no previous live exists: 
    - Copy complete current live to future live 
     Restore database credentials at future live settings.php
  - If previous live exists:
    - Copy files directory from current live to future live
* Place the files/code on the server
  - Git scenario: git checkout of branch or tag
  - Rsync scenario: rsync future live with Jenkins workspace
* Prepare database for deployment
  - Copy live database to future live
* Update the database
  - Run update script on database
  - Update (revert) features
  - Clear cache
  - Optionally: clear reverse proxy cache
* Finalize deployment
  - Unlink current live
  - Create symlink for new live
  - Create symlink previous live (required for rollback)

### FTP Strategy
Overwrite the live code base. Use the same database for current and future live.
Rollback by means of re-deploy previous code and database backup.
* Build future live code
  - Create a build in Jenkins workspace based on git branch or git tag
* Prepare database for deployment
  - Backup the live database (required for rollback)
  - Put the site offline
* Prepare server file system for deployment
  - no activity
* Place the files/code on the server
  - ftp from Jenkins workspace to server
* Update the database
  - Run update script on database
  - Update (revert) features
  - Clear cache
  - Optionally: clear reverse proxy cache
* Finalize deployment
  - Put the site online

## Task: Deploy to acceptance, test
### Update strategy
Update the code base, do not change the content related files.
No database backup is made. No rollback facility.
* Build future live code
  - Create a build in Jenkins workspace based on git branch or git tag
* Prepare server file system for deployment
  - Create directory to receive the code base
  - In case of rsync strategy: copy current live to future live
* Place the files/code on the server
  - Option 1: git checkout of branch or tag
  - Option 2: rsync future live with Jenkins workspace
  - Option 3: ftp from Jenkins workspace (zip) to future live
  - If required: update database credentials in settings.php
* Prepare database for deployment
  - Copy live database to future live
* Update the database
  - Copy live database to future database
  - Run update script on database
  - Update (revert) features
  - Clear cache
  - Optionally: clear reverse proxy cache
* Finalize deployment
  - Unlink current live
  - Create symlink for new live
  - Create symlink previous live (required for rollback)
