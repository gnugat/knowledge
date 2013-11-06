# Local application in bundles

Manually testing a standalone bundle is always a good idea. To do so, there is
no need to create an empty symfony project, install the bundle inside it and
launch the application.

The bundle can have its own local application.

## Local console

Create the application's kernel (don't forget to register your bundle):

    <?php
    // File: Resources/local/app/AppKernel.php

    use Symfony\Component\HttpKernel\Kernel;
    use Symfony\Component\Config\Loader\LoaderInterface;

    class AppKernel extends Kernel
    {
        public function registerBundles()
        {
            return array(
                new Symfony\Bundle\FrameworkBundle\FrameworkBundle(),
                // <-- Put your bundle here!
            );
        }

        public function registerContainerConfiguration(LoaderInterface $loader)
        {
            $loader->load(__DIR__.'/config/config.yml');
        }
    }

The FrameworkBundle needs some configuration:

    # File: Resources/local/config/config.yml
    framework:
        secret: "Three can keep a secret, if two of them are dead."

Then create a console:

    #!/usr/bin/env php
    <?php
    // File: Resources/local/app/console
    require_once __DIR__.'/../../../vendor/autoload.php';

    require_once __DIR__.'/AppKernel.php';

    use Symfony\Bundle\FrameworkBundle\Console\Application;
    use Symfony\Component\Console\Input\ArgvInput;
    use Symfony\Component\Debug\Debug;

    Debug::enable();

    $input = new ArgvInput();

    $kernel = new AppKernel('test', true);
    $application = new Application($kernel);
    $application->run($input);

Finally run the console:

    php Resources/local/app/console
