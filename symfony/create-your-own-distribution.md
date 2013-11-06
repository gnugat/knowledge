# Create your own Symfony Distribution

Creation of the "Symfony Without AcmeDemoBundle Edition" distribution, as an
example to show how to create your own Symfony distribution and keep it up to
date with the standard one.

## Creation of the repository

    mkdir symfony-without-acme
    cd symfony-without-acme
    git init
    git remote add origin git://github.com/<pseudo>/symfony-without-acme.git
    git remote add symfony-standard git://github.com/symfony/symfony-standard.git
    git fetch symfony-standard

### Creating the master

    git branch symfony-standard/master symfony-standard/master
    git merge symfony-standard/master

Change the composer package name:

    sed -i 's#symfony/framework-standard-edition#<pseudo>/symfony-framework-without-acme-edition#' composer.json
    sed -i 's/Standard/Without AcmeDemoBundle/' composer.json
    git add composer.json
    git commit -m 'Changed Composer package name'

You might also want to change the `README.md` and `LICENSE` files.

Remove AcmeDemoBundle:

    git rm -rf src/Acme
    sed -i '/Acme/d' app/AppKernel.php
    git add app/AppKernel.php
    sed -i '/Acme/d' app/config/routing_dev.yml
    sed -i '/acme/d' app/config/routing_dev.yml
    git add app/config/routing_dev.yml
    cat <<EOT > app/config/security.yml
    security:
        encoders:
            Symfony\Component\Security\Core\User\User: plaintext

        role_hierarchy:
            ROLE_ADMIN:       ROLE_USER
            ROLE_SUPER_ADMIN: [ROLE_USER, ROLE_ADMIN, ROLE_ALLOWED_TO_SWITCH]

        providers:
            in_memory:
                memory:
                    users:
                        user:  { password: userpass, roles: [ 'ROLE_USER' ] }
                        admin: { password: adminpass, roles: [ 'ROLE_ADMIN' ] }

        firewalls:
            dev:
                pattern:  ^/(_(profiler|wdt)|css|images|js)/
                security: false

            login:
                security: false

            secured_area:
                anonymous: ~

        access_control:
            #- { path: ^/login, roles: IS_AUTHENTICATED_ANONYMOUSLY, requires_channel: https }
    EOT
    git add app/config/security.yml

Saving and publishing the changes:

    git commit -m 'Removed AcmeDemoBundle'
    git push -u origin master

### Creating version 2.3

    git checkout -b 2.3 symfony-standard/2.3

Then re-do and save the modifications done on `master` by hand, as versions
might not require the exact same changes.

To publish the changes:

    git push -u origin 2.3
    git tag -d v2.3.0 # The Symfony Standard tag
    git tag -a v2.3.0 -m 'created tag 2.3.0'
    git push origin v2.3.0 # Be careful to not push all tags

To prepare future updates:

    git branch symfony-standard/2.3 symfony-standard/2.3

## Update of the repository

### Updating master

    git checkout symfony-standard/master
    git pull
    git checkout master
    git merge symfony-standard/master

Solve the conflicts, if any.

    git push

### Updating version 2.3

    git checkout symfony-standard/2.3
    git pull
    git checkout 2.3
    git merge symfony-standard/2.3

Solve the conflicts, if any.

    git tag -d v2.3.1 # The Symfony Standard tag
    git tag -a v2.3.1 -m 'created tag 2.3.1'
    git push origin v2.3.1 # Be careful to not push all tags
    git push
