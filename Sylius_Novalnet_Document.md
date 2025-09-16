# 1. Installation and Environment Setup
## Requirements
1. Unix System (MacOS or Linux)
2. PHP 8.0 or higher
3. MySQL or MariaDB
4. Composer
5. Yarn
## Sylius Shop Installation
1. To begin creating your new project, run this command:
    ```bash
    composer create-project sylius/sylius-standard MyFirstShop
    ```
    It will create a ```MyFirstShop``` directory with a brand new Sylius application inside. Next, move to the project directory:
    ```
    cd MyFirstShop
    ```
2. Before processing next check the **folder and file permissions**.

    Most of the application folders and files require only ***read access***, but a few folders need also the ***write access*** for the **Apache/Nginx** user:

    - var/cache
    - var/log
    - public/media

3. Sylius uses environment variables to configure the connection with the database and mailer services. You can look up the default values in the ```.env``` file and customize them by creating ```.env.local``` with variables you want to override. For example, if you want to change your database name from the default **sylius_%kernel.environment%** to **my_custom_sylius_database**, the contents of that new file should look like the following snippet:
    ```env
    DATABASE_URL=mysql://username:password@host/my_custom_sylius_database_%kernel.environment%
    ```
4. After everything is in place, run the following command to install Sylius:
    ```bash
    bin/console sylius:install
    ```
> [!WARNING]
> During the **sylius:install** command you will be asked to provide important information like Currency, Locale, User Account details, Sample data import, etc.

5. In order to see a fully functional frontend you will need to install its assets.

    Sylius uses **Webpack** to build frontend assets using **Yarn** as a JavaScript package manager.
    ```
    yarn install
    yarn build
    ``````
6. Access the Shop using the **Symfony Local Web Server** by running the below command and then accessing https://127.0.0.1:8000 in your web browser to see the shop.
    ```
    symfony server:start
    ```
    You can log to the administrator panel located at **/admin** with the credentials you have provided during the installation process.
> [!TIP]
> This is a very short introduction to Sylius installation. For a more detailed guide please head to [Sylius Installation](https://docs.sylius.com/en/latest/book/installation/installation.html).

## Novalnet Payment Plugin installation for Sylius Shop
1. Head to the shop directory and Require with composer

    ```bash
    composer require novalnet/sylius-novalnet-payment-plugin --no-scripts
    ```
2. When using Symfony Flex the proper bundle class will be automatically registered in your bundles.php file. Otherwise, add it to your `config/bundles.php` file:

    ```php
    return [
        // ...
        Novalnet\SyliusNovalnetPaymentPlugin\NovalnetSyliusNovalnetPaymentPlugin::class => ['all' => true],
    ];
    ```
3. Import the required config in your `config/packages/_sylius.yaml` file:

    ```yaml
    # config/packages/_sylius.yaml

    imports:
        ...
        - { resource: "@NovalnetSyliusNovalnetPaymentPlugin/Resources/config/config.yaml" }
    ```

4. Import the routing in your `config/routes.yaml` file:

    ```yaml
    # config/routes.yaml

    novalnet_sylius_plugin:
        resource: "@NovalnetSyliusNovalnetPaymentPlugin/Resources/config/routing.yaml"
    ```
5. Install assets
    ```bash
    bin/console assets:install
    ```
6. Clear cache
    ```bash
    bin/console cache:clear
    ```
7. Execute migrations
    ```bash
    bin/console doctrine:migrations:migrate
    ```
> [!Note]
> If your install plugin in `prod`, add `--env=prod --no-debug` to **bin/console** commands.

> [!NOTE]
> To install plugin form the local, add the local repository path to your **composer.json**

```json
  "repositories": [
        {
            "type": "path",
            "url": "/var/www/pugazhenthi/Sylius/NovalnetSyliusNovalnetPaymentPlugin"
        }
    ],
```

## Setup new plugin development environment
  The best way to create your plugin is to use **Sylius plugin skeleton**, which has built-in infrastructure for designing and testing using Behat.
> [!NOTE]
> The plugin can be created anywhere, not only inside a Sylius application, because it already has the test environment inside.

1. Set up a new plugin using the PluginSkeleton.
    ```bash
    composer create-project sylius/plugin-skeleton NovalnetSyliusMyTestPlugin
    ```
> [!IMPORTANT]
> It’s important to follow the naming convention. Symfony requires a "Bundle" postfix to
    load associated services. Sylius stipulates a ***vendor-name prefix*** and **"Plugin"** postfix. In my case, I would name it NovalnetSyliusMyTestPlugin.
2. Get familiar with basic plugin design

    The skeleton comes with a simple application that greets a customer. There are feature scenarios in **features** directory; an exemplary bundle with a controller, a template, and a routing configuration in **src**; and the testing infrastructure in **tests**.

> [!NOTE]
> The **tests/Application** directory contains a sample Symfony application used to test your plugin.

3. Before proceeding check [Sylius Name Conventions](https://docs.sylius.com/en/latest/book/plugins/creating-plugin.html#naming-conventions).

    Besides the way you are creating plugins (based on our skeleton or on your own), there are a few naming conventions that should be followed:

    - Repository name should use PascalCase, must have a __Sylius*__ prefix and a **Plugin** suffix
    - Project composer name should use dashes as a separator, and must have a **sylius** prefix and a **plugin** suffix, e.g.: **sylius-my-test-plugin**
    - Bundle class name should start with ***vendor name***, followed by Sylius and suffixed by ***Plugin*** (instead of Bundle), e.g.: **NovalnetSyliusMyTestPlugin**
    - Bundle extension should be named similarly, but suffixed by the Symfony standard **Extension**, e.g.: **NovalnetSyliusMyTestExtension**.
    - Bundle class must use the **Sylius\Bundle\CoreBundle\Application\SyliusPluginTrait** trait.
    - Namespace should follow **PSR-4**. The top-level namespace should be the vendor name. The second level should be prefixed by Sylius and suffixed by Plugin (e.g. Novalnet\SyliusMyTestPlugin)

> [!NOTE]
> Following the convention mentioned above generates the default configuration key as e.g. **novalnet_sylius_my_test_plugin**

4. **PluginSkeleton** provides some default classes and configurations. However, they must have some default values and names that should be changed. Ref: [Sylius PluginSkeleton Naming changes](https://docs.sylius.com/en/latest/book/plugins/guide/naming.html#naming-changes).

    - In **composer.json**
        - **sylius/plugin-skeleton** -> **novalnet/sylius-my-test-plugin**
        - **Acme example plugin for Sylius.** -> **Sample Test plugin description** (or sth similar)
        - **Acme\\\\SyliusExamplePlugin\\\\** -> **Novalnet\\\\SyliusMyTestPlugin\\\\** (the same changes should be done in namespaces in **src/** directory)
        - **Tests\\\\Acme\\\\SyliusExamplePlugin\\\\** -> **Tests\\\\Novalnet\\\\SyliusMyTestPlugin\\\\** (the same changes should be done in namespaces in **tests/** directory)
    - Rename files
        - src/AcmeSyliusExamplePlugin.php -> src/NovalnetSyliusMyTestPlugin.php
        - src/DependencyInjection/AcmeSyliusExampleExtension.php -> src/DependencyInjection/NovalnetSyliusMyTestExtension.php
    - Adjust namespaces
        - Acme\SyliusExamplePlugin\ -> Novalnet\SyliusMyTestPlugin
        - Similarly will be done for the Sylius application residing under the tests folder:

            - **tests/Application/Kernel.php** has the namespace ***Tests\Acme\SyliusExamplePlugin\Application***, it would become ***Tests\Novalnet\SyliusMyTestPlugin\Application***

            - Do not adjust the namespace for **src/Kernel.php**. It is set to work under the **App** namespace and is not required to be configured to work with the plugin. The same goes for **public/index.php**
    - Adjust **use** statements in files with the correct use statements like below.

        - **tests/Application/public/index.php**:
 ***Tests\Acme\SyliusExamplePlugin\Application\Kernel** -> ***Tests\Novalnet\SyliusMyTestPlugin\Application\Kernel***

    - Add a plugin to load with the test-application update bundle array in **tests/Application/config/bundles.php**
        - **Acme\SyliusExamplePlugin\AcmeSyliusExamplePlugin::class** -> **Novalnet\SyliusMyTestPlugin\NovalnetSyliusMyTestPlugin::class
    - We need to add a configuration node for the plugin. We will do that by modifying **src/DependencyInjection/Configuration.php**:

        - Change **acme_sylius_example_plugin** to **novalnet_sylius_my_test_plugin**.

5. Update Composer, so it will autoload all the files correctly
    ```bash
    composer dump-autoload
    ```
6. Install plugins test Application.
    - Adjust **tests/Application/.env** to configure database details:

        ```yaml
        DATABASE_URL=mysql://dbusername:dbpassword@127.0.0.1:8080/sylius_%kernel.environment%?charset=utf8mb4
        ```
    - If **dependencies have not been installed**, from ***project-root***, run install command (Conditional)
        ```bash
        composer install
        ```
    - Change directory to **tests/Application**
        ```
        cd tests\Application
        ```
    - Install assets and UI components. In **tests/Application**:
        ```bash
        yarn install
        yarn build
        bin/console assets:install public
        ```
    - Set up database and load fixtures. In **tests/Application**:
        ```bash
        bin/console doctrine:database:create
        bin/console doctrine:migrations:migrate
        bin/console sylius:fixtures:load
        ```

    - Start Symfony’s local web server. In **tests/Application**
        ```bash
        symfony server:start -d
        ```
    - You can now view the Sylius test application using the local web address shown in CLI (Eg: https://127.0.0.1:8000)

        ```
        Login credentials loaded with the fixtures:
        User: Sylius
        Password: Sylius
        ```

## Setup Novalnet Sylius payment plugin update environment
1. Begin by cloning the Novlanet Sylius payment plugin project from Git.
    ```bash
    git clone https://github.com/Novalnet-AG/Sylius-payment-integration-novalnet.git
    ```
2. Change to project directory **Sylius-payment-integration-novalnet**
    ```bash
    cd Sylius-payment-integration-novalnet
    ```
3. Run Composer update and install in **Sylius-payment-integration-novalnet**
    ```bash
    composer update
    composer install
    ```
4. Next, proceed with the step **6. Install plugins test Application** procedure outlined in the **Setup new plugin development environment**.

# Novalnet Sylius Plugin installation

1. Require composer

    ```bash
    composer require novalnet/sylius-novalnet-payment-plugin --no-scripts
    ```
2. When using Symfony Flex the proper bundle class will be automatically registered in your bundles.php file. Otherwise, add it to your `config/bundles.php` file:

    ```php
    return [
        // ...
        Novalnet\SyliusNovalnetPaymentPlugin\NovalnetSyliusNovalnetPaymentPlugin::class => ['all' => true],
    ];
    ```
3. Import the required config in your `config/packages/_sylius.yaml` file:

    ```yaml
    # config/packages/_sylius.yaml
    imports:
        ...
        - { resource: "@NovalnetSyliusNovalnetPaymentPlugin/Resources/config/config.yaml" }
    ```
4. Import the routing in your `config/routes.yaml` file:
    ```yaml
    # config/routes.yaml
    novalnet_sylius_plugin:
        resource: "@NovalnetSyliusNovalnetPaymentPlugin/Resources/config/routing.yaml"
    ```
5. Install assets
    ```bash
    bin/console assets:install
    ```
6. Clear cache
    ```bash
    bin/console cache:clear
    ```
7. Execute migrations
    ```bash
    bin/console doctrine:migrations:migrate
    ```
# Novalnet payment configuration in Sylius

1. Log in to the Sylius Admin Interface:

    Go to the admin URL of your Sylius shop. This is usually something like https://yourdomain.com/admin.

2. Access Payment Method Create Section:

    Once logged in, navigate to the **Configuration -> Payment Methods -> Create -> Novalnet Payment** section of the admin interface.

    ![Screenshot payment method create navigation](/images/payment_config/nav_create_payment_method.png)

3. Configure New Payment Method Details:

    Begin by providing essential information about the new payment method. This typically includes:
    - **Payment Details:** Configure the payment method's unique code, position, and enablement for specific channels.

        ![Payment details config](/images/payment_config/payment_details_config_section.png)
    - **Payment Access Credentials:** Enter the payment method's access credentials, such as API keys or merchant IDs, for integration.

        ![Gateway config section](/images/payment_config/payment_gateway_config_section.png)

    - **Payment Method Name and Description:** Enter a descriptive name for the payment method, such as "Credit Card", "Novalnet Payments", or "External Payments" and provide a brief description of the payment method to inform users about its functionality or any specific requirements.

        ![Payment method name and description section](/images/payment_config/payment_name_descritpion_section.png)

    - **Create new payment method:** Click on the 'Create' button to proceed.

        ![Payment method create button](/images/payment_config/payment_method_creat_button.png)

    - **Configure payment gateway required configuration:** After creating the payment method, you'll need to configure essential payment gateway settings such as payment tariff, payment action, and payment test mode.

        ![Payment Gateway Configuration](/images/payment_config/payment_gateway_config_step_2_section.png)

    - **Save payment configuration:** Remember to save the payment method again.

        ![Payment Save button](/images/payment_config/payment_method_save.png)
4. Now, the newly created payment method will be available at checkout for placing orders.

# Novalnet Extension Process
1. Log in to the Sylius Admin Interface
2. **Caputre:** To capture payment for an order, please follow the navigation steps provided below.

    - **Sales > Orders > View specific order > Click 'Complete' button**

        ![Capture button screenshot](/images/payment_extension/transaction_capture_button.png)

    - After the transaction is captured, you will receive a success notification and the transaction note will be added.

        ![Capture success screenshot](/images/payment_extension/transaction_capture_success.png)

3. **Refund:** To refund the payment, please follow the navigation steps provided below.

    - **Sales > Orders > View specific order > Click 'Refund' button**

        ![Refund button screenshot](/images/payment_extension/transaction_refund_button.png)

    - After the transaction is refunded, you will receive a success notification and the transaction note will be added.

        ![Refund success screenshot](/images/payment_extension/transaction_refund_success.png)

4. **Cancel:** To initiate the transaction cancellation, we utilize the order cancellation callback. Therefore, to cancel the payment, it's necessary to cancel the order.

    - **Sales > Orders > View specific order > Click 'Cancel' button**

        ![Order cancel button](/images/payment_extension/order_cancel_button.png)

    - After the transaction is canceled, you will receive a success notification and the transaction note will be added.

        ![Order and transaction cancele success](/images/payment_extension/order_transaction_cancel_success.png)


# How It's Done Guide

## Payment Integration

Refer to the [Sylius payment integration](https://docs.sylius.com/en/latest/cookbook/payments/custom-payment-gateway.html) for further details.

1. Created a new plugin using the PluginSkeleton.
2. We have prepared a gateway factory class and added it to the service.
    ![Gateway Factory class](/images/how_to_do/payment_integration/gateway-factory.png)

    ![Gateway Service](/images/how_to_do/payment_integration/gateway_service.png)
3. Next, we have created a configuration form and added it to the service.
    ![Payment config form](/images/how_to_do/payment_integration/payment_config_form.png)

    ![Payment config form service](/images/how_to_do/payment_integration/payment_config_form_service.png)

4. To introduce support for new configuration fields, we created a value object that will be passed to action, so we can use an API Key provided in the form.

    ![API Client](/images/how_to_do/payment_integration/api_client.png)

    In **src/Factory/NovalnetPaymentGatewayFactory.php** we added support for newly created **NovalnetApiClient** by adding **$config['payum.api']** like below.

    ![Novalnet Api Client config](/images/how_to_do/payment_integration/gateway-factory_api_config.png)

5. From now on, your new Payment Gateway will be available in the admin panel.

    ![Payment gateway buttong](/images/how_to_do/payment_integration/nav_create_payment_method.png)

6. Next have configured the required actions

    - Created **StatusAction** and added it to the service.
        ![StatusAction](/images/how_to_do/payment_integration/status_action.png)

        ![StatusActionService](/images/how_to_do/payment_integration/status_action_service.png)

    - Created **ConvertPaymentAction** and added it to the service.

        ![ConverPaymentAction](/images/how_to_do/payment_integration/converpayment_action.png)

        ![ConverPaymentActionService](/images/how_to_do/payment_integration/converpayment_action_service.png)

    - Created **CaptureAction** and added it to the service.

        ![CaptureAction](/images/how_to_do/payment_integration/capture_action.png)

        ![CaptureAction](/images/how_to_do/payment_integration/capture_action_service.png)

    - Created **AuthorizeAction** and added it to the service.

        ![AuthorizeAction](/images/how_to_do/payment_integration/authorize_action.png)

        ![AuthorizeActionService](/images/how_to_do/payment_integration/authorize_action_service.png)

7. We handle both the redirection return and status updates within the **StatusAction**.

    ![StatusActionExecute](/images/how_to_do/payment_integration/status_action_execute_method.png)

    ![StatusActionRequestStatusUpdate](/images/how_to_do/payment_integration/status_action_request_status.png)

> [!NOTE]
> In addition to the required actions mentioned above, we also need to register additional actions to enable the plugin's functionalities.

## Admin extension process
The payment extension process is done by adding the callback to the **syilus_payment** state machine's callback event.
> [!IMPORTANT]
> If you’re not familiar with our State Machine customization, take a look at Sylius [Customizing State Machines](https://docs.sylius.com/en/latest/customization/state_machine.html).

1. You can attach a callback to it either before or after the transition. The callback is simply a method of a service you want to be executed.
   ![Callback config](/images/how_to_do/extension_process/payment_callback_config.png)
2. In the service method we checked the state machine transition and executed the respective extension process requestion.

    ![Callback service method](/images/how_to_do/extension_process/callback_service_method.png)

3. Register Action for that specific requestion.

    For Ex. **Capture**

    ![Capture action register](/images/how_to_do/extension_process/service_action_cofig.png)

4. Finally verify the request and send an API request for the extension process.

    For Ex. **Caputue**

    ![Capture Action](/images/how_to_do/extension_process/capture_api_action.png)

## Novalnet webhook handling
1. To handle Novalnet Webhook events, we utilized the NotificationController and specified the URL in the ***hook_url*** parameter with the payment method code.

    ![Webhook url](/images/how_to_do/webhook/wehbook_url.png)

2. To process the webhook notification, we retrieve the payment configuration using the payment code passed in the URL and authenticate the webhook notification. After successfully authenticated **Notify** Gateway action executed.

    ![Notification action](/images/how_to_do/webhook/notification_controller.png)

3. In the NotifyAction, the transaction for the current event is retrieved, and the specific action (e.g., TRANSACTION_CAPTURE, TRANSACTION_CANCEL, etc.) is executed.
    ![NotifyAction](/images/how_to_do/webhook/notify_action.png)
> [!NOTE]
> We utilized a payment state machine to update the payment status.
5. Notify action throws an **HttpResponse** to respond to the webhook event.

    ![Webhook Response](/images/how_to_do/webhook/webhook_response_method.png)


# Important Directory and Files Overview
- **/src:** Contains the source code files of the application.
  - **/Action:** Contains the shop payment flow actions.
    - **/Api:** Contains the Novalnet API action.
  - **/Client:** Contains the Novalnet API Client.
  - **/ContextProvider:** Contains context providers for email and payment note templates.
  - **/Controller:**
    - ***NotifyController.php*** Controller to handle Novalnet webhook notifications.
  - **/DependencyInjection:** Contains the Plugins Configurations and Extensions class files.
  - **/Entity:** Contains the Novalnet Transaction and Novalnet History Entities.
  - **/Form:**
    - **/Type:**
      - ***NovalnetGatewayConfigurationType.php*** Payment method configurations form.
  - **/Helper:** Contains utility class for Novalnet plugin.
  - **/Migrations:** Contains the plugin migrations.
  - **/Repository:** Contains the Novalnet Transaction and Novalnet History Repositories.
  - **/Request:**
    - **/Api:** Contains the Payum Generic requests for Novalnet actions.
  - **/Resources:**
    - **/config:** Configuration files for the application.
    - **/public:** Public assets accessible to users.
    - **/translations:** Contains plugins translations.
    - **/views:** Contains the shop templated files.
  - **/Sender** Contains the email sender class.
  - **/StateMachine**
    - ***ExtensionProcessor.php*** Class to handle sylius payment transitions and execute Novlanet extension requests.
  - **/Transaction** Contains Novalnet transaction entity actions.
  - **/Validator** Conatins validator for Novalnet Credentials.
- **/tests:** Houses all the unit tests and integration tests.
- **/features:** Contains ".feature" files, which are used in Behavior-Driven Development (BDD) testing to describe the behavior of software features

# Running plugin tests

  - PHPUnit

    ```bash
    vendor/bin/phpunit
    ```

  - PHPSpec

    ```bash
    vendor/bin/phpspec run
    ```

  - Behat (non-JS scenarios)

    ```bash
    vendor/bin/behat --strict --tags="~@javascript"
    ```

  - Behat (JS scenarios)

    1. [Install Symfony CLI command](https://symfony.com/download).

    2. Start Headless Chrome:

      ```bash
      google-chrome-stable --enable-automation --disable-background-networking --no-default-browser-check --no-first-run --disable-popup-blocking --disable-default-apps --allow-insecure-localhost --disable-translate --disable-extensions --no-sandbox --enable-features=Metal --headless --remote-debugging-port=9222 --window-size=2880,1800 --proxy-server='direct://' --proxy-bypass-list='*' http://127.0.0.1
      ```

    3. Install SSL certificates (only once needed) and run test application's webserver on `127.0.0.1:8080`:

      ```bash
      symfony server:ca:install
      APP_ENV=test symfony server:start --port=8080 --dir=tests/Application/public --daemon
      ```

    4. Run Behat:

      ```bash
      vendor/bin/behat --strict --tags="@javascript"
      ```

  - Static Analysis

    - Psalm

      ```bash
      vendor/bin/psalm
      ```

    - PHPStan

      ```bash
      vendor/bin/phpstan analyse -c phpstan.neon -l max src/
      ```

  - Coding Standard

    ```bash
    vendor/bin/ecs check
    ```

# Opening Sylius with your plugin

- Using `test` environment:

    ```bash
    (cd tests/Application && APP_ENV=test bin/console sylius:fixtures:load)
    (cd tests/Application && APP_ENV=test bin/console server:run -d public)
    ```

- Using `dev` environment:

    ```bash
    (cd tests/Application && APP_ENV=dev bin/console sylius:fixtures:load)
    (cd tests/Application && APP_ENV=dev bin/console server:run -d public)
    ```
