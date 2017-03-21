---
layout: post
title:  "Creating commands in Magento 2"
date:   2016-02-19 22:25:00 -0300
categories: magento2
published: true
---

Magento 2 provides a new command-line interface based in Symfony 2 Console Component. It's a great addition for developers that need to write cli tasks or commands for their own modules.

We'll cover in this post how to create a new command and register it in Magento's system so it's available to be used within the `bin/magento` tool.

Let's say we occasionally need to get a CSV of current customer's names and emails in our Magento 2 site. In order to make this happen, we can create a command that can be called by a cron job or manually by a system administrator. Let's take advantage of the new Magento cli tool.

We won't go deep into modules creation.

- Create the module's folder in `app/code`

    Convention: `<magento-root>/app/code/<vendor>/<module>/`

    In my example: `<magento-root>/app/code/Federivo/CommandsExample/`

- Create the module.xml file

    `<magento-root>/app/code/Federivo/CommandsExample/etc/module.xml`:

    {% gist ed0601169b135bc78fa6595a47f186b2 module.xml %}

- Create the di.xml file

    `<magento-root>/app/code/Federivo/CommandsExample/etc/di.xml`:

    {% gist ed0601169b135bc78fa6595a47f186b2 di.xml %}

Now let's dig a little bit in the `di.xml`

The `di.xml` file enables us to inject arguments in class constructors (among other interesting things). Check the class `Magento\Framework\Console\CommandList`. You'll see its constructor has an argument named "commands".

So with the `di.xml` file we are telling Magento to inject a new object in the "commands" argument of `Magento\Framework\Console\CommandList` constructor. The object we are injecting will be an instance of `Federivo\CommandsExample\Command\ExportCustomerCommand`. This way Magento will be aware of the existence of our new command.
And that brings us to the class we need to create that will perform the actual export.

- Create the ExportCustomerCommand class

    `<magento-root>/app/code/Federivo/CommandsExample/Command/ExportCustomerCommand.php`





