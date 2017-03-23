---
layout: post
title:  "Creating commands in Magento 2"
date:   2017-03-22 22:25:00 -0300
categories: magento2
published: true
---

Magento 2 provides a new command-line interface based in Symfony 2 Console Component. It's a great addition for developers that need to write cli tasks or commands for their own modules.

We'll cover in this post how to create a new command and register it in Magento's system so it's available to be used within the `bin/magento` tool.

Let's say we occasionally need to get a CSV of current customer's names and emails in our Magento 2 site. In order to make this happen, we can create a command that can be called by a cron job or manually by a system administrator. Let's take advantage of the new Magento cli tool.

For the sake of the example let's define the required functionality as this:

> User should be able to run a command in order to get a file with a list of customers including customer name and email. User should be able to send a parameter to specify the desired output format (csv or json).

Let's do it.

We won't go deep into modules creation.

#### Create the module's folder in `app/code`

Convention: `<magento-root>/app/code/<vendor>/<module>/`

In my example: `<magento-root>/app/code/Federivo/CustomerExport/`

#### Create the module.xml file

`<magento-root>/app/code/Federivo/CustomerExport/etc/module.xml`

{% gist ed0601169b135bc78fa6595a47f186b2 module.xml %}

#### Create the di.xml file

`<magento-root>/app/code/Federivo/CustomerExport/etc/di.xml`

{% gist ed0601169b135bc78fa6595a47f186b2 di.xml %}

Now let's dig a little bit in the `di.xml`

The `di.xml` file enables us to inject arguments in class constructors (among other interesting things). Check the class `Magento\Framework\Console\CommandList`. You'll see its constructor has an argument named "commands".

So with the `di.xml` file we are telling Magento to inject a new object in the "commands" argument of `Magento\Framework\Console\CommandList` constructor. The object we are injecting will be an instance of `Federivo\CustomerExport\Command\ExportCustomerCommand`. This way Magento will be aware of the existence of our new command.
And that brings us to the class we need to create that will perform the actual export.

#### Create the ExportCustomerCommand class

`<magento-root>/app/code/Federivo/CustomerExport/Command/ExportCustomerCommand.php`

{% gist f6605e943c3414e9d8da25b148efbcdc ExportCustomerCommand.php %}


There are 3 important things: Extend your command from `\Symfony\Component\Console\Command\Command`, the `configure()` method and the `execute()` method.

Let's review each one.

#### Extend your class from `\Symfony\Component\Console\Command\Command`. 

{% highlight php startinline %}
<?php
use Symfony\Component\Console\Command\Command;

class ExportCustomerCommand extends Command
{% endhighlight %}

Inject all the dependencies needed in the `__construct()` method.

{% highlight php startinline %}
<?php
public function __construct(
    CustomerRepositoryInterface $customerRepository,
    SearchCriteriaBuilder $searchCriteriaBuilder,
    Csv $csvHandler,
    DirectoryList $directoryList,
    File $fileHandler,
    $name = null
)
{
    $this->customerRepository = $customerRepository;
    $this->searchCriteriaBuilder = $searchCriteriaBuilder;
    $this->csvHandler = $csvHandler;
    $this->directoryList = $directoryList;
    $this->fileHandler = $fileHandler;
    parent::__construct($name);
}
{% endhighlight %}

#### Implement the `configure()` method. 

Here we set the configuration for our command. Notice the `setName()`, `setDescription()` and `setDefinition()`.

`setName()` allows you to define how we'll be calling this command from the shell. `setDescription()` will set the description of our command that will be displayed in the shell when we run `bin/magento` without arguments. `setDefinition()` accepts an array of `Symfony\Component\Console\Input\InputOption` where we define our command's arguments. In this case, the "output" argument.

#### Implement the `execute()` method.

This is where the real work happens. This method is the one that gets triggered when you call the command from the shell.
Notice the arguments `Symfony\Component\Console\Input\InputInterface $input` and `Symfony\Component\Console\Output\OutputInterface $output`.

The $input argument will give you access to the parameters entered by the user when calling the command. We use it like: `$input->getOption("output")`.

The $output argument will let you send messages back to the shell by using its `write()` or `writeln()` methods. We use it for informing the user that the export process is finished and where the export file is located: `$output->writeln("[INFO] Export finished. Export file generated: {$filename}");`

Now the only thing left is to install our module and start using it.

Install: 

```shell
bin/magento module:enable Federivo_CustomerExport
bin/magento setup:upgrade
```

Use:

```shell
bin/magento federivo:export-customers --output=json
bin/magento federivo:export-customers --output=csv
```

And that's the end of our customers export command.

You can download / fork / check the working code sample for this post from this [Github repo](https://github.com/federivo/magento-code-samples)

Hope this have been useful.




